<a href="https://github.com/substack/node-seq">Seq</a> is a node.js library for
chainable, asynchronous flow control. With Seq, you can turn complicated nested
callback logic into a cleaner and more straightforward pipeline-style. Even the
error handling is chainable! Plus, you can do fancy things like set limits on
the number of parallel tasks executing at once.
</p>

<p>
Here's a simple example of Seq that executes some shell commands and reads
files.
</p>

<code>
// parseq.js
var fs = require('fs');
var exec = require('child_process').exec;

var Seq = require('seq');
Seq()
    .seq(function () {
        exec('whoami', this)
    })
    .par(function (who) {
        exec('groups ' + who, this);
    })
    .par(function (who) {
        fs.readFile(__filename, 'ascii', this);
    })
    .seq(function (groups, src) {
        console.log('Groups: ' + groups.trim());
        console.log('This file has ' + src.length + ' bytes');
    })
;
</code>

<p>
In each <span class="code">.seq()</span> and <span class="code">.par()</span>,
use <span class="code">this</span> in place of a callback and the next action
down the chain waiting on the result will fire when the result becomes available.
</p>

<p>
<span class="code">.seq()</span> waits for all running actions to stop before
itself firing and the next action in the chain doesn't fire until 
<span class="code">.seq()</span> has yielded its result.
</p>

<p>
<span class="code">.par()</span> executes immediately and moves along to the
next action in the chain. <span class="code">.par()</span>'s result is pushed
onto the argument stack in the order it appeared in the chain, so you can 
chain together parallel actions and <span class="code">.seq()</span> to join the
results:
</p>

<code>
var Seq = require('seq');
Seq()
    .par(function () {
        var that = this;
        setTimeout(function () { that(null, 'a') }, 300);
    })
    .par(function () {
        var that = this;
        setTimeout(function () { that(null, 'b') }, 200);
    })
    .par(function () {
        var that = this;
        setTimeout(function () { that(null, 'c') }, 100);
    })
    .seq(function (a, b, c) {
        console.dir([ a, b, c ])
    })
;
</code>

<p></p>

<code>
$ node join.js 
[ 'a', 'b', 'c' ]
</code>

<p>
Here, despite 'c' finishing first, its result shows up last in the final call to
<span class="code">.seq()</span> because it was last in the chain of
<span class="code">.par()</span>s.
</p>

<p>
Errors are propagated through the first argument to
<span class="code">this()</span>.
When an error occurs (a non-falsy value is given),
it travels down the chain to the first
<span class="code">.catch()</span> it sees.
If there hasn't been an error, <span class="code">.catch()</span>
just gets skipped. By default there is a
<span class="code">.catch()</span>
at the end of all chains for uncaught errors that looks like:
</p>

<code>
.catch(function (err) {
    console.error(err.stack ? err.stack : err)
})
</code>

<p>
This approach gets rid of a lot of the duplication and hassle of error handling
since most of the time when an error occurs you just want to print it and abort
the transaction. You can throw in custom logic if you need it at the same time,
so I figure this is a pretty reasonable default.
</p>

<p>
Seq borrows heavily from other flow control libraries like
<a href="https://github.com/creationix/step">Step</a> and
<a href="https://github.com/caolan/async">Async.js</a>
but provides a chainable interface instead of array-based actions.
This chainable approach means you don't need to chain multiple functions
together in the same call, but more importantly gives fancier parallel flow
control a more consistent interface. For instance:
</p>

<code>
// stat_all.js

var fs = require('fs');
var Hash = require('traverse/hash');
var Seq = require('seq');

Seq()
    .seq(function () {
        fs.readdir(__dirname, this);
    })
    .flatten()
    .parEach(function (file) {
        fs.stat(__dirname + '/' + file, this.into(file));
    })
    .seq(function () {
        var sizes = Hash.map(this.vars, function (s) { return s.size })
        console.dir(sizes);
    })
;
</code>

<p></p>

<code>
$ node stat_all.js 
{ 'join.js': 443
, 'parseq.js': 464
, 'stat_all.js': 404
}
</code>

<p>
In the above example, the first <span class="code">.seq()</span> pulls down the
directory list asynchronously from <span class="code">__dirname</span>, flattens
the results so they fill out the argument stack instead of the first argument,
and computes the size of each of the files in parallel with
<span class="code">.parEach()</span>.
</p>

<p>
<span class="code">.parEach()</span> takes an optional concurrency limit as its
first argument, so to make that previous example only have at most 2
asynchronous <span class="code">fs.stat()</span> operations waiting at a time,
just change:
</p>

<code>
.parEach(function (file) {
    fs.stat(__dirname + '/' + file, this.into(file));
})
</code>

<p>
to read:
</p>

<code>
.parEach(2, function (file) {
    fs.stat(__dirname + '/' + file, this.into(file));
})
</code>

<p>
Super easy!
To read more about Seq and to check out the code, go to
<a href="https://github.com/substack/node-seq"
>http://github.com/substack/node-seq</a>.

For a more complicated example of Seq in action, check out
<a href="https://github.com/substack/rowbit/blob/master/bin/server.js">this DNode-based IRC bot</a>
I've been hacking on.
</p>

<p>
To install with <a href="http://github.com/isaacs/npm">npm</a>, just do:
</p>

<code>
npm install seq
</code>

<img src="/images/marley.png" width="279" height="593">
