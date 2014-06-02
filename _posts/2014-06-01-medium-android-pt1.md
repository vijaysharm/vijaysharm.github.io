---
layout: post
title: Medium Sized Android Application, Part 1&#58; Setting the stage
published: false
---
*In this series, I'll be documenting my journey towards publishing a medium-sized Android application.*

### Description
The android application I will be creating is for an existing backend REST service I wrote called Apuesta. The backend code can be found [here][1]. The service was used by my friends and me to help choose NFL teams we thought would win on any given week. It's a pretty simple backend but provides REST endpoints that perform the following:

1. User logins
1. Fetching the current week's list of games including picks and scores
1. Fetching selections made by groups of users
1. Allowing users to select their choice for winning team

### UI
I'd already written a web version of the UI, which I'll use as a basis for the android application. It doesn't really ahere to the Android design guidelines, but I'll try to adjust it as we go along. I'm more interested in animating views than I am about conforming to the design guidelines. If I find myself fighting the platform more than I am able to make progress, than I'll reconsider my design.

### Technology Stack
I'm going to use Android Studio and Gradle for my IDE and build environment all running on OSX Maverick. I'll also try playing around with some new libraries that I've had my eye on. Some of those libraries include

1. [Mortar][2]
1. [Volley][5]
1. [RxJava][4]
1. [Picasso][3]
1. [Espresso][6]
1. [Robolectric][7]
1. [Genymotion][8]

I'm also hoping to employ some of the debug dev tools (through a debug drawer) that are described in [U2020][9]. Finally, I'll be targeting newer versions of Android. I'm thinking Jelly bean (version 16), but I may jump straight to Kitkat since I'm more interested in developing the application, than I am about supporting a large user base.

### Motivation
So why am I doing this? The short answer? Just for the heck of it. There's a lot of information out there on writing android applications, and I'm just hoping to document yet another means for creating one. I'm hoping this turns out to be beneficial to others looking to write a pretty standard CRUD-like application with some thought about good software architecture.

### Creating the project
After installing Android Studio, I went ahead and created a new blank android application. No activity, just the manifest with an application element block, and all the usual things that come with autogenerating a project.

[1]: https://github.com/vijaysharm/apuesta
[2]: https://github.com/square/mortar
[3]: https://github.com/square/picasso
[4]: https://github.com/netflix/rxjava
[5]: https://github.com/mcxiaoke/android-volley
[6]: https://code.google.com/p/android-test-kit/
[7]: https://github.com/robolectric/robolectric
[8]: http://www.genymotion.com/
[9]: https://github.com/jakewharton/u2020
