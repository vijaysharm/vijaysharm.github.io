---
layout: post
title: Fluid layout splash screens
published: false
---
I need a splash screen. I have no idea how to make one. All I know are my requirements:
+ It needs to display a single line of styled text which may wrap to another line
..+ The text needs to be centered vertically and horizontally
..+ The text can be of any length
+ It needs to be responsive
+ It can use javascript or CSS3 animations to animate the text in and out
+ It can animate from either the side edge or the top edge into the middle of the screen.

What I don't know is how I'm going to do this. Where do I start? Well, I'll guess I'll start with trying to get my page to display text that's centered vertically and horizontally on the page....(fast forward 10 minutes of googling)... Well, clearly its [not][1] [obvious][2]. I'm not sure what the best way to do this is, but I'll just use the div/table method. It seems to do the trick with very little CSS. I'm sure the other techniques are semantically better, but I'll have to weigh the pros and cons later. The fact of the matter is, it centers my text, and does so in a responsive way. Four requirements down!


<p data-height="268" data-theme-id="0" data-slug-hash="Lxabj" data-default-tab="css" class='codepen'>See the Pen <a href='http://codepen.io/vijaysharma/pen/Lxabj'>Lxabj</a> by Vijay Sharma (<a href='http://codepen.io/vijaysharma'>@vijaysharma</a>) on <a href='http://codepen.io'>CodePen</a>.</p>


Now I need to figure out how to animate this thing in. Historically, the way I tend to know how I'm going to animate something is by rendering it at its start position, and then at its end position. Then I try to do the mental substraction that tells me what properties I need to change to get it from position A, to position B. The good news is I already have position B done!


To figure out what position A is, I'll first need the freedom to move the text inside the block. Since the block is defined as a table taking up 100% of the space with a single cell, the title cell will take up all the space as well, the only thing I can grab a hold of and animate are the edges of the cell. I'll do this by first giving my splash-title element a position of relative and a top of 0% and left of 0%. This still gives me my position B, but now I can play around with the top and left values using ChromeDevTools or FireBug.


<p data-height="268" data-theme-id="0" data-slug-hash="rlzgJ" data-default-tab="css" class='codepen'>See the Pen <a href='http://codepen.io/vijaysharma/pen/rlzgJ'>rlzgJ</a> by Vijay Sharma (<a href='http://codepen.io/vijaysharma'>@vijaysharma</a>) on <a href='http://codepen.io'>CodePen</a>.</p>


Note that I added overflow: hidden to the parent block so as I play around with the left and top values of the title, I don't get a scroll bar. If I set the left to 100%, the text slides all the way off screen, but I want it to slide in the from the left (just because I think it looks better), I can set the left to -100%. That means I can just animate the left position from -100% to 0, and Ill have my splash screen ready to go! I could even have it drop from the top of the screen using a starting position of -100% and an end position of 0% and voila! Since my requirements state that I can use either javascript or CSS3, I'll just demo using CSS3 transitions.


<p data-height="268" data-theme-id="0" data-slug-hash="ofAmj" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/vijaysharma/pen/ofAmj'>ofAmj</a> by Vijay Sharma (<a href='http://codepen.io/vijaysharma'>@vijaysharma</a>) on <a href='http://codepen.io'>CodePen</a>.</p>


All I need to do here is pretty things up with some custom fonts, and some color, and use a much better animation than a simple slide in (anyone try [animate.js][3]? or [rebound.js][4]? or [impress.js][5]?).


At this point you might have a lot of question, like "Why did you use the div-table method?", or "Isn't it inefficient to move a whole div around the screen than try to figure out how to move just text (saving not having to re-render a 100%x100% element and just that piece and not the whole view)". The choices I made were in the name of getting it done. From here, we can discuss alternative solutions.

[1]: http://css-tricks.com/centering-in-the-unknown/
[2]: http://designshack.net/articles/css/how-to-center-anything-with-css/
[3]: http://daneden.github.io/animate.css/
[4]: http://facebook.github.io/rebound/
[5]: http://bartaz.github.io/impress.js/