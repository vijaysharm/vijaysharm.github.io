---
layout: post
title: Writing an Android app
published: false
---

## Motivation
Writing a whole mobile applications can be a daunting thing. Be it Android or iOS, there are so many moving pieces to consider. One of those pieces is dealing with synchronizing those pieces based on event. The new hotness in event synchronization is reactive programming. It makes sense; be it user input, or events from the system, a user application is very much suited to reactive programming. Using RxJava in your Android applications can get you there.

I didn't want to write yet another article on using RxJava in Android. What I'm hoping to do is to show you how I used it to solve concrete problems.

## Login Screen
Our application is going to have a login screen. Maybe later, I'll document a way of allowing your application to work without requiring a login, but let's assume for now that we'll require the user to login.

The login screen itself is very basic. We'll use fragments to create a login fragment. The fragment contains two edit text fields, and a login button. I've also put a simple text field that we'll use to show the user in the case of error.

At some point, the login screen needs to make a network call to log a user in. 

1. You need to make your network observable never throw an exception. This kills the ViewObservable subscription to the button. This means that if there's a failure logging the user in (bad credentials, no network, etc...), then your Retrofit Observable will throw an exception. In order to get around this, you really need to 

1. None of your Subscribers can throw an exception. This means you need to keep them lean, and make sure they handle all possible input values. If a subscriber throws an exception while processing on a _live_ observable, the the subscriber will be removed.

1. Always consider the initial state, then hook up events. Although we're making use of a reactive library to propagate events throughout the system, always keep in mind that your UI is a finite state machine, and if events transition your UI from one state to another, then you __must__ think about what the initial state must be. In our case, we can have one of several first states. 
  1. The username/password are empty, in which case you want the cursor in the username field with the login button disabled, and the error message hidden
  1. The Username is provided, but the password is empty, in which case, you might want to put the cursor in the password field, with the login button disabled, and the error message hidden
From here, you can connect the events of a user entering their username, password, and the affect they might have on the button or on validation

```
public interface AuthenticationService {
    @POST("/login") public Observable<Token> login(@Body Credentials credentials);
    @GET("/logout") public Observable<Void> logout(@Query("token") String token);

    public static class Token {
        private final String username;
        private final String token;

        public Token(String username, String token) {
            this.username = username;
            this.token = token;
        }

        public String getToken() {
            return token;
        }

        public String getUsername() {
            return username;
        }

        @Override
        public String toString() {
            return "Token {" + "username='" + username + '\'' + ", token='" + token + '\'' + '}';
        }
    }

    public static class Credentials {
        private final String username;
        private final String password;

        public Credentials(String username, String password) {
            this.username = username;
            this.password = password;
        }

        public String getUsername() {
            return username;
        }

        public String getPassword() {
            return password;
        }
    }
}
```

```
public class Service {
    public static Observable<Token> login(String username, String password) {
        RestAdapter restAdapter = new RestAdapter.Builder()
                .setEndpoint(Constants.HOST)
                .build();
        final AuthenticationService service = restAdapter.create(AuthenticationService.class);
        final Credentials credentials = new Credentials(username, password);

        return service.login(credentials);
    }
}
```

