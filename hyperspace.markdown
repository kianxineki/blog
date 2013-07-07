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
write some rendering logic that looks like:

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

which will work in both node and the browser! The keys are css query selector
strings and the values are the attributes and content to update at the nodes
matching the selector. The values are objects mapping element attribute names to
values but with special keys `_html` to set inner html and `_text` to sent
entity-encoded inner text. If the value is a string, it's treated as the `_text`
parameter.

The only odd part is that in the browser you will get a full dom element, but in
node you just get an object with an `outerHTML` string property, which is
adequate for writing html content to the http server response. 

## hyperspace

[hyperspace](https://github.com/substack/hyperspace) puts a stream on top of
hyperglue and adds some browser-specific functionality so that you can write
shared rendering logic that starts on the server and seamlessly picks up where
the server left off on the browser.

With hyperspace we can write a shared render.js that looks like:

```
var hyperspace = require('hyperspace');
var fs = require('fs');
var html = fs.readFileSync(__dirname + '/article.html');

module.exports = function () {
    return hyperspace(html, function (row) {
        var name = doc.title.replace(/[^A-Za-z0-9]+/g,'_');
        return {
            '.title a': {
                name: name,
                href: '#' + name,
                _text: doc.title
            },
            '.commit': doc.commit,
            '.author': doc.author,
            '.date': doc.date,
            '.body': { _html: doc.body }
        };
    };
}
```

Now when we `require('./render.js')` in either the browser or in node, we get a
function that returns a through stream. The stream expects json data to be
written to it (either naked objects or newline-separated json text) and outputs
a stream of html that can be piped directly to an http server response:

```
var http = require('http');
var fs = require('fs');
var render = require('./render.js');

var server = http.createServer(function (req, res) {
    res.setHeader('content-type', 'text/html');
    fs.createReadStream(__dirname + '/data')
        .pipe(render())
        .pipe(res)
    ;
});
server.listen(5000);
```

Here we're just piping newline-separated json from a file, but it should be very
simple to swap that part out for a real database when you need one.

In the browser, `hyperstream()`'s return value is still a stream, but that
stream has some more browser-appropriate functions on it:

```
var render = require('./render.js')();
render.on('element', function (elem) {
    elem.addEventListener('click', function onclick () {
        elem.classList.remove('summary');
        elem.removeEventListener('click', onclick);
    });
});
render.appendTo('#articles');

var shoe = require('shoe');
shoe('/article-stream').pipe(render);
```

Here we're using the `'element'` event to bind a click listener on every article
element. The `'element'` listener also fires for elements that were pre-rendered
server-side once the page loads.

Any new data that comes down the pipe from
[shoe](https://npmjs.org/package/shoe) (which hasn't yet been wired
up server-side in this example) will get added to the rendering automatically.

Instead of `.pipe()` which `render` still has in the browser, we're using
`.appendTo()` to put content into the `#articles` element. Using `.appendTo()`
here will insert rendered elements as they are written to the render stream and
it will fire the `'element'` events for any elements that were rendered
server-side.

It's also possible to give a sorting function to hyperspace for browser code,
which is really useful when you've got a realtime feed in conjunction with
on-demand loading so that your server code can be dumb and simply write realtime
updates and requested on-demand data directly to the same data stream without
adding any extra transformations.

## trumpet

Going back to the server code example, we'll just have article elements on the
page and no containing `<html>` or `<body>` elements to wrap the article
content.

We can use trumpet to pipe the stream that hyperspace returns into some existing
html file:

```
var http = require('http');
var fs = require('fs');
var trumpet = require('trumpet');
var render = require('./render.js');

var server = http.createServer(function (req, res) {
    res.setHeader('content-type', 'text/html');
    
    var tr = trumpet();
    fs.createReadStream(__dirname + '/data')
        .pipe(render())
        .pipe(tr.select('#content').createWriteStream())
    ;
    fs.createReadStream(__dirname + '/index.html').pipe(tr).pipe(res);
});
server.listen(5000);
```

Here we're streaming the rendered html into an element named `#content` and
piping the index html file with rendered data at `#content` to the response.

# stream everything

By using streams with hyperspace and trumpet, we get the benefit of APIs that
compose well together, but streams are easy to serialize and make it easy to
route data over a different transport or to a different destination.

Some new database APIs even have
[streaming realtime feeds](https://github.com/dominictarr/level-live-stream)
built-in to listen for
[live updates](http://guide.couchdb.org/draft/notifications.html).

[LevelDB](https://github.com/rvagg/node-levelup) is particularly fascinating
because you can browserify most of the modules and use the same database
interfaces backed to IndexDB when you're in a browser.

# links

* [browserify](http://browserify.org)
* [brfs](https://github.com/substack/brfs)
* [hyperglue](https://github.com/substack/hyperglue)
* [hyperspace](https://github.com/substack/hyperstream)
* [trumpet](https://github.com/substack/node-trumpet)
* [levelup](https://github.com/rvagg/node-levelup)
* [level-live-stream](https://github.com/dominictarr/level-live-stream)
