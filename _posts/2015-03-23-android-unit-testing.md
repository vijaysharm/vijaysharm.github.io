---
layout: post
title: Android Testing
---

Testing on Android gets a pretty bad rap. However, it is what it is, and we have to make the most of what we've got. In this post, I'll describe how I go about testing on Android, striving for as much coverage as possible.

The first thing to note, in this article is, I'll cover how I test Activities specfically. More specifically, I'll describe how I do instrumentation testing of Activities. I'll also make use of the Espresso and Mockito libraries to do the for testing and verification, but you could bring your own assertion mechanisms.

The hardest thing about testing on Android is how coupled most things are to the Context. Articles on using Dependency Injection attempt to solve this problem, but I've always found setting up injection for testing to be a little unsatisfying. Moreover, I was never a fan of the fact that most examples of using DI required getting the object graph by getting a handle to an instance of the Application. To me, that just meant I had to create some kind of Application stand-in for every test. I wanted a way where I had more control over the objects I was giving my activity.

To overcome the above, the first place I start with is the Activity. Here, I have a contrived example of a LoginActivity in which has a class to handle the "business logic". In this example, the "business logic" is factored out into a class called `LoginOperations` which has a single asynchronous interface to perform the login action.

```java
public class LoginActivity extends Activity {
	LoginOperations operations;
	private Button login;
	...

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        ...
    }

    @Override
    protected void onPostCreate(Bundle savedInstanceState) {
        super.onPostCreate(savedInstanceState);
        ...
        login.setOnClickListener(new View.OnClickListener() {
	        operations.login(username, password, new LoginOperations.Callback() {
	            @Override
	            public void onSuccess() {
	            	...
	            }
	            @Override
	            public void onFail() {
	            	...
	            }
	        });
        });
        ...
    }
}
```
The most interesting tidbit from the above is the fact that I've attached the listener in the `onPostCreate` method. What? Why? The reason becomes more clear when we look at the `Application`. Wading through `Application` class, I discovered that there is a way for me to receive callbacks on an Activity's lifecycle. Why is this important? Well, it gives me access to the `Activity` just after creation.

```java
public class App extends Application implements Application.ActivityLifecycleCallbacks {
...
	@Override
    public void onCreate() {
        super.onCreate();
        registerActivityLifecycleCallbacks(this);
    }

    @Override
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        if (activity instanceof LoginActivity) {
            ((LoginActivity) activity).operations = new LoginOperations();
        }
    }
...
}
```

So from the above, the callback for `onActivityCreated` is invoked __after__  the Activities `onCreate` method. This means, at this point in the `Application`, I have access to the Activity, and can _inject_ whatever properties the class needs. In the example here, I simply assign a new instance, which isnt very useful. In practice you might pass an object with the same lifetime as the application. The tradeoff here ofcourse is that the fields in the Activity are not marked as private. Alternatively, you could using something like Dagger to fill the object using injection by replacing the above line with something like `ObjectGraph.inject(activity)`. Using Dagger means the fields cannot be marked private in any case, so the above point might be mute.

At this point, it should be clear why I'm using `onPostCreate`. I move all the code I would normally call in `onCreate` into `onPostCreate` since at this point, I have an activity with all of its fields inflated. `onCreate` has been regulated to simply calling 'setContentView' and assigning views using `findViewById`.

So that's just the volley. In order to spike the benefits of doing the above, we need to talk about writing an actual test. Below is an example of a test I might write verifying what the UI might look like on a successful login.

```java
public class LoginActivityTest extends ActivityInstrumentationTestCase2<LoginActivity> {
	private LoginOperations operations;
	private LoginActivity activity;
	...
    protected void setUp() throws Exception {
        super.setUp();
        operations = Mockito.mock(LoginOperations.class);
        activity = getActivity();
        activity.operations = operations;
        getInstrumentation().callActivityOnPostCreate(getActivity(), null);        
	}

	public void test_login_success() throws InterruptedException {
        ...
        onView(withId(R.id.login)).perform(click());

        verify(operations).login(
            eq(USERNAME),
            eq(PASSWORD),
            captor.capture()
        );

        final LoginOperations.Callback callback = captor.getValue();
        final CountDownLatch latch = new CountDownLatch(1);
        getInstrumentation().runOnMainSync(new Runnable() {
            @Override
            public void run() {
                callback.onSuccess();
                latch.countDown();
            }
        });
        latch.await(500, TimeUnit.MILLISECONDS);

        onView(withId(R.id.error)).check(matches(withText("Success!")));
    }
}
```

From the above, I create a mock instance of my `LoginOperations` class, and pass it into the activity on `setUp`. Then I invoke `onPostCreate` using a handy instrumentation call (you could just call it youself, but you need to make sure you call it on the main Android UI thread). In the test, I make use of Mockito's `ArgumentCaptor` to grab the callback object, and invoke the call I want. In another test, I could just as easily have called `onFail`. The tricky part here is I need to call it on the main UI thread, so I make use of `CountDownLatch` as barrier for the call to complete before checking the state of my UI. In practice, your Activity might launch another activity, in which case you can make use of the `ActivityMonitor`.

Since I've adopted this pattern, I feel like its allowed me better opportunities to test my Activities using Instrumentation. Hopefully this sort of pattern is helpful to you.
