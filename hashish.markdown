<p>
I've been writing a lot of node.js lately and I often need to perform
complicated operations on hashes (the {} kind), so I snuck a nifty hash library
into <a href="http://github.com/substack/js-traverse">js-traverse</a>.
With this library, you can chain together hash operations like this:
</p>

```
var Hash = require('traverse/hash');
Hash({ a : 1, b : 2 })
    .map(function (v) { return v + 1 })
    .update({ b : 30, c : 42 })
    .filter(function (v) { return v % 2 == 0 })
    .tap(function () {
        var anyC = this.some(function (value, key) { return key == 'c' });
        // or just this.keys.some, but anyways
        console.log(anyC
            ? "There's a C key this far."
            : "There's no C key this far."
        );
    })
    .items
;
```

<p>
which will print "There's a C key this far." and return the hash
</p>

```
{ a : 2, b : 30, c : 42 }
```

<p>
The library borrows a lot from ruby hashes and javascript arrays. It's good for
chaining together lots of operations but also good for simple stuff like
Hash(obj).values since there is no Object.values() (yet).
</p>

<p>
    Create a hash instance with Hash(obj). Hash instances provide:
    <ul>
        <li>map</li>
        <li>forEach</li>
        <li>filter</li>
        <li>reduce</li>
        <li>some</li>
        <li>update</li>
        <li>merge</li>
        <li>valuesAt</li>
        <li>tap</li>
        <li>extract</li>
        <li>items</li>
        <li>keys</li>
        <li>values</li>
        <li>clone</li>
        <li>copy</li>
    </ul>
</p>

<p>
Where callbacks are provided, they usually get called with
f(value, key) with the Hash instance as 'this'.
</p>

<p>
The Hash function itself also has some functions tacked on so you can do:
</p>
```
Hash.map({ a : 1, b : 2 }, function (x) { return x + 1 })
```

and get back an object without having to call .items on it:
```
{ a: 2, b: 3 }
```

<p>
Other libraries like
<a href="http://www.prototypejs.org/api/hash">prototype's Hash</a>
don't make so much of a fuss over chaining. Anyways, mine is better.
</p>

<p>
To install, just:
</p>

```
npm install traverse
```

or else:
```
git clone <a href="http://github.com/substack/js-traverse">http://github.com/substack/js-traverse.git</a>
```

<p>
then
</p>

```
var Hash = require('traverse/hash');
```

