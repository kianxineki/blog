Announcing <a href="https://github.com/substack/node-browserify">browserify</a>,
a node.js module that bundles up all your javascript into a single file so you
can use node.js-style <span class="code">require()</span> from the browser.
</p>

<p>
Check this:
</p>

<p>server.js</p>
<code>
var connect = require('connect');
var server = connect.createServer();

server.use(connect.static(__dirname));
server.use(require('browserify')(__dirname + '/js'));

server.listen(9797);
</code>

<p>js/foo.js</p>
<code>
var bar = require('./bar');

module.exports = function (x) {
    return x * bar.coeff(x);
};
</code>

<p>js/bar.js</p>
<code>
exports.coeff = function (x) {
    return Math.log(x) / Math.log(2) + 1;
};
</code>

<p>index.html</p>
<code>
&lt;html&gt;
&lt;head&gt;
    &lt;script type="text/javascript" src="/browserify.js"&gt;&lt;/script&gt;
    &lt;script type="text/javascript"&gt;
        var foo = require('./foo');
        
        window.onload = function () {
            document.getElementById('result').innerHTML = foo(100);
        };
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    foo = &lt;span id="result"&gt;&lt;/span&gt;
&lt;/body&gt;
&lt;/html&gt;
</code>

<p>
Then fire up the server (<span class="code">node server.js</span>) and load up
http://localhost:9797/
</p>

<p><img src="/images/browserify/foo.png" width="456" height="144"
style="border: 1px red solid"></p>

<p>
Amazing! Browserify just hosted all of those files at
<span class="code">/browserify.js</span> and made
<span class="code">require()</span> work just like in node.js!
</p>

<p>
But wait! There's more! You can specify npm modules to use and then you can
<span class="code">require()</span> those too! Just pass the "require" parameter
to browserify:
</p>

<p>server.js</p>
<code>
var connect = require('connect');
var server = connect.createServer();

server.use(connect.static(__dirname));
server.use(require('browserify')({
    require : [ 'traverse' ]
}));

server.listen(9797);
</code>

<p>
Then spew out some HTML:
</p>

<p>index.html</p>
<code>
&lt;html&gt;
&lt;head&gt;
    &lt;script type="text/javascript" src="/browserify.js"&gt;&lt;/script&gt;
    &lt;script type="text/javascript"&gt;
        var Traverse = require('traverse');
        
        window.onload = function () {
            var obj = { a : 1, b : [ 2, 3, 4 ], c : [ { d : 5, e : 6 } ] };
            var leaves = Traverse(obj).reduce(function (acc, x) {
                if (this.isLeaf) acc.push(x);
                    return acc;
                }, [])
            ;
            document.getElementById('result').innerHTML = leaves.join(', ');
        };
    &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;span id="result"&gt;&lt;/span&gt;
&lt;/body&gt;
&lt;/html&gt;
</code>

<p>
Then fire up the server and check http://localhost:9797/
</p>

<p><img src="/images/browserify/npm.png" width="457" height="143"
style="border: 1px red solid"></p>

<p>
Success! It pulled out the leaves from that pesky nested data structure using
<a href="http://github.com/substack/js-traverse">traverse</a>!
If the npm modules have dependencies, those dependencies will get bundled along
recursively!
</p>

<p>
Plus, browserify looks for the "browserify" field in
<span class="code">package.json</span> files up on
<a href="http://npmjs.org/">npm</a>, so you can serve up custom browser versions
of packages. Here's the "browserify" part of
<a href="https://github.com/substack/dnode">dnode</a>'s package.json:
</p>

<code>
    "browserify" : {
        "name" : "dnode",
        "main" : "./browser/index.js",
        "base" : "./browser",
        "require" : [ "dnode-protocol" ]
    },
</code>

<p>
With this field in the package.json and "dnode" in the "require" list,
<span class="code">require('dnode')</span> serves up the browser-side dnode
code!
</p>

<p>
Here are some packages that have been tested with browserify:
</p>

<ul>
    <li><a href="https://github.com/visionmedia/jade">jade</a></li>
    <li><a href="https://github.com/documentcloud/backbone">backbone</a></li>
    <li><a href="https://github.com/creationix/step">step</a></li>
    <li><a href="https://github.com/jmars/jquery-browserify">jquery-browserify</a></li>
    <li><a href="https://github.com/substack/dnode">dnode</a></li>
    <li><a href="https://github.com/substack/node-seq">seq</a></li>
    <li><a href="https://github.com/substack/js-traverse">traverse</a></li>
</ul>

<p>
And that's not all!
</p>

<img src="/images/browserify/maysbot.png" width="296" height="406">

<ul>
    <li>
        Browserify bundles with
        <a href="https://github.com/kriskowal/es5-shim">es5-shim</a>
        by default so you can use shiny new javascript features like
        <span class="code">Function.prototype.bind()</span>
        even if your browser doesn't have those yet!
    </li>
    <li>
        You can specify post-filters on the bundled source to run your code
        through javascript minifiers!
    </li>
    <li>
        If you have
        <a href="https://github.com/jashkenas/coffee-script">.coffee</a>
        files in your base directory they are parsed automatically so you can
        <span class="code">require()</span> them too!!
    </li>
    <li>
        Browserify comes with compatability wrappers so you can
        <span class="code">require('events').EventEmitters</span>,
        <span class="code">require('path')</span>,
        and <span class="code">process.nextTick()</span>
        just like in node.js, to name a few!
    </li>
</ul>

<p>
What an amazing amount of features!
<a href="https://github.com/substack/node-browserify">Check it out on github</a>!
</p>

<p>
Want to make sure your browserified code runs on all the browsers?
Check out my startup, <a href="http://browserling.com">browserling</a>!
Cross-browser testing from your browser!
</p>

<img src="/images/browserify/browserify.png" width="485" height="403">
