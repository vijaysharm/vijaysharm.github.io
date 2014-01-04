---
layout: post
title: Lessons learned from the Google I/O Android application – Part 1: Fragments
---
The Google I/O 2012 Android application has a plethora of knowledge useful to any developer looking for solutions to common development problems. The source code for the for the project can be found here. This series of articles aims to disect some of the nuggets of information I’ve found useful for my development.

### Fragments

Android first introduced fragments with Honeycomb, and has since added them to the support library. If you’ve never heard of or used fragments, do yourself the pleasure of searching the internet for more information about them, or picking up any good Android book. Most importantly, learn about a fragments lifecycle and the fragment manager. You can think of fragments as reusable building blocks for your UI, if you plan to have your application work on both phone sized devices and tablets (and eventually TVs).

### Single Pane Activity

When I write applications, I tend to first design applications with a single view in mind. That is to say, I imagine creating a single list view, or a single form view (a view with buttons, labels, etc..) that I’d want to see on a phone sized device. When I first started writing Android applications, I would have my activity’s xml layout define the entire view, and my activity would always extend ListActivity or something of the like. Then when I had to support tablet applications, I’d refactor my views to be loaded into a fragment, and then have my fragment load the view instead. This means that I went from having an Activity, with a corresponding xml layout, to having an Activity, with an xml layout which loaded a Fragment, which loads an xml layout (bringing my grand total from 2 files up to 4 files). So much boilerplate!

One trick I learned from the Google I/O source code was, create an abstract single pane activity which loads an empty xml layout, and then have any class that extends the abstract class define which Fragment to load, and have the abstract class put add the fragment to the view for me! This brings my total count down to only 3 files! Yes, I realize that I might have just saved one file, but that’s one less file to maintain, and now for every new single pane Activity that simply needs to load a fragment, this means I save having to create a separate activity xml resource just to load my fragment.

### Let’s see some code

So what does the xml look like? Here’s my activity_singlepane_empty.xml
```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/root_container"
	android:layout_width="match_parent"
	android:layout_height="match_parent" />
```

That’s it! Now I create an abstract class, which will load this xml resource and will use the transaction manager to commit the Fragment to this layout. Note that I like using generics in this situation so that I can have some type safety. Google’s application doesn’t really do this, but I say, ‘To each his own..’.

```language_java
public abstract class SimpleSinglePaneActivity<T extends Fragment> extends FragmentActivity
{
	private static final String FRAGMENT_TAG = "single_pane_fragment";
	private T mFragment;

	protected abstract T onCreateFragment();

	@Override
	protected void onCreate( Bundle bundle )
	{
		super.onCreate( bundle );
		setContentView( R.layout.activity_singlepane_empty );
		final String customTitle = getIntent().getStringExtra( Intent.EXTRA_TITLE );
		setTitle( customTitle != null ? customTitle : getTitle() );
												        
		if ( bundle == null )
		{
			mFragment = onCreateFragment();
			getSupportFragmentManager().beginTransaction()
				.add( R.id.root_container, mFragment, FRAGMENT_TAG )
				.commit();
		}
		else
		{
			mFragment = (T) getSupportFragmentManager().findFragmentByTag( FRAGMENT_TAG );
		}
	}

	protected T getFragment()
	{
		return mFragment;
	}
}
```

The onCreate method will instantiate the fragment to display, and commit it the view using the fragment manager. If the view is ever recreated, the fragment is simply reloaded from the fragment manager. In the google application, this goes a bit further by converting the intent into arguments for the fragments, but I’ll leave that out so I don’t clutter the important information. The take away here is, now I can extend SimpleSinglePaneActivity, which will return the fragment type I’m trying to show. I found this to be a nifty little trick pretty useful since now I always think in Fragments, which helps scale my application if i need to extend it to support tablets. Hopefully you’ll find this as useful as I did. Enjoy!

