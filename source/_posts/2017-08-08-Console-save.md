---
layout: post
title:  "Save .json files from chrome developer tools"
date:   2017-08-07 16:16:01 -0600
permalink: /save-json-file-from-chrome-developer-tools/
---
I spend a lot of my time inside of chrome developer tools and using the javascript console trying things out and testing various things I am working on. The other day I wanted to persist some data that I had built up while playing around in chrome javascript console but I couldn't find an easy way other than `JSON.stringify` and copy&paste. So I decided to extend the global [console](https://developer.mozilla.org/en/docs/Web/API/console) object to include a `save` method that would save any object I passed to it as a json file. Here is the code I wrote:

```javascript
    (function(console){
      console.save = function(data, filename){
        if(!data) {
          console.error('Console.save: No data')
          return;
        }

        if(!filename) filename = 'console.json'

        if(typeof data === ""object""){
          data = JSON.stringify(data, undefined, 4)
        }

        var blob = new Blob([data], {type: 'text/json'}),
            e    = document.createEvent('MouseEvents'),
            a    = document.createElement('a')

        a.download = filename
        a.href = window.URL.createObjectURL(blob)
        a.dataset.downloadurl =  ['text/json', a.download, a.href].join(':')
        e.initMouseEvent('click', true, false,
                         window, 0, 0, 0, 0, 0,
                         false, false, false, false, 0, null)
        a.dispatchEvent(e)
      }
    })(console)
```

But I didn't feel like copy and pasting this code into my console every time I wanted to use it. So I threw together a chrome extension to inject this code into every page. I put all this code up on [github](https://github.com/Decad/Console.save) check it out.

If you found this method useful Console.save is now part of [devtools-snippets](https://github.com/bgrins/devtools-snippets) which contains even more useful snippets for chromes developer tools.
