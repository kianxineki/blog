Everybody has their own PubSub module for node.js these days and so can you!
First we'll make a PubSub for plain-old network sockets and then we'll retrofit
the code to talk websockets too.
</p>

<p>
<b>Updated March 27, 2011 for dnode 0.6 and
<a href="https://github.com/substack/node-browserify">browserify</a>
</b>
</p>

<p>
If you want to run the code in this article for yourself, you'll need
<a href="http://nodejs.org/">node.js</a> and
<a href="https://github.com/substack/dnode">dnode</a>.
DNode is an RPC library that I wrote to make network programming awesome.
It transparently wraps nested callbacks you supply in arguments to remote
methods so that when you call a wrapped function, you're actually
making an RPC call back to the other side of the connection! And the arguments
to that wrapped function are also transparently wrapped!
</p>

<p>
Now then, let's build a PubSub with dnode!
</p>

<p>
First let's write a publish() function that just prints out the arguments that
it gets called with.
To make sure it works, we can set an interval to call publish() with the string
'data' and a random number from 0 to 99, inclusive.
</p>

```
// spew.js

function publish (ev, n) {
    console.log(ev + ': ' + n);
}

setInterval(function () {
    var n = Math.floor(Math.random() * 100);
    publish('data', n);
}, 1000);
```

<p>
Then we can run it to make sure it works:
</p>

```
$ node spew.js 
data: 55
data: 91
data: 92
data: 64
data: 89

^C
```

<p>
Yep! It prints out a random integer from 0 to 99 inclusive every second as
expected.
</p>

<p>
Now we can write a simple dispatch for publish().
A hash <span class="code">subs</span> will map unique subscription IDs to emitter functions.
Using the
<a href="http://substack.net/posts/c393f3">hash library</a> (now called hashish)
from my last blog post to iterate over <span class="code">subs</span>, the arguments to
publish() are just passed along to each emitter function.
</p>

<p>
To test publish() now, I'll just hard-code <span class="code">subs</span> with two
subscriptions.
</p>

```
// pub.js

<b>var Hash = require('hashish');
var subs = {};</b>

function publish () {
<b>    var args = arguments;
    Hash(subs).forEach(function (emit) {
        emit.apply(emit, args);
    });</b>
}

setInterval(function () {
    var n = Math.floor(Math.random() * 100);
    publish('data', n);
}, 1000);

<b>// for testing purposes:
subs.bar = function (ev, n) { console.log('bar.' + ev + ': ' + n) };
subs.baz = function (ev, n) { console.log('baz.' + ev + ': ' + n) };</b>
```

<p>
Now let's run <span class="code">pub.js</span>:
</p>

```
$ node pub.js
bar.data: 67
baz.data: 67
bar.data: 61
baz.data: 61
bar.data: 10
baz.data: 10

^C
```

<p>
Every second, both <span class="code">bar</span> and <span
class="code">baz</span> subscriptions print a
message. Looks good!
</p>

<p>
Now we can take out the test subscriptions and add a super-simple stub of a
dnode server.
</p>

```
// stub.js

var Hash = require('hashish');
var subs = {};

function publish () {
    var args = arguments;
    Hash(subs).forEach(function (emit) {
        emit.apply(emit, args);
    });
}

setInterval(function () {
    var n = Math.floor(Math.random() * 100);
    publish('data', n);
}, 1000);

<b>var dnode = require('dnode');
dnode(function (client, conn) {
    // ...
}.listen(5050);</b>
```

<p>
Now dnode will listen on port 5050 with an empty handler function.
The second argument to the handler, <span class="code">conn</span> is an
EventEmitter that represents the status of the connection so you can listen for
changes in its status.
</p>

<p>
The first argument to the handler, <span class="code">client</span>, is
a transparently wrapped reference to the dnode object of a client that connects.
<span class="code">client</span> is just {} initially, an empty object, until after the
function executes but you can listen for <span class="code">conn</span>'s "ready" event or
use a reference in functions that you define.
</p>

<p>
But enough about dnode, let's write a subscription function!
</p>

```
// sub.js

var Hash = require('hashish');
var subs = {};

function publish () {
    var args = arguments;
    Hash(subs).forEach(function (emit) {
        emit.apply(emit, args);
    });
}

setInterval(function () {
    var n = Math.floor(Math.random() * 100);
    publish('data', n);
}, 1000);

var dnode = require('dnode');
dnode(function (client, conn) {
    <b>this.subscribe = function (emit) {
        subs[conn.id] = emit;
        
        conn.on('end', function () {
            delete subs[conn.id];
        });
    };</b>
}).listen(5050);
```