```
public class LoginFragment extends Fragment {
    public interface Callback {
        void subscribe(PublishSubject<Token> token);
    }

    private static final String USERNAME_KEY = "username";

    @InjectView(R.id.username) EditText usernameEditText;
    @InjectView(R.id.password) EditText passwordEditText;
    @InjectView(R.id.login) Button loginButton;
    @InjectView(R.id.error) TextView error;

    private final PublishSubject<Token> token = PublishSubject.create();
    private String username;
    private Subscription login;

    public static LoginFragment newInstance(String param1) {
        LoginFragment fragment = new LoginFragment();
        Bundle args = new Bundle();
        args.putString(USERNAME_KEY, param1);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            username = getArguments().getString(USERNAME_KEY);
        }
    }

    @Override
    public View onCreateView(
        LayoutInflater inflater,
        ViewGroup container,
        Bundle savedInstanceState
    ) {
        View view = inflater.inflate(R.layout.fragment_login, container, false);
        ButterKnife.inject(this, view);

        usernameEditText.setText(username);
        Observable<Boolean> usernameValid = ViewObservable.text(usernameEditText, !username.isEmpty())
            .map(new Func1<OnTextChangeEvent, Boolean>() {
                @Override
                public Boolean call(OnTextChangeEvent onTextChangeEvent) {
                    return Checks.isUsernameValid(onTextChangeEvent.view.getText().toString());
                }
            });

        if ( !username.isEmpty() ) { passwordEditText.requestFocus(); }
        Observable<Boolean> passwordValid = ViewObservable.text(passwordEditText)
            .map(new Func1<OnTextChangeEvent, Boolean>() {
                @Override
                public Boolean call(OnTextChangeEvent onTextChangeEvent) {
                    return Checks.isPasswordValid(onTextChangeEvent.view.getText().toString());
                }
            });

        Observable.combineLatest(usernameValid, passwordValid, new Func2<Boolean, Boolean, Boolean>() {
            @Override
            public Boolean call(Boolean isUsernameValid, Boolean isPasswordValid) {
                return isUsernameValid && isPasswordValid;
            }
        }).subscribe(new ObserverAdapter<Boolean>() {
            @Override
            public void onNext(Boolean enabled) {
                loginButton.setEnabled(enabled);
            }
        });

        login = ViewObservable.clicks(loginButton)
            .doOnEach(new ObserverAdapter<OnClickEvent>() {
                @Override
                public void onNext(OnClickEvent onClickEvent) {
                    usernameEditText.setEnabled(false);
                    passwordEditText.setEnabled(false);
                    loginButton.setEnabled(false);
                    error.setVisibility(View.GONE);
                }
            })
            .flatMap(new Func1<OnClickEvent, Observable<Token>>() {
                @Override
                public Observable<Token> call(OnClickEvent onClickEvent) {
                    return login(
                        usernameEditText.getText().toString(),
                        passwordEditText.getText().toString()
                    );
                }
            })
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(token);

        token.subscribe(new ObserverAdapter<Token>() {
            @Override
            public void onNext(Token token) {
                usernameEditText.setEnabled(true);
                passwordEditText.setEnabled(true);
                loginButton.setEnabled(true);

                if (token == null) {
                    error.setVisibility(View.VISIBLE);
                    error.setText("Invalid username or password");
                }
            }
        });

        return view;
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        if (activity instanceof Callback) {
            Callback callback = (Callback) activity;
            callback.subscribe(token);
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        if (login != null) login.unsubscribe();
    }

    private static Observable<Token> login(final String username, final String password) {
        return Observable.create(new Observable.OnSubscribe<Token>() {
            @Override
            public void call(final Subscriber<? super Token> subscriber) {
                Service.login(username, password).subscribe(new ObserverAdapter<Token>() {
                    @Override
                    public void onNext(Token token) {
                        subscriber.onNext(token);
                    }

                    @Override
                    public void onError(Throwable e) {
                        subscriber.onNext(null);
                    }
                });
            }
        });
    }
}
```

```
public class MainActivity extends Activity implements LoginFragment.Callback {
	...
	@Override
    public void subscribe(PublishSubject<AuthenticationService.Token> token) {
        token.subscribe(new ObserverAdapter<AuthenticationService.Token>() {
            @Override
            public void onNext(AuthenticationService.Token token) {
                SharedPreferences.Editor editor = preferences.edit();
                if ( token == null ) {
                    editor.remove(Constants.TOKEN_KEY);
                } else {
                    editor.putString(Constants.TOKEN_KEY, token.getToken());
                    Log.i("Tag", "Logged in: " + token);
                }
                editor.commit();
            }
        });
    }
    ...
}
```