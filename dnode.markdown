
<p>
Introducing
<a href="http://github.com/substack/dnode">DNode</a>,
a node.js library for asynchronous bidirectional remote method invocation.
Network socket and websocket-style
<a href="http://www.rosepad.com/socket-io-sockets-for-the-rest-of-us/">
socket.io</a>
transports are presently available so system processes can communicate with each
other and with processes running in the browser using the same interface.
</p>

<p>
Remote method invocation (RMI) is the object-oriented cousin of remote procedure
calls. In RMI, each side of the connection hosts an object that the other side
can call the methods of. A popular example of RMI worth checking out is
<a href="http://ruby.about.com/od/advancedruby/a/drb.htm">Ruby's DRb</a>.
</p>

<p>
DNode is different from DRb in that all remote method calls are asynchronous.
Instead of returning explicitly, hosted methods pass return values to the other
side of the connection by invoking callbacks that were passed in as arguments.
These callbacks execute on the side of the connection where they were defined,
and a representation of them gets passed to the remote, so there's no eval().
</p>

<p>
Here's a simple example.
</p>

<code>
// server:
var DNode = require('dnode');

var server = DNode({
    timesTen : function (n,f) { f(n * 10) },
}).listen(6060);
</code>
<br>
<code>
// client:
var DNode = require('dnode');
var sys = require('sys');

DNode.connect(6060, function (remote) {
    remote.timesTen(5, function (result) {
        sys.puts(result); // 5 * 10 == 50
    });
});
</code>

<p>
Unlike many asynchronous RPC systems, DNode allows the programmer to pass
functions as arguments to remote methods, which when called on the remote end
signal the near side to execute the function with the arguments that the remote
provides.
Callbacks are automatically scrubbed and collected from a recursive walk of the
arguments list, so they can be arbitrarily nested.
</p>

<p>
Additionally, each side of the connection can call the other side's methods or
the callbacks the other side provides with callbacks of its own or any arguments
that can be serialized to JSON.
</p>

<h2>Bidirectional Example</h2>

<p>
Here's an example where the client calls a method on the server that calls a
method on the client.
</p>

<code>
// server:
var DNode = require('dnode');

DNode(function (client) {
    this.timesX = function (n,f) {
        client.x(function (x) {
            f(n * x);
        });
    }; 
}).listen(6060);
</code>
<br>
<code>
// client:
var DNode = require('dnode');

DNode({
    x : function (f) { f(20) }
}).connect(6060, function (remote) {
    remote.timesX(3, function (res) {
        sys.puts(res); // 20 * 3 == 60
    });
});
</code>

<h2>DNode Browser Example</h2>

<p>
As I mentioned in the first paragraph, websocket-style connections between the
browser and node.js DNode servers are available thanks to
<a href="http://www.rosepad.com/socket-io-sockets-for-the-rest-of-us/">
socket.io</a>.
The following example should work in Chrome, Firefox, Opera, and IE 5.5+.
See the <a href="http://wiki.github.com/substack/dnode/browser-compatibility">
browser compatibility page</a> for more info.
</p>

<p>
In this example, the client code runs on the browser and requests the server's
timesTen method. The result of the timesTen call is put into the first span
element. The client also calls the server's whoAmI method. The server calls the
client's name method and performs some regular expression substitions on the
result it gets before sending the result back to the client through a callback.
The client's name with the substitions applied shows up in the second span
element on the page.
</p>

<p>
Here's an example of what the browser-side code looks like:
</p>

<code>
&lt;script type="text/javascript" src="/dnode.js"&gt;&lt;/script&gt;
&lt;script type="text/javascript"&gt;
    DNode({
        name : function (f) { f('Mr. Spock') }
    }).connect(function (remote) {
        remote.timesTen(10, function (n) {
            document.getElementById("result").innerHTML = String(n);
        });
        remote.whoAmI(function (name) {
            document.getElementById("name").innerHTML = name;
        });
    });
&lt;/script&gt;

&lt;p&gt;timesTen(10) == &lt;span id="result"&gt;?&lt;/span&gt;&lt;/p&gt;
&lt;p&gt;My name is &lt;span id="name"&gt;?&lt;/span&gt;.&lt;/p&gt;
</code>

<p>
Here's the server-side compliment to the browser-side code above:
</p>

<code>
#!/usr/bin/env node

var DNode = require('dnode');
var sys = require('sys');
var fs = require('fs');
var http = require('http');

var html = fs.readFileSync(__dirname + '/web.html');
var js = require('dnode/web').source();

var httpServer = http.createServer(function (req,res) {
    if (req.url == '/dnode.js') {
        res.writeHead(200, { 'Content-Type' : 'text/javascript' });
        res.end(js);
    }
    else {
        res.writeHead(200, { 'Content-Type' : 'text/html' });
        res.end(html);
    }
});
httpServer.listen(6061);

DNode(function (client) {
    this.timesTen = function (n,f) { f(n * 10) };
    this.whoAmI = function (reply) {
        client.name(function (name) {
            reply(name
                .replace(/Mr\.?/,'Mister')
                .replace(/Ms\.?/,'Miss')
                .replace(/Mrs\.?/,'Misses')
            );
        })
    };
}).listen({
    protocol : 'socket.io',
    server : httpServer,
    transports : 'websocket xhr-multipart xhr-polling htmlfile'.split(/\s+/),
}).listen(6060);
</code>

<h2>Installation</h2>

<p>
DNode is available on <a href="http://npmjs.org/">npm</a>, a package manager for
node.js libraries. You can install dnode by typing:
</p>
<code>
npm install dnode
</code>

<p>
Or you can check out the repository and link your development copy with npm:
</p>
<code>
git clone http://github.com/substack/dnode.git
cd dnode
npm link .
</code>

<p>
DNode depends on
<a href="http://github.com/LearnBoost/Socket.IO-node">Socket.IO-node</a>,
<a href="http://github.com/substack/node-bufferlist">bufferlist</a>
and <a href="http://github.com/substack/js-traverse">traverse</a>,
all of which are available on npm and are automatically installed when you
type `npm install dnode`.
</p>

<p>
You can fork and follow
<a href="http://github.com/substack/dnode">DNode on github</a>.
</p>

<h2>And another thing</h2>

<p>
I've documented the DNode protocol
<a href="http://github.com/substack/dnode/blob/master/README.markdown">
on the README</a> to make it easier to implement the DNode protocol in other
languages. Stay tuned.
</p>

<p>
Special thanks goes out to <a href="http://catonmat.net/">pkrumins</a> for 
getting the DNode browser code to work in Internet Explorer.
</p>

<p>
Thanks for reading. Enjoy this complimentary robot free of charge:
</p>

<img src="/images/message-passing.png" width="394" height="424" alt="message-passing robot">
