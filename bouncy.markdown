Introducing <a href="https://github.com/substack/bouncy">bouncy</a>, a
websocket and https-capable http router proxy / load balancer in node.js!
</p>

<p>
It's super simple to use, just call <span class="code">bounce()</span>!
</p>

```
var bouncy = require('bouncy');

bouncy(function (req, bounce) {
    if (req.headers.host === 'bouncy.example.com') {
        bounce(8000);
    }
    else if (req.headers.host === 'trampoline.example.com') {
        bounce(8001)
    }
}).listen(80);
```

<p>
You can <span class="code">bounce()</span> to a port, a
<span class="code">host, port</span>, or a stream you open yourself.
</p>

<p>
Since bouncy is just parsing the http headers and sending along the raw tcp
stream, you can use websockets on the place you
<span class="code">bounce()</span> to without writing any special code!
</p>

<p>
Bouncy uses node's delicious http parsing innards, so the
<span class="code">req</span>
object you get is a bona-fide
<a href="http://nodejs.org/docs/v0.4.10/api/all.html#http.ServerRequest"
>http.ServerRequest</a> with all the fixins.
</p>

<p>
Plus, bouncy comes with a simple command-line tool if you have a static routing
table kicking around in a json file. Just throw a
<span class="code">routes.json</span> like this one:
</p>

```
{
    "beep.example.com" : 8000,
    "boop.example.com" : 8001
}
```

<p>
at the <span class="code">bouncy</span> command and give it a port to listen on:
</p>

```
bouncy routes.json 80
```

<p>
Super easy!
<a href="https://github.com/substack/bouncy">Check out the code on github</a>
or with <a href="http://npmjs.org/">npm</a> do:
</p>

```
npm install bouncy
```

<p>
to install the library or
</p>

```
npm install -g bouncy
```

<p>
to install the command-line tool.
</p>

<img src="/images/bouncy.png" width="468" height="586">