<p>
All the subscription function needs to do is store the emitter function that the
client supplies and then delete the function when the client disconnects.
DNode takes care of the rest!
</p>

<p>
To test this server, let's write a client!
</p>

```
// client.js

var dnode = require('dnode');
var EventEmitter = require('events').EventEmitter;

dnode.connect(5050, function (remote) {
    var em = new EventEmitter;
    em.on('data', function (n) {
        console.log('data: ' + n);
    });
    
    var emit = em.emit.bind(em);
    remote.subscribe(emit);
});
```

<p>
The client creates an event emitter and listens for 'data' events.
Then the event emitter's <span class="code">emit</span> function is passed to
the server so it can publish to it.
The <span class="code">.bind(em)</span> part just makes sure that the
<span class="code">this</span>
supplied to the EventEmitter prototype will be the
<span class="code">em</span> object.
</p>

<p>
Now we can run the server:
</p>

```
$ node sub.js
```

<p>
And in another shell launch the client:
</p>

```
$ node client.js
data: 86
data: 30
data: 12
data: 65

^C
```

<p>
Aww yiss, it works! Launch another client instance in another shell with the
first instance still running and you should get the same data in both clients!
We now have a working PubSub!
</p>

<p>
But wait, there's more!
We can retrofit this example for browser clients with websockets and fallbacks
for browsers that don't have them through dnode's
<a href="http://socket.io/">socket.io</a> support.
</p>

<p>
All we need to do is add some
<a href="http://github.com/senchalabs/Connect/">Connect</a>
code to the top of <span class="code">sub.js</span>
and add another <span class="code">.listen()</span>.
</p>

<p>
We'll use <a href="https://github.com/substack/node-browserify">browserify</a>
to bundle up the dnode source at
<span class="code">/browserify.js</span> so we can
<span class="code">require('dnode')</span> and
<span class="code">require('events').EventEmitters</span>
in the browser.
</p>

```
// web.js

<b>var connect = require('connect');
var webserver = connect.createServer();

webserver.use(connect.static(__dirname));
webserver.use(require('browserify')({ require : 'dnode' }));

webserver.listen(5051);
console.log('http://localhost:5051/');
</b>

var Hash = require('hashish');
var subs = {};

function publish () {
    var args = arguments;
    Hash(subs).forEach(function (emit) {
        emit.apply(emit, args);
    });
}

setInterval(function () {
    var n = Math.floor(Math.random() * 100);
    publish('data', n);
}, 1000);

var dnode = require('dnode');
dnode(function (client, conn) {
    this.subscribe = function (emit) {
        subs[conn.id] = emit;
        
        conn.on('end', function () {
            delete subs[conn.id];
        });
    };
}).listen(5050)<b>.listen(webserver)</b>;
```

<p>
Pow, that was easy! Now just drop an
<span class="code">index.html</span> into the same
directory as <span class="code">web.js</span>:
</p>

```
&lt;!-- index.html --&gt;
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Rolling your own PubSub with DNode!&lt;/title&gt;
&lt;script src="/browserify.js" type="text/javascript"&gt;&lt;/script&gt;
&lt;script type="text/javascript"&gt;
    var dnode = require('dnode');
    var EventEmitter = require('events').EventEmitter;
    
    window.onload = function () {
        dnode.connect(function (remote) {
            var em = new EventEmitter;
            
            em.on('data', function (n) {
                document.getElementById('output').innerHTML += n + ' ';
            });
            
            var emit = em.emit.bind(em);
            remote.subscribe(emit);
        });
    };
&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;

&lt;div id="output"&gt;&lt;/div&gt;

&lt;/body&gt;
&lt;/html&gt;
```

<p>
Now run it by going to <span class="code">http://localhost:5051</span>.
You can even still run <span class="code">node client.js</span>
and you'll get the same data that the browser gets!
</p>

<p>
Now we've rolled our own PubSub with dnode and it was super easy!
It even works on the web browser with shiny websockets!
</p>

<p>
You can check out the code from this article at
<a href="https://github.com/substack/dnode-pubsub">substack/dnode-pubsub on github</a>.
</p>

<p>
You should also <a href="http://github.com/substack">follow me on github</a>!
</p>
