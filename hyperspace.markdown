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

I'm not a big fan of html templates since you've got to nest a pseudo-language
into your html and I would rather just write ordinary html that I can update
procedurally from my rendering code.

In browsers it's easy to update the html on the page with the DOM. I can just do
a quick `.querySelector()` to fetch an element and then I can easily update
attributes or inner content.

If I've got a lot of data to insert into the DOM, calling `.querySelector()`,
`.setAttribute()`, and assigning `.innerHTML` or `.textContent` all the time can
get verbose, but the approach is not so unpleasant.

In node, there's not a fast, reliable DOM library for doing updates with the
intent of producing html strings that will be sent to the browser in the initial
payload. Luckily, the full DOM isn't strictly necessary to serve html strings to
the browser in this "dom style".

## hyperglue

With [hyperglue](https://github.com/substack/hyperglue), we can solve both the
verbosity of updating the DOM at query selectors and node compatability at
once.

hyperglue takes an html element or string and an object that maps query
selectors to attributes and content. With hyperglue and brfs together you can
write some rendering logic like:

```
var hyperglue = require('hyperglue');
var fs = require('fs');
var html = fs.readFileSync(__dirname + '/article.html');

module.exports = function (doc) {
    var name = doc.title.replace(/[^A-Za-z0-9]+/g,'_');
    return hyperglue(html, {
        '.title a': {
            name: name,
            href: '#' + name,
            _text: doc.title
        },
        '.commit': doc.commit,
        '.author': doc.author,
        '.date': doc.date,
        '.body': { _html: doc.body }
    });
}
```

which will work in both node and the browser!

The only odd part is that in the browser you will get a full dom element, but in
node you just get an object with an `outerHTML` string property, which is
adequate for writing html content to the http server response. 

## hyperspace

[hyperspace](https://github.com/substack/hyperspace) puts a stream on top of
hyperglue and adds some browser-specific functionality so that you can write
shared rendering logic that starts on the server and seamlessly picks up where
the server left off on the browser.




hyperspace has 'element' event that fires for each new element on the page



-> streaming, so you can pipe into trumpet


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

