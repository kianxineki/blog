So you're using the same language on the browser and the server. Great!

You can get the benefits of fast initial page loads server-side that are easily
indexed by search engines while simultaneously rendering realtime and on-demand
content browser-side for a rich, responsive user experience!

This sounds really great, but now you might be thinking:

* How do I load the shared code in both environments without making a mess?
* How can I load files like html or templates in a way that works in both node
and the browser?
* How do I render the data?
* How should I route the data where it needs to go?

These questions are not obvious and there are many ways to answer them!
The rest of this article is some answers that I've discovered or built that work
well together.

# import shared code

Node already has a really good built-in way of importing and exporting code with
`require()` and `module.exports`.

[browserify](http://browserify.org) makes `require()` and `module.exports` work
in the browser pretty much exactly the same as they work in node.

By using node-style modules, our shared rendering logic can just use
`module.exports` to expose a function and we can use `require()` to load other
project files or even [npm modules](https://npmjs.org).

Let's stub out the shared render file, render.js:

```
module.exports = function () {
    // shared logic goes here
};
```

# load files

The next thing we'll need to figure out is how our shared render logic should
load the non-js files it will need into memory.

In node, to load something into memory at process start-up, it's common to use
`fs.readFileSync(filename)` to synchronously return the file contents at
`filename`.

[brfs](https://github.com/substack/brfs) can make `fs.readFileSync()` work for
browser code too! Instead of performing synchronous IO when the program runs,
instead at compile time the file contents are inlined into the bundle.

For example if we have some code:

```
var fs = require('fs');
var src = fs.readFileSync(__dirname + '/file.txt');
```

and if `file.txt` is just the string "beep boop\n", after running browserify
with brfs, our file will be turned into:

```
var fs = require('fs');
var src = "beep boop\n";
```

To run browserify with brfs, just use the `-t` switch:

```
$ browserify -t brfs main.js > bundle.js
```

# render the data

Use ordinary html.

## hyperglue

works in both node and the browser

In the browser, you get full DOM nodes.

## hyperspace

-> streaming, so you can pipe into trumpet

hyperspace has 'element' event that fires for each new element on the page

## trumpet

# stream everything

trumpet and hyperspace are already streaming

Streams are easy to serialize.
Streams make it easy to shuffle the same data to a new destination.
You can use the same logic for rendering the initial 



For applications that make use of databases with
[streaming database APIs](https://github.com/dominictarr/level-live-stream)
to listen for [live updates](http://guide.couchdb.org/draft/notifications.html)
this 

