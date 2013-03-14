    src="/images/binary-stream.png" alt="binary stream"
    class="right-art"
    width="248" height="361"
/>

<p>
Recently, I've been collaborating on a web-based 
<a href="http://github.com/substack/stackvm">virtual machine interface</a> to
<a href="http://qemu.org/">qemu</a> over its VNC interface.
The latest prototype uses <a href="http://nodejs.org/ ">node.js</a> for the
<a href="http://socket.io/">socket.io</a> library, which provides an
abstract, portable, and efficient interface for passing messages between a
browser and web server.
</p>

<p>
In order to bring down latency,
I needed a way to decode the RFB protocol, but node.js is asynchronous,
so I couldn't just do:
</p>
<code>
var width = sock.read(2);
var height = sock.read(2);
</code>
<p>
I had to collect up lots of tiny
<a href="http://nodejs.org/api.html#buffers-3">buffers</a>
emitted asyncronously and treat them
all as one for the purposes of parsing, which went something like this:
</p>
<code>
var buffers = [];
var bytes = 0, offset = 0;
sock.addListener('data', function (buf) {
    buffers.push(buf);
    bytes += buf.length;
    
    if (offset == 0 &amp;&amp; bytes &gt;= 2) {
        var width = read(2);
    }
    else if (offset == 2 &amp;&amp; bytes &gt;= 4) {
        var height = read(2);
    }
    
    function read (n) {
        var buffer = new Buffer(n);
        var current = buffers[offset];
        var current_i = 0;
        for (var i = 0; i &lt; n; i++) {
            if (current_i &gt;= current.length) {
                buffers.shift();
                current = buffers[0];
                current_i = 0;
            }
            buffer[i] = current[current_i++];
        }
        offset += n;
        return buffer;
    }
});
</code>
<p>Yikes, what a mess!</p>

<p>
To preventatively cull down the impending complexity,
I hacked together
<a href="http://github.com/substack/node-bufferlist ">node-bufferlist</a>
to build linked lists of buffers
from the network stream, which made the code simpler:
</p>

<code>
var BufferList = require('bufferlist').BufferList;
var bufferList = new BufferList;
var state = 0;

sock.addListener('data', function (buf) {
    bufferList.push(buf);
    
    if (state == 0 &amp;&amp; bufferList.length &gt;= 2) {
        var width = read(2);
        state ++;
    }
    else if (state == 1 &amp;&amp; bufferList.length &gt;= 2) {
        var height = read(2);
        state ++;
    }
    
    function read (n) {
        var s = bufferList.take(n);
        bufferList.advance(2);
        return s;
    }
});
</code>

<p>
But even with this tool, keeping track of the parser state by hand was very
error-prone and ugly.
Plus, the real RFB parser would need branches, loops, and nested parsing.
</p>

<img
    src="/images/method-chain.png" alt="method chain"
    width="437" height="219"
/>

<p>
Borrowing some ideas from haskell's 
<a href="http://hackage.haskell.org/package/binary-0.5"
>binary Get monad</a> and ruby's
<a href="http://projects.gregweber.info/methodchain">methodchain gem</a>,
I built a nifty 
<a href="http://www.martinfowler.com/bliki/FluentInterface.html"
>fluent interface</a>
for monadic, asynchronous binary bufferlist parsing.
Said more simply in code:
</p>

<code>
var BufferList = require('bufferlist').BufferList;
var Binary = require('bufferlist/binary').Binary;

var bufferList = new BufferList;
Binary(bufferlist)
    .getWord16be('width')
    .getWord16be('height')
    .tap(function (vars) {
        var width = vars.width;
        var height = vars.height;
        // ...
        // You can even start a new chain inside this block!
    })
    .end()
;
sock.addListener('data', function (buf) {
    bufferList.push(buf);
});

</code>

<p>
Sooooooo much better.
Nicer still, with this approach I was able to add goodies like when, unless,
repeat, forever, and into, along with some nifty
<a href="http://nodejs.org/api.html#eventemitter-14 ">EventEmitter</a>
hooks.
These abstractions made the resulting
<a href="http://github.com/substack/node-rfb">node-rfb</a>
far prettier and maintainable than it would have been otherwise.
</p>

<p>
For the code and more examples, you can
<a href="http://github.com/substack/node-bufferlist"
>checkout node-bufferlist on github</a>.
</p>
