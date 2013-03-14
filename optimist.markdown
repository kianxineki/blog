<a href="https://github.com/substack/node-optimist">Optimist</a>
is an option parser I wrote for
<a href="http://nodejs.org">node.js</a>
that generates a hash from process.argv and gets out of your way.
You don't need to define any optstrings or option data structures, just do:
</p>

```
#!/usr/bin/env node
var argv = require('optimist').argv;
console.dir(argv);
```

<p></p>

```
$ ./opts.js --foo bar -xzf moo.tar somedir/
{ _: [ 'somedir/' ]
, '$0': 'node ./opts.js'
, foo: 'bar'
, x: true
, z: true
, f: 'moo.tar'
}
```

<img src="/images/optimistic.png" width="376" height="399">

<p>
So easy!
You can do long options, short options, grouped options,
<span class="code">--key value</span> or <span class="code">--key=value</span>,
and even non-hyphenated options, which end up in
<span class="code">argv._</span>.
It all ends up in <span class="code">argv</span>.
</p>

<p>
But that's not all!
You can tack on extra goodies pipeline-style between the
<span class="code">require('optimist')</span> and the <span
class="code">.argv</span> like this:
</p>

```
var argv = require('optimist')
    .usage('Usage: $0 -x [num] -y [num]')
    .demand(['x','y'])
    .argv
;
console.log(argv.x / argv.y);
```

<p>
which complains at you if you don't pass in -x and -y:
</p>

```
$ ./divide.js -x 4.91 -z 2.51
Usage: ./divide.js -x [num] -y [num]
Missing arguments: y
```

<p>
but otherwise just gives you back a plain ol' hash and just works™.
</p>

```
$ ./divide.js -x 55 -y 11
5
```

<p>
<a href="https://github.com/substack/node-optimist"
>Check out the code on github</a>
for more examples and features. I like pull requests!
</p>
