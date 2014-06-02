---
layout: post
title: Why I wrote Ehyo
---
I recently [published][1] a project I've been working on for the last couple of months. I figured a blog post about it seemed reasonable. 

### What is Ehyo?
As per the description, Ehyo is a command-line-tool to help with Android development. It allows you to quickly add permissions, manage dependencies, and work with templates.

### Why did you write it?
In March 2014, I was watching a dev byte on youtube about the big cookie model while  vacationing in Mexico. Being a lazy delveloper with terrible memory, I realized that I would never remember all the steps I'll ever need to reproduce that same model from project to project. Moreover, I really didn't want to have to refer to some manual or wiki page everytime I wanted to have so much boilerplate to my application. So I hit the interwebs looking for a solution.

After much googling, I didn't find anything that fit my needs. The templates that Google provide with IDE do a lot of what I was looking for, but the scope of the templates leaves a lot to the imagination. Jeff Gilfelt provides a [nice library][2] that covers a bit more ground, but in my opinion, there are so many more templates that can be useful. Moreover, where were the templates that make use of third-party libraries? How about a starter project that lets you add depenency injection like dagger, or android annotations? What about testing frameworks like Espresso, or Robolectric? How do I add those to my project without having to google the right answer everytime on every project?

The other thing I wanted was a *dry-run*-like feature for applying templates. I have no idea why, but I like to know what the result of applying some magical template to my project results in. True, I could just use my version control tool to help give me the diffs, but I'm a control freak like that. I'm a bit of a command line junkie, and wanted a tool that would let me apply these templates without relying on the IDE.

Finally, I'm not just an Android developer, but I share half my time doing web development, and the inspiration for Ehyo cam after I'd started using [yeoman][4] as my scaffolding application about a year ago. I found myself really addicted to it! It does such a simple thing in such a unobtrusive way, that I kept asking myself why there was no similar tool for Android...

### Birth of Ehyo
Ehyo is playfully named after the previous meantioned yeoman tool. Moreover, being Canadian, I thought adding the 'eh' in front made it very appropos. 

I got to writing ehyo as a means of appeasing my OCD for the above. Ehyo was developped to address the small hole I thought existed in Android development. But the reality is, everything Ehyo does can be done using different tools. It is the epitome of "I have a problem, I know, I'll write X. Now I have 2 problems". However it does what it does fairly well. Give it a template, and it will apply it like nobody's business. Wanna know what the template does? simply add a '-n' to the end of your command and you get a diff of what would have happened.

A feature that sort of *fell out* of working on this project was the ability to manage the depenencies in my project. Chris Broadfoot has a very convinient [web page][3] that let's you just search for a library you want to add to your project. Adding templates requires you to add dependencies to your build file, that I figured it was a short step to just copy what Gradle Please was doing in Ehyo. The one advantage that Ehyo has is that you can do wholesale upgrades of your dependencies, and you can choose which flavor you want the dependency to be applied to. 

### Ewww, why is it written in Java?
I choose Java for no good reason as my language of choice. Firing up a JVM for something as silly as applying a template to a project is overkill. Ultimately it'll probably be the reason that ehyo will never see any kind of mass popularity, but that's not what I was gunning for.

### What's the future of Ehyo?
Personally, I'd love to see Ehyo gather momentum with more templates. But not just templates, *smart* templates. Templates that can understand the context under which they are being applied. However to do that, Ehyo would have to establish some conventions for applications that are written, and would have to model Java classes in a way that allows it to understand some of its context while applying the template. I don't think that's completely out of reach, its just a limitation of my domain knowledge. IDEs today can do a lot of this stuff, its just a matter of marrying the two concepts together.

As for Ehyo in the near future, I plan on writing more templates that can be used with either Ehyo, or the Android IDE, but that focus on using more advanced 3rd party libraries. I'd love to see Ehyo interact with a repository of templates and working within the context of that template for adding component. Does your application use Flow/Mortar? Why not a template that will automatically create a presenter/view for you? Using Robolectric for testing? Why not automatically create a shadow if need be?

I'd love to also work on creating that repository source, so users don't have to download templates. The easiest solution to this would be to integrate with github repositories or gists in some way. This is definitely high on my list. 

Finally, I'd love to create a tool that can automatically create a template for you based on some diff of your current project. The wonderful [androidsnippets][5] exists today, and I believe all we really want, is the ability to apply those snippets to our project, but spend too much time trying to figure out how. Imagine you've just spent a day trying to figure out how to get your project to push AARs to maven. You dont want to have to go through the same madness on every project you create. You should be able to generate a patch, turn that patch in to a template and push that the above repository. Next time you want to apply that template, it should just be a single command away. 

### Are you done talking?
Yes. Well, no... In theory, I think Ehyo has a lot of promise. In practise, it'll probably be DOA. If you have any interest in the project, feel free to message me, or send pull requests. The project was developed entirely in isolation and so it might just be a product of my insanity. However, I'm overly welcome to open conversations about this project or any others that are like it.

Now I'm done talking :)

 [1]: https://github.com/vijaysharm/ehyo
 [2]: https://github.com/jgilfelt/android-adt-templates
 [3]: http://gradleplease.appspot.com/
 [4]: http://yeoman.io/
 [5]: http://www.androidsnippets.com/