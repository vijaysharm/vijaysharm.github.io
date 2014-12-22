---
layout: post
title: Mobile App Architecture
published: false
---
# In the beginning

I have an app. Like many of you out there, I've written my share of apps following the platform guidelines, trying to adhere to all the best practices they describe in their sample app. But I'm never happy with the outcome of my code. The app works, but when I come back to it after a few days/weeks/months, I'm always turned unimpressed with the way its structured, or orgaznied from a code perspective.

In this article, I hope to take a common app structure consuming a REST endpoint which requires authentication. The plan is to iterate on it and explore refactoring the code in a way that makes me a feel a little less dirty. Although I'll be using Android as my platform of choice, I hope to describe an aritecture that can be used across any mobile platform. I'll look at things like reactive programming, and how you can use it practically in your application. Offline first, as design guideline for improving your user experience. Testing, and MV(Whatever) design.

# What do we have to work with

On both Android and iOS, most applications often get written in a way where the controller ends up being this monsterous file with 5000+ lines of code tangled up with business logic, UI animation, asynchronous network calls, etc... If you've ever seen any of the Fragment or Activity classes in Google's own I/O, then you know what I'm talking about. How can you possibly improve this?

# Understanding your object graph

On Android, every application has an Application instance. It's a singleton and is often used as a storing ground for objects that need to survive Android's Activity/Fragment lifecycle. There are other techniques for working with lifecycle management, but creating an object at the same level as your application, and then accessing it from your activity/fragment is a common practice for business objects. The other strategies are really on a case by case basis. To have your UI state survive the lifecycle, the Bundler is used during the onSaveInstanceState/onCreate pair. Deferring long running tasks to Android Services is another technique. 

# Login Screen