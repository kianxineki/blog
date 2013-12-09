In node I use simple test libraries like [tap](https://npmjs.org/package/tap) or
[tape](https://npmjs.org/package/tape) that let you run the test files directly.
For code that needs to run in both the browser and node I use
[tape](https://npmjs.org/package/tape) because tap doesn't run in the browser
very well and the APIs are mostly interchangeable.

The simplest kind of test I might write in test/ looks like:

``` js
var test = require('tape');
var someModule = require('../');

test('fibwibblers and xyrscawlers', function (t) {
    t.plan(2);
    
    var x = someModule();
    t.equal(x.foo(2), 22);
    
    x.beep(function (err, res) {
        t.equal(res, 'boop');
    });
})
```

To run a single test file in node I just do:

```
node test/fibwibbler.js
```

And if I have multiple tests I want to run I do:

```
tape test/*.js
```

or I can just use the tap command even if I'm just using tape because `tap` only
looks at stdout for tap output:

```
tap test/*.js
```

The best part is that since [tape](https://npmjs.org/package/tape) just uses
`console.log()` to print its tap-formatted assertions, all I need to do is
browserify my test files.

To compile a single test in the browser I can just do:

```
browserify test/fibwibbler.js > bundle.js
```

or to compile a directory full of tests I just do:

```
browserify test/*.js > bundle.js
```

Now to run the tests in a browser I can just write an index.html:

``` html
<script src="bundle.js"></script>
```

and `xdg-open` that index.html in a local browser. To shortcut that process, I
can use the `testling` command (`npm install -g testling`):

```
browserify test/*.js | testling
```

which launches a browser locally and prints the `console.log()` statements that
executed browser-side to my terminal directly. It even sets the process exit
code based on whether the TAP output had any errors:

```
substack : defined $ browserify test/*.js | testling

TAP version 13
# defined-or
ok 1 empty arguments
ok 2 1 undefined
ok 3 2 undefined
ok 4 4 undefineds
ok 5 false[0]
ok 6 false[1]
ok 7 zero[0]
ok 8 zero[1]
ok 9 first arg
ok 10 second arg
ok 11 third arg
not ok 12 (unnamed assert)
  ---
    operator: ok
    expected: true
    actual:   false
    at: Test.ok.Test.true.Test.assert (http://localhost:47079/__testling?show=true:7772:10)
  ...
# (anonymous)
ok 13 should be equal

1..13
# tests 13
# pass  12
# fail  1
substack : defined $ echo $?
1
substack : defined $ 
```

# coverage

bonus content: if I want code coverage, I can just sneak that into the pipeline
using [coverify](https://npmjs.org/package/coverify). This is still experimental
but here's how it looks:

```
$ browserify -t coverify test.js | testling | coverify

TAP version 13
# beep boop
ok 1 should be equal

1..1
# tests 1
# pass  1

# ok

# /tmp/example/test.js: line 7, column 16-28

  if (err) deadCode();
           ^^^^^^^^^^^

# /tmp/example/foo.js: line 3, column 35-48

  if (i++ === 10 || (false && neverFires())) {
                              ^^^^^^^^^^^^

```

or to run the tests in node, just swap `testling` for `node`:

```
$ browserify -t coverify test.js | node | coverify
TAP version 13
# beep boop
ok 1 should be equal

1..1
# tests 1
# pass  1

# ok

# /tmp/example/test.js: line 7, column 16-28

  if (err) deadCode();
           ^^^^^^^^^^^

# /tmp/example/foo.js: line 3, column 35-48

  if (i++ === 10 || (false && neverFires())) {
                              ^^^^^^^^^^^^

```

# why write tests this way?

The [node-tap](https://npmjs.org/package/tap) API is pretty great because it
feels asynchronous by default. Since you plan out the number of assertions ahead
of time, it's much easier to catch false positives where asynchronous handlers
with assertions inside didn't fire at all.

By using simple text-based interfaces like stdout and `console.log()` it's easy
to get tests to run in node and the browser and you can just pipe the output
around to simple command-line tools. If you stick to tools that just do one
thing but expose their functionality in a hackable way, it's easy to recombine
the pieces however you want and swap out components to better suit your specific
needs.
