---
layout: post
title: "Efficiently Detecting When An Element Changes Size"
date: 2017-08-20 12:36:49 +0100
comments: true
categories: [HTML, JS, FED]
permalink: /efficiently-detecting-when-an-element-changes-size
draft: false
---

While working on [MediaFaker](https://github.com/Decad/MediaFaker), an experimental idea for displaying the effects of media queries inside any DOM node without the use of iframes, I needed a method of detecting when an element changes size. My initial thoughts lead me to a pretty naive solution involving [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame).

```javascript
function hasChangedSize(rect1, rect2) {
  return rect1.width != rect2.width || rect1.height != rect2.height;
}

function watchForResize(element, cb) {
    var startingSize = element.getBoundingClientRect();

    function checkForRezize() {
        var currentSize = element.getBoundingClientRect();
        if(hasChangedSize(startingSize, currentSize)) {
            startingSize = currentSize;
            cb();
        }
        requestAnimationFrame(checkForRezize);
    }
    requestAnimationFrame(checkForRezize);
}
```

For a rough prototype this method worked but I knew that this would not scale well when watching many elements. This method could be optimised to remove additional requestAnimationFrame calls when watching more than one element but it would still be making calls to [getBoundingClientRect](https://developer.mozilla.org/en/docs/Web/API/Element/getBoundingClientRect) each frame which is on the [naughty list](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) of things to avoid causing layout / reflow. As you can see from one of my tests with 50 elements there is a lot of work being done 60 times a second.

{% img /images/posts/expensiveRaf.png %}

What I wanted was a method that wouldn't require the use of `requestAnimationFrame` or `getBoundingClientRect`. So I did what all good developers would do, ~~I thought long and hard about the problem and used my extensive domain knowledge to come up with a solution that's simple and elegant.~~ I googled it. I found an extremely interesting approach by [Marc J. Schmidt](https://github.com/marcj) in his CSS-Element-Queries polyfill. The [ResizeSensor](https://github.com/marcj/css-element-queries/blob/master/src/ResizeSensor.js) module uses only native scroll events to detect when an element changes size, How I thought?

The method that the ResizeSensor uses works by adding hidden children to the element.

```HTML
<div class="resize-sensor" style="position: absolute; left: 0px; top: 0px; right: 0px; bottom: 0px; overflow: hidden; z-index: -1; visibility: hidden;">
    <div style="position: absolute; left: 0px; top: 0px; transition: 0s; width: 10000px; height: 10000px;"></div>
</div>
```

These elements consist of a containing div, positioned absolutely covering the entire parent element. As well as a child element that is also positioned absolutely however this child has its height and width set to 10000px making its huge.

How does this help us determine if an element has changed size though? The massive child element overflows its parent causing the parent to have scrollbars, these are hidden with overflow auto.

This is where the solution gets clever, once the hidden element are added to the DOM the javascript sets the parents scroll offset (`scrollLeft` and `scrollTop` properties) to 10000 effectively scrolling all the way to the end. The actual scroll offset set wont actually be 10000 though because the maximum scroll offset is `10000px - elementSize = maximum scroll offset`. Noticing this formular can tell what will happen when the element changes size.

When the element changes size so does the scroll offset because we are already at the maximum, which in turn triggers an onScroll event!

Pretty neat.
