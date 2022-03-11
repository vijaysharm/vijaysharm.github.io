---
layout: post
title: Loading a million items
---

Loading a lot of items into memory can be difficult to manage. For the most part, if your view models are very slim, developers are more than willing to hold potentially a thousand objects in memory without thinking twice. 

It's a perfectly valid approach that many apps take. If you need more than a few thousand objects, maybe you'll figure out how to implement paging, and that'll be that. 

What I'm about to describe next is the approach I took to displaying up to a million objects without paging. You might be asking why wouldn't I just implement paging, or going with some other approach, sadly, I can't really go into the _why_ of my requirement, so I'll just focus on the _how_. Note that my solution is very much tailored to the data I'm working with, so this technique may not work for you.

1. Rarely changing data: For the mos part, the data I'm working with doesn't change very often. It does so maybe once a day at night while the user is not using the app. I'll dump my thoughts on how I might approach dealing with highly dynamic data towards the end of this post.
1. Fixed SQL query: My data is returned from a fixed un-parameterized SQL query. Think `SELECT * FROM <table> ORDER BY <X>`.
1. We're using RxJava: Isn't everybody? Although this technique is not tied to using RxJava, its just the tool I used to solve this particular problem.

So the approach I decided to take was to load the object on demand. What does 'on-demand' mean on this case? It means I only load the object being displayed when the user scrolls to that position. Think 'Endless Scrolling' except with a stock `ListView` and instead of pages, only one item is loaded at a time.

The first thing I needed is the number of total items. So I took the SQL query from above and added a new method to return the item count. Something like `SELECT COUNT(*) FROM <table> ORDER BY <x>`. Then I modified my `BaseAdapter` to only work with the item count:

```
public class Adapter extends BaseAdapter {
    public Adapter(Context context, int items) {
		this.inflater = LayoutInflater.from(context);
		this.items = items;
	}
...

    @Override
	public int getCount() {
		return items;
	}

	@Override
	public Object getItem(int position) {
		return position;
	}

	@Override
	public long getItemId(int position) {
		return position;
    }
...
}
```

My adapter no longer holds on to any list containing all the view objects in memory. All it has is an `int` which holds the maximum number of items that it can show. Now, in my adapter's `getView` call, I make a call to fetch my data using the position as an argument. The implementation is irrelevant, but here's the method call signature. Note that the returned `Observable` makes an asynchronous call, and returns on the Android main thread.

```
private static Observable<ViewModel> fetch(int position) {...}
```

You must now be terribly confused. How does this work? How are you able to turn a position into a view model? This is where things get a little sneaky. Recall I mentioned that my data rarely changes, so I'm comfortable assuming that the position indicates the index of the value that would have been returned from the list of result from the SQL query above. How do I get that specific object without loading the whole list again? I used `OFFSET` of course! I failed to mention that I also modified the above SQL query to resemble the following `SELECT * FROM <table> ORDER BY <x> LIMIT 1 OFFSET <position>`. With this query, I get a single object returned, indexed at some some position. When this object is returned asynchronously through the `Observable`, I update my item with the result. Effectively, I've assumed that the data's order will not change, which means it's position can be assumed to be its ID in the result set.

One of the advantages of RxJava is the ability to unsubscribe from requests which are in-flight. An optimization I made was to store this subscription in the View holder object I use in my adapter, and to cancel any pending request while binding. This means that once the user has scrolled away from a row, we stop that request and fetch the data that user is actually looking at. So roughly speaking, my adapters `getView` method looks as follows:

```
@Override
public View getView(int position, View view, ViewGroup parent) {
	if (view == null) {
		view = inflater.inflate(R.layout.item_list, parent, false);
		view.setTag(new ViewHolder(view));
	}

	final ViewHolder holder = (ViewHolder) view.getTag();
	if (holder.subscription != null)
		holder.subscription.unsubscribe();

	holder.subscription = fetch(position)
		.subscribe(new Action1<ViewModel>() {
			@Override
			public void call(ViewModel model) {
				holder.bind(model);
			}
		});

	return holder.view;
}
```

This approach required me to have a 'loading' state for my individual cells, which was acceptable to our Product Owner. Note that I also had to defer click/tap handling a cell until the object itself is actually loaded.

Finally, in my `fetch` implementation, I added caching, so that the fetch call reads from and writes to an LRU cache of objects. This means that my list has a maximum roof for the amount of memory used, while trying to give the user a more pleasant scrolling experience. I also attempted to add a Scroll Listener to the `ListView`, which would pre-fetch and populate my LRU cache for even faster view loading.

This approach ended up working really well for me. It reminds me of how android might be using the `Cursor` interface along with its `LoaderManager` pattern.

As I mentioned, I had the privilege of my data not changing from underneath me while the user is scrolling. If it did, it would yield unpredictible result. What I might consider doing if my data were constantly changing would be to store the fact that the data has changed, but not modify the backing store's data. Instead, let the user know that the data has been updated, and allow them to request a refresh (pull to refresh, some button indicator like twitter or android, etc...), merge the new data into the backing store, and then proceed as normal. This might require you staging any new incoming data, and the handling of that alone may not be worth the investmenting in this technique, but it's an option for you to consider.

Again, this approach is highly tailored to my read-only list. This technique would fail pretty misearbly if my list need to support deletion or inline additions, however, take this approach into consideration the next time you're faced with loading a number of items; Only load what you need when you need it!

To anyone asking about sample code, have a look at this [gist](https://gist.github.com/vijaysharm/17045deae91e3174190e)

