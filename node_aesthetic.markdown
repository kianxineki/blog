I would like to document an emerging set of programming conventions,
philosophies, and values that I see evolving in the
<a href="http://nodejs.org">node.js</a> community.
I call this the node aesthetic.
</p>

<h2>callback austerity</h2>

<p>
The very first example of node you are likely to see is on the
<a href="http://nodejs.org">node.js home page</a>.
This snippet is exemplary of the the radical simplicity pioneered by 
projects like <a href="http://www.sinatrarb.com/">sinatra</a>.
</p>

<code>var http = require('http');
http.createServer(function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(1337, "127.0.0.1");
console.log('Server running at http://127.0.0.1:1337/');
</code>

<p>
Instead of the http server being an external <i>service</i> that we
<i>configure</i> to run our code, it becomes just another tool in our arsenal.
Want to spin up a second http server on a different port? Super easy: just call
<span class="code">http.createServer()</span> once more.
Want to print out how many requests per second you're averaging every minute?
Just plop down a counter and a <span class="code">setInterval()</span>.
Node makes what was previously merely <i>configurable</i> into something more
<i>programmable</i>, <i>orthogonal</i>, and more <i>powerful</i>.
</p>

<p>
A big part of what empowers node to make these kinds of interfaces possible is
its asynchronous nature. No longer do you have statelessness imposed from on
high by the likes of apache or rails. You can keep intermediate state around in
memory just like in any ordinary program. You don't need to reason about
multiple instruction pointers or mutexes or re-entrant, interruptible
execution because all the javascript you write just lives in a single thread.
</p>

<p>
Depending on the event system to <i>already just be there</i> means that the
end-users of modules don't have to think about what hoops they need to jump
through in order to get the event reactor up, running, and plugged into the
component they're trying to use.
The event machinery is all running quietly under the hood without getting in the
way or making a fuss.
</p>

<h2>limited surface area</h2>

<p>
Note also that the http snippet from earlier isn't inheriting from an
<span class="code">http.Server</span> base class or anything of the sort.
The primary focus of most node modules is on <i>using</i>, not <i>extending</i>.
In classical object-oriented design, it's customary to define specific custom
functionality by extending more abstract classes. This idea flows pretty
naturally from thinking about how you might write a compiler to efficiently pack
collections of related properties into memory blocks, but is not such a great
approach for writing <i>usable</i> and perhaps more importantly,
<i>re-usable</i> interfaces.
</p>

<p>
Simple function calls with callback arguments have the superficial but important
benefit of having less exposed surface area and more encapsulation than most
classical designs.
But moreover, when you extend a base class, your class is somewhat
<i>stuck</i> with the interface and internals that the base class uses.
If you want to make things prettier you'll need to shave some yaks to write
wrapper classes just to get around the restrictions of the class system.
Sometimes this classical style of reusability is appropriate, but in node it is
the exception, not the rule, because the implicit consensus is that the
interface is paramount, so usability should trump extensibility.
</p>

<p>
A big part of what makes node modules so great is how they tend to have really
obvious entry points as a consequence of focusing on usability and limited
surface area.
Many modules on <a href="http://npmjs.org">npm</a> export only a single function
by assigning onto <span class="code">module.exports</span> such that
<span class="code">require('modulename')</span> returns just that single
function of interest. It also helps that module names have a direct
correspondence to the string that you put in your call to
<span class="code">require()</span> and that
<span class="code">require()</span> just <i>returns the module</i>
instead of <i>modifying the environment</i> to <i>include the module</i>.
This is known in some circles as doing a <i>qualified import</i>.
In node you can't even do any other kind of import without getting really
hackish about it and <i>this is a very good thing</i>.
This import style means that when you fire up the REPL to play with a new module
when you do <span class="code">require('modulename')</span> you can see
<i>exactly what functionality 'modulename' provides</i>.
Even more importantly, it means that when you read through some new foreign
piece of code that uses many modules, you can tell
<i>exactly where all the module exports came from</i>.
</p>

<h2>batteries not included</h2>

<p>
If software ecosystems are like economies then core libraries are at best
government bureaucracies and at worst nationalized state corporations.
That is, modules in the core distribution get an unfair advantage over the
libraries in userspace by virtue of availability and prominance.
Worse still, because these modules are all maintained in the core project,
contributing, experimenting, and iterating is much harder than with a typical
library on npm. This is especially compounded by the fact that core libraries
are pegged to the core project version and don't often have independent
versioning.
</p>

<p>
Core distributions with too many modules result in neglected code that can't
make meaningful changes without breaking everything. If a neglected library
lives in userspace then at least people can ignore it more easily and the
independent versioning makes necessary breaking changes easier to mitigate for
legacy code.
</p>

<p>
The "batteries included" approach may have struck a worthwhile tradeoff back
when package managers were primitive and modules were global, but that advantage
evaporates in the face of baked-in concurrent library versioning and
sophisticated package management.
</p>

<h2>radical reusability</h2>

<p>
While the limited surface area approach can hurt extensibility,
you can win a great deal of extensibility back by breaking up
problems into lots of tiny modules.
When you write a module for general enough consumption to put up on 
<a href="http://npmjs.org">npm</a>,
you can't contaminate your own implementation with too many
implementation-specific details. It's also much easier to test and iterate on
code in complete isolation from your application business logic.
Node and <a href="http://npmjs.org">npm</a> go to great lengths to help you do
this.
</p>

<p>
In node the only modules that can be said to be "global" are compiled into the
binary itself. There is not really such a thing as a global, system-level module
anymore. Instead, when you run a script that performs a
<span class="code">require()</span>, node will search for the module in a
"node_modules" directory from the directory that the script is in all the way
down to "/", stopping at the nearest "node_modules" directory.
Similarly, when you do an <span class="code">npm install</span>, node will place
the modules into the nearest "node_modules" directory or "./node_modules".
</p>

<p>
All of this preferred locality in the module loader has the very important
practical upshot in that you can depend on <i>different versions of modules</i>
in different places in your program's dependency graph and node will pick the
most local dependency for each module.
So if a module "foo" was tested against and depends on "baz@0.1.x" and a module
"bar" depends on "baz@0.2.x", then when you go to use both "foo" and "bar" in
your own program, "foo" and "bar" will both use the version of "baz" that they
were tested and depend against!
</p>

<p>
This approach to versioning means that nearly all modules should be
interoperable with each other. It also means that module authors can be more
aggressive with iterating and pushing out backwards-incompatible breaking
changes to their APIs, so long as the authors are careful to increment the
module version appropriately.
Concurrent versioning and locality preference drastically accelerate the rate
that we can iterate on new experiments and the levels of reuse that we can
attain both in a single large project and among projects.
</p>

<h2>the future</h2>

<p>
I am wildly optimistic about where this emerging aesthetic of radical
reusability and module-driven development will take node and programming in
general.
</p>
