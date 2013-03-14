Pushing out incremental changes to a service-oriented cluster can be tricky,
especially when changes span multiple services.

Introducing <a href="http://github.com/substack/seaport">seaport</a>, a service
registry written in node.js based on <a href="http://semver.org/">semvers</a>.
</p>

<p>
With <a href="http://github.com/substack/seaport">seaport</a>, services are
brought up with a <span class="code">name@version</span> string and other
processes can connect to services that match a
<span class="code">name@semver</span> pattern.
</p>

<p>
First spin up a new seaport server on a port:
</p>

```
$ seaport 9090 --secret='beep boop'
```

<p>
Here's an example seaport service that registers a new service web@1.2.3:
</p>

```
var seaport = require('seaport');
var ports = seaport.connect('localhost', 9090, { secret : 'beep boop' });
var http = require('http');

var server = http.createServer(function (req, res) {
    res.end('beep boop\r\n');
});

ports.service('web@1.2.3', function (port, ready) {
    server.listen(port, ready);
});
```

<p>
and here's some code that connects to the web@1.2.3 service:
</p>

```
var seaport = require('seaport');
var ports = seaport.connect('localhost', 9090, { secret : 'beep boop' });
var request = require('request');

ports.get('web@1.2.x', function (ps) {
    var u = 'http://' + ps[0].host + ':' + ps[0].port;
    request(u).pipe(process.stdout);
});
```

<p>
Running the last script prints out <span class="code">beep boop</span> from the
web@1.2.3 http server.
</p>

<p>
You can create seaport servers programatically. This is useful for integrating
seaport with an http host router.
Here's an example using <a href="http://github.com/substack/bouncy">bouncy</a>:
</p>

```
var seaport = require('seaport');
var ports = seaport.createServer().listen(5001);

var bouncy = require('bouncy');
bouncy(function (req, bounce) {
    var domains = (req.headers.host || '').split('.');
    var service = 'http@' + ({
        unstable : '0.1.x',
        stable : '0.0.x',
    }[domains[0]] || '0.0.x');
    
    var ps = ports.query(service);
    
    if (ps.length === 0) {
        var res = bounce.respond();
        res.end('service not available\n');
    }
    else {
        bounce(ps[0].host, ps[0].port);
    }
}).listen(5000);
```

<p>
We can then spin up http services for the semvers 0.1.x and 0.0.x:
</p>

```
var seaport = require('seaport');
var ports = seaport.connect('localhost', 5001);
var http = require('http');

var server = http.createServer(function (req, res) {
    res.end('version 0.0.0\r\n');
});

ports.service('http@0.0.0', function (port, ready) {
    server.listen(port, ready);
});
```

<p>
and the other server...
</p>

```
var seaport = require('seaport');
var ports = seaport.connect('localhost', 5001);
var http = require('http');

var server = http.createServer(function (req, res) {
    res.end('version 0.1.0\r\n');
});

ports.service('http@0.1.0', function (port, ready) {
    server.listen(port, ready);
});
```

<p>
Now we can dynamically route to semvers based on the http host header!
</p>

```
$ curl -H host:stable.localhost localhost:5000
version 0.0.0
$ curl -H host:unstable.localhost localhost:5000
version 0.1.0
```

<p>
These http servers could themselves have dependencies on other services in the
cluster. By using seaport, new code can be pushed out that spans multiple
servers with very explicit backwards compatability that tolerates and encourages
multiple versions of services to satisfy dependencies. The easier it is to push
out crazy new experiments, the more often it will happen!
</p>

<p>
Seaport is a component for
<a href="https://gist.github.com/1734155">these hypothetical cluster commands</a>
that would make continuous deployment for clusters super simple.
</p>

<p>
Check out the
<a href="http://github.com/substack/seaport">seaport source on github</a>
or <span class="code">npm install seaport</span>!
</p>

<img src="/images/crane.png">
