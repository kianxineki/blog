One of the most common objections I've heard about embracing modularity and
favoring libraries [that do a single thing well](http://substack.net/many_things)
is that it can be difficult and time-consuming to find packages for each piece
of functionality you might need for a given task.

This is certainly true at first, but over time and with practice, it is less and
less of a problem as you train up your own heuristics and develop a broad
working memory of useful packages and authors who tend to produce useful code
that suits your own aesthetic preference.

With a bit of training and practice, you will be skimming npm search results at
great speed in no time!

# my heuristic

Here's my own internal heuristic for evaluating npm packages:

* I can install it with npm

* code snippet on the readme using require() - from a quick glance I should see
how to integrate the library into what I'm presently working on

* has a very clear, narrow idea about scope and purpose

* knows when to delegate to other libraries - doesn't try to do too many things itself

* written or maintained by authors whose opinions about software scope,
modularity, and interfaces I generally agree with (often a faster shortcut
than reading the code/docs very closely)

* inspecting which modules depend on the library I'm evaluating - this is baked
into the package page for modules published to npm

When a project tries to do too many things, parts of it will invariably become
neglected as the maintenance burden is unsustainable. The more things a project
tries to do, the easier it is to be completely wrong about some assumption and
this can also lead to abandonment because it's very difficult to revisit
assumptions later.

The best, longest-lasting libraries are small pieces of code that are very
tricky to write, but can be easily verified. Highly mathematical tend to be very
well represented in this category, like the
[gamma function](https://npmjs.org/package/gamma) or an ecosystem of highly
decoupled matrix manipulation modules such as
[ndarray](https://npmjs.org/package/ndarray).

When a library is embedded in an ecosystem of other libraries in a thoroughly
decoupled way, a mutual dynamic results where the main library doesn't need to
inflate its scope but gets enough attention to find subtle bugs while the
dependent libraries can offer excellent interoperability and fit into a larger
informal organizational structure.

# not too important

Here are some things that aren't very important:

* number of stars/forks - often this is a reverse signal because projects with
overly-broad scope tend to get much more attention, but also tend to flame out
and become abandoned later because they take too much effort to maintain over
a long period of time. However! Some libraries are genuinely mistakes but it
took writing the library to figure that out.

* activity - at a certain point, some libraries are finished and will work as
long as the ecosystem around them continues to function. Other libraries do
require constant upkeep because they attack a moving problem but it's important
to recognize which category of module you're dealing with when judging
staleness.

* a slick web page - this is very often (but not always) a sign of a library
that put all of its time into slick marketing but has overly-broad scope. It is
sometimes the case that solid modules also have good web pages but don't be
tricked by a fancy web page where a solid readme on github would do just as good
for a job.

The main crux of this blog post first appeared as a
[reddit comment](http://www.reddit.com/r/javascript/comments/2378xo/the_best_frontend_interview_question_i_got_when/cgv9oky).
