---
layout: post
title: Writing an Android app - Offline First
published: false
---
Offline first is a real thing. It's a hard thing, but its a real thing.

Strategies
1. Always write to local disk. Don't assume you can always update directly using a network call.
1. Let your server be the source of truth for your data. If you make a change locally, that's in conflict with the server, the server should always win. Similarly, the server needs to deal with bad data from the a client. Never trust incoming data. Use it as a suggestion for how the state of the object should be modified, and update it accordingly.
1. Forget REST when it comes to mobile. The idea may work with web pages, but the strategy is poorly suited for a mobile experience. You only need two types of calls, GET and POST. One to get ALL data, and one to push ALL data.
1. You need a way to store the order of events your device is producing. You can use the file system to serialize your changes, send them up to your server in the same order, but don't remove them off disk until its been handed off.
1. Your server needs to be idempotent with respect to the changes you send up.
1. Your GET needs to take a timestamp which tells the server to return all the data that's been updated since this timestamp. This is telling your server, my data is as fresh as of this timestamp, what's changed?
1. Let your device use its persistence layer as its REST endpoint. You can devise an interface to which your UI talks to your persistence layer using something that looks like REST (POST to create objects, PUT to update, etc...). Behind the scenes, the service will store these as events to send up to the server (i.e. deletes don't actually delete data, they should mark them as deleted. Your GET from the server should tell you when to delete something)

serialize GET request items on device, and execute them serially. You have to take the server's response as gospel. If the server tells you that the state of the object is X, but you have local state that says Y, you have to accept X and deal with it appropriately.
