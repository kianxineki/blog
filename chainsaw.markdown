I write a lot of modules with chainable interfaces, if you haven't already
noticed. I found myself writing these things so often, I wrote a module to make
writing modules with chainable interfaces super simple.
<a href="http://github.com/substack/node-chainsaw">Introducing chainsaw</a>!
</p>

<img src="/images/chainsaw.png" width="284" height="416" class="right-art">

<p>
"Chainable interfaces", you might be thinking... to yourself... in my voice...
"Those are easy, just
<span class="code">return this</span> and you're done, right?"
</p>

<p>
Well, only sometimes!
Very often you'll want to compose chains out of asynchronous actions that
haven't returned yet, so you'll need to push all the actions to a stack and then
process them one at a time as the results become available.
With chainsaw, all of that chain-stack busywork is taken care of!
</p>

<p>
In this silly example, the AddDo() function takes an initial sum and then
modifies that sum in chainable form through <span class="code">.add()</span>
and <span class="code">.do()</span>.
</p>

<code>
var Chainsaw = require('chainsaw');

function AddDo (sum) {
    return Chainsaw(function (saw) {
        this.add = function (n) {
            sum += n;
            saw.next();
        };
        
        this.do = function (cb) {
            saw.nest(cb, sum);
        };
    });
}
</code>

<p>
Then we can use the streaming interface like this:
</p>

<code>
AddDo(0)
    .add(5)
    .add(10)
    .do(function (sum) {
        if (sum &gt; 12) this.add(-12).add(2);
    })
    .do(function (sum) {
        console.log('Sum: ' + sum);
    })
;
</code>

<p>
Note that in the <span class="code">.do()</span> blocks, further commands can be
nested! <span class="code">saw.nest()</span> takes care of all the complexity
for making nested chains work.
</p>

<p>
And now for a more real-world, asynchronous example: consider this module that
provides a chainable interface on top of
<a href="https://github.com/pkrumins/node-lazy">node-lazy</a>
to read lines.
</p>

<code>
var Chainsaw = require('chainsaw');
var Lazy = require('lazy');

module.exports = Prompt;
function Prompt (stream) {
    var waiting = [];
    var lines = [];
    var lazy = Lazy(stream).lines.map(String)
        .forEach(function (line) {
            if (waiting.length) {
                var w = waiting.shift();
                w(line);
            }
            else lines.push(line);
        })
    ;
    
    var vars = {};
    return Chainsaw(function (saw) {
        this.getline = function (f) {
            var g = function (line) {
                saw.nest(f, line, vars);
            };
            
            if (lines.length) g(lines.shift());
            else waiting.push(g);
        };
        
        this.do = function (cb) {
            saw.nest(cb, vars);
        };
    });
}
</code>

<p>
And now we can read stdin with a bunch of <span class="code">.getline()</span>s
chained together!
</p>

<code>
var stdin = process.openStdin();
Prompt(stdin)
    .do(function () {
        util.print('x = ');
    })
    .getline(function (line, vars) {
        vars.x = parseInt(line, 10);
    })
    .do(function () {
        util.print('y = ');
    })
    .getline(function (line, vars) {
        vars.y = parseInt(line, 10);
    })
    .do(function (vars) {
        if (vars.x + vars.y < 10) {
            util.print('z = ');
            this.getline(function (line) {
                vars.z = parseInt(line, 10);
            })
        }
        else {
            vars.z = 0;
        }
    })
    .do(function (vars) {
        console.log('x + y + z = ' + (vars.x + vars.y + vars.z));
        process.exit();
    })
;
</code>

<p>
Check out <a href="https://github.com/substack/node-chainsaw"
>the code on github</a> for more.
You can install chainsaw with
<a href="https://github.com/isaacs/npm">npm</a>
by doing:
</p>

<code>
npm install chainsaw
</code>

<p>
Happy, um... hacking!
</p>
