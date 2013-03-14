<a href="https://github.com/mishoo/UglifyJS">Uglify</a>
is a pretty nifty javascript minifier but it also has the lesser-known ability
to generate and walk the AST.
</p>

<p>
Unfortunately, uglify's traverser is
<a href="https://github.com/mishoo/UglifyJS/blob/6a873a7757895fdc06501854e4ba84e15c12a74a/tmp/instrument.js">a bit cumbersome to use</a>
so I wrote a library using
<a href="https://github.com/substack/js-traverse">my own traverser</a>
to make it easy!
</p>

<div>
<img src="/images/burrito.png" width="834" height="205">
</div>

<p>
Introducing
<a href="https://github.com/substack/node-burrito">burrito</a>,
a crazy new library to make AST traversals and manipulations crazy easy.
</p>

<p>
Given this snippet of code:
</p>

<code>
f() &amp;&amp; g(h());

foo();
</code>

<p>
It's super easy to wrap all the function calls in another function call,
<span class="code">qqq()</span>:
</p>

<code>
var burrito = require('burrito');

var src = burrito('f() &amp;&amp; g(h())\nfoo()', function (node) {
    if (node.name === 'call') node.wrap('qqq(%s)');
});

console.log(src);
</code>

<p></p>

<code>
$ node wrap.js 
qqq(f()) &amp;&amp; qqq(g(qqq(h())));

qqq(foo());
</code>

<p>
Or maybe dive in and surgically replace that
<span class="code">&amp;&amp;</span>
with a
<span class="code">||</span>, how about!
</p>

<code>
var burrito = require('burrito');

var src = burrito('f() &amp;&amp; g(h())\nfoo()', function (node) {
    if (node.name === 'binary') node.wrap('%a || %b');
});

console.log(src);
</code>

<p></p>

<code>
$ node wrap.js 
f() || g(h());

foo();
</code>

<p>
Plus you can <span class="code">.microwave()</span> your burrito too, which
evals the code using node's <span class="code">vm.runInNewContext()</span>
(which even runs in the browser thanks to
<a href="http://github.com/substack/node-browserify">browserify</a>).
</p>

<code>
var burrito = require('burrito');

var res = burrito.microwave('Math.sin(2)', function (node) {
    if (node.name === 'num') node.wrap('Math.PI / %s');
});

console.log(res);
</code>

<p>
This snippet takes the expression <span class="code">Math.sin(2)</span> and
replaces the <span class="code">2</span> with <span class="code">Math.PI /
2</span> thanks to some <span class="code">%s</span> interpolation.
And since <span class="code">Math.sin(Math.PI / 2)</span> evaluates at <span
class="code">1</span>, the code should print as much...
</p>

<code>
$ node microwave.js
1
</code>

<p>
And it does! Hooray!
</p>

<div>
<img src="/images/microwave.png" width="555" height="280">
</div>

<p>
This craziness makes an amazing number of things possible, like
<a href="https://github.com/substack/node-stackedy">stack traces</a>
that work in all the browsers without relying on a stack trace API or code
coverage tools without any binary extensions!
</p>

<p>Give burrito a spin!</p>

<code>
<a href="http://npmjs.org">npm</a> install burrito
</code>

<p>
or <a href="https://github.com/substack/node-burrito">get the code on github!</a>
</p>
