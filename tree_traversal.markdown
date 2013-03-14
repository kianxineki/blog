Lately I've been hacking together a crazy remote method invocation system on top
of node.js. Expectedly,
JSON.stringify doesn't do functions. In arrays functions get
turned into null and objects with keys that point at functions are ignored.
</p>

<code>
&gt; JSON.stringify([ 7, 8, function () {}, 9, { b : 4, c : function () {} } ])
'[7,8,null,9,{"b":4}]'
</code>

<img src="/images/tree-traversal.png" class="right-art"
    width="333" height="250"
/>

<p>
I needed a way to pull the functions out of arbitrarily complicated
objects so that I can send data about them in a seperate JSON field
from the primary data structure.
</p>

<p>
I couldn't find any modules on the web that traverse and transform arbitrarily
nested objects in javascript, possibly owing to all the noise around DOM tree
traversal, so tonight I wrote a small library:
<a href="http://github.com/substack/js-traverse ">js-traverse</a>.
</p>

<p>
Here's a small example that uses Traverse to collect all the leaf nodes from a
complicated data structure.
</p>

<code>
#!/usr/bin/env node
var sys = require('sys');
var Traverse = require('traverse').Traverse;

var acc = [];
Traverse({
    a : [1,2,3],
    b : 4,
    c : [5,6],
    d : { e : [7,8], f : 9 }
}).forEach(function (x) {
    if (this.isLeaf) acc.push(x);
});
sys.puts(acc.join(' '));

/* Output:
    1 2 3 4 5 6 7 8 9
*/
</code>

<p>
    Here's another simple example that modifies a tree, adding 128 to all negative numbers.
</p>

<code>
#!/usr/bin/env node
var sys = require('sys');
var Traverse = require('traverse').Traverse;

var fixed = Traverse([
    5, 6, -3, [ 7, 8, -2, 1 ], { f : 10, g : -13 }
]).modify(function (x) {
    if (x < 0) this.update(x + 128);
}).get()
sys.puts(sys.inspect(fixed));

/* Output:
    [ 5, 6, 125, [ 7, 8, 126, 1 ], { f: 10, g: 115 } ]
*/
</code>

<p>
And finally, this code solves the problem from the introduction.
It pulls out the functions and puts them into a secondary data structure.
This example also replaces the functions with the string "[Function]" so I won't
forget that I'm supposed to replace those on the remote end with actual
functions.
</p>

<code>
#!/usr/bin/env node
var sys = require('sys');
var Traverse = require('traverse').Traverse;

var id = 54;
var callbacks = {};
var obj = { moo : function () {}, foo : [2,3,4, function () {}] };

var scrubbed = Traverse(obj).modify(function (x) {
    if (x instanceof Function) {
        callbacks[id] = { id : id, f : x, path : this.path };
        this.update('[Function]');
        id++;
    }
}).get();

sys.puts(JSON.stringify(scrubbed));
sys.puts(sys.inspect(callbacks));

/* Output:
    {"moo":"[Function]","foo":[2,3,4,"[Function]"]}
    { '54': { id: 54, f: [Function], path: [ 'moo' ] }
    , '55': { id: 55, f: [Function], path: [ 'foo', '3' ] }
    }
*/
</code>

<p>
The traversal library is
<a href="http://github.com/substack/js-traverse"
>available on github</a>.
You can also install this library with <a href="http://npmjs.org/">npm</a>, a
nifty package manager for node.js.
</p>

<code>
    npm install traverse
</code>

<p>Happy hacking!</p>
