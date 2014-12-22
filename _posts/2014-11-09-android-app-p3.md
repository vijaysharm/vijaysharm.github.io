---
layout: post
title: Writing an Android app - Youre using ContentProviders wrong
published: false
---
Content providers are useful. They idea of having an abstraction around your data source makes lots of sense from good programming perspective. The problem is, the ContentProvdider falls short of being useful. To me, it often feels like the Android's Loader system was only ever designed to work with CusorLoaders and ContentProviders. Anything else is really up to you to figure out. Under the hood, CursorLoaders subscribe themselves to a ContentObservers tied to your data before returning the Cursor. If someone adds/updates/deletes data through a _properly_ implemented ContentProvider will benefit from automatic reloading of the cursor. Suddenly, your view is updated without you doing very much.

Sweet!

But what if you wanted to know what data changed? What if you wanted to implement some sweet list view animation of the changed data? How do you do this????

After some mulling over, I've come to the conclusion that you don't need 