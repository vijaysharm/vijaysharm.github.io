---
layout: post
title: Animanted responsive grid menu
---
I was working on a list type view for a mobile web application. I not only wanted to display a list of items in a grid, but I wanted the items to animate into view in some fancy way. What I settled on was a staggered slide-fade into screen. That means, that each item in the list will load one after the other from either the bottom or the right from their expected end position. The only other requirement I was looking for was that the items should be responsive to the size of the page. They should resize according to the page width, and it should be rather easy to change the number of columns dislayed in the grid. So I start off with a list of item and some base styling.

```
<div class="outer-container">
  <div class="container">
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div>
    <div class="item">item</div> 
  </div>
</div>
```


The styling is pretty straight forward, Just a bunch of cells with some text. You can pretty much put anything in each cell. What is important here is that each cell is a percentage width. I've set it to 31%, giving me three columns, but can easily be changes to more columns, or responsively changed based on media queries, so I can have more columns on desktop, and less columns on mobile.


```
.outer-container {
  width: 100%;
}

.container {
  width: 100%;
  overflow: hidden;
}

.item {
  background-color: tomato;
  width: 31%;
  height: 100px;
  border: dotted #BB1 1px;
  text-align: center;
  float: left;
}
```


Now for the animation. The items themselves will animate on screen from the bottom. What's important here is the fact that I marked the container's overflow as hidden, and the item's position as relative. The combination of the two allows me to play with the position of the item from their natural position. I can move the items down with a start position of 200px (chosen randomly), and start opacity of 0. This means that they will not be displayed, and I'm free to animate them on to screen. My CSS now looks something like this:


```
.outer-container {
  width: 100%;
}

.container {
  width: 100%;
  overflow: hidden;
}

.item {
  position: relative;
  top: 200px;
  opacity: 0;
}

.item {
  background-color: tomato;
  width: 31%;
  height: 100px;
  border: dotted #BB1 1px;
  text-align: center;
  float: left;
}

.top {
  top: 0px;
  opacity: 1;
}
```


Now I want the menu to animate in when the document finishes loading, so I'll use some javascript to do the trick. I've also added an on-click callback to the container so you can play with the animation to see how it behaves. In terms of the animation, I wanted that staggered feel where each item came in from the bottom of the screen one after the other. I decided to this in JavaScript because this allows me to add a new row, or take away a row and still achieve the same effect. My javascript now looks as follows:


```
$(document).ready(function() {
  $('.item').each(function(key, value) {
    $(value).css({
      transition: 'all 1s ease',
      'transition-delay': (0.1 * key) + 's'
    });
  });
  
  $('.item').toggleClass('top');
});

$('.container').on('click', function(e) {
  $('.item').toggleClass('top');
});
```


Here, I used the CSS3 transition to do the animations. I could have used the jQuery animate method, but decided on the CSS3 animations so I could take advantage of whatever hardware acceleration the mobile browser this would run on. Does jQuery revert to CSS3 under the hood? I'm not sure. In either case, this was a trade-off I was happy to live with. Discussions are welcome. The final product looks as follows:


<p data-height="268" data-theme-id="0" data-slug-hash="FAmBb" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/vijaysharma/pen/FAmBb'>FAmBb</a> by Vijay Sharma (<a href='http://codepen.io/vijaysharma'>@vijaysharma</a>) on <a href='http://codepen.io'>CodePen</a>.</p>