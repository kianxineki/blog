I thought http sessions in node.js were a bit cumbersome so I hacked up
<a href="http://github.com/substack/node-sesame">sesame</a> to make it crazy
simple: just update <span class="code">req.session</span>!

Using harmony proxies, this <span class="code">req.session</span> interface even
works with a database store so your sessions persist across restarts and crashes!
</p>

<p>
Here's a simple example that increments a counter,
<span class="code">times</span>.
</p>

<code>
var connect = require('connect');
var webserver = connect.createServer();
webserver.use(require('sesame')());

webserver.use(connect.router(function (app) {
    app.get('/', function (req, res) {
        req.session.times = (req.session.times || 0) + 1;
        
        res.writeHead(200, { 'Content-Type' : 'text/plain' });
        res.end(req.session.times + ' times!');
    });
}));

console.log('Listening on 9090');
webserver.listen(9090);
</code>

<p>
With no arguments, <span class="code">require('sesame')()</span> just loads up
sessions in-memory with no disk backup. You can pass in a storage engine like
nStore and supermarket to save the sessions to disk:
</p>

<code>
var connect = require('connect');
var webserver = connect.createServer();
webserver.use(require('sesame')(<b>{
    store : new(require('supermarket'))({
        filename : __dirname + '/supermarket.db', json : true,
    })
}</b>));

webserver.use(connect.router(function (app) {
    app.get('/', function (req, res) {
        req.session.times = (req.session.times || 0) + 1;
        
        res.writeHead(200, { 'Content-Type' : 'text/plain' });
        res.end(req.session.times + ' times!');
    });
}));

console.log('Listening on 9090');
webserver.listen(9090);
</code>

<p>
And now the <span class="code">req.session.times</span> state persists across
server restarts! Thanks to some harmony proxy hacks on the backend, the
interface doesn't change at all!

You can update deeply nested values too:
</p>

<code>
var connect = require('connect');
var webserver = connect.createServer();
webserver.use(require('sesame')({
    store : new(require('supermarket'))({
        filename : __dirname + '/supermarket.db', json : true,
    })
}));

webserver.use(connect.router(function (app) {
    app.get('/', function (req, res) {
        req.session.times = (req.session.times || 0) + 1;
        <b>if (!req.session.foo) {
            req.session.foo = { bar : { baz : 1 } };
        }
        req.session.foo.bar.baz *= 2;</b>
        
        res.writeHead(200, { 'Content-Type' : 'text/plain' });
        res.end(
            req.session.times + ' times!'
            <b>+ ' baz = ' + req.session.foo.bar.baz</b>
        );
    });
}));

console.log('Listening on 9090');
webserver.listen(9090);
</code>

<p>
And it all just works&trade;.

Plus, the writes are buffered with a
<span class="code">process.nextTick()</span>
so if you update a bunch of elements in <span class="code">req.session</span>
one after the other sesame will only do one write for all of them.
</p>

<p>
<a href="https://github.com/substack/node-sesame">sesame on github</a>
</p>

<p>
Open sesame!
</p>

<p>
<img src="/images/sesame.png" width="537" height="380">
</p>
