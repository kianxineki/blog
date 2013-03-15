# so it is

As a true believer in the UNIX way of tiny self-contained modules that each do
just one thing, I find myself at a fundamental disagreement when talking with
IDE-adherents or framework-oriented programmers.

When abstractions are kept to the minimum necessary, it's much easier to see
which ones are good ideas and which ones are mistaken ideas. With larger
collections of ideas branded into cohesive framework ideologies, it's very easy
for unjustified abstractions to sneak in under the cover of the more useful
pieces.

So it is with many things.

Do one thing well even if that one thing turns out to be a bad idea.
Publish your mistakes to save the next person who gets the same idea about how
things could be the trouble of re-implementing the same mistakes.

# monopoly

It used to be the languages had a monopoly on abstractions. The standard
distribution would bundle some handy libraries to do common things like option
parsing, making http requests, and sending email. This made sense at the time.
When you installed a distribution of UNIX or Linux you get the BSD or GNU
toolchain of handy utilities so the same logic should apply.

But these standard libraries grew rotten.
Nobody could give them the love and care they needed because the core team was
too busy evolving the language, iterating on the runtime, and improving
performance. The core team forgot about poor little distutils.

It's actually far worse than merely forgetting about a neglected expanded core:
the versioning on core modules is intrinsically linked to the distribution
release cycle as a whole so making any changes becomes a beaurocratic protracted
struggle that few potential contributors have the patience for. Warts
accumulate. Code calcifies around bugs that make it impossible to make necessary
changes without breaking everything.

# pulling from the outside

The reason bundles can work for system distributions is that the people working
on the distributions are just pulling in from external resources, not pushing
the new iterations themselves. The work happens outside from the community at
large and is pulled in. This independence makes it much less daunting to fix
bugs, improve APIs, and experiment.

Need to make a breaking change? No problem, just bump the major version. Trust
that any packages downstream will be sufficiently careful with their dependency
semvers.

# welded shut

Frameworks side-step the versioning problem that core platforms have
by adopting the system distribution model for bundling up handy, common
utilities in a convenient way.

However, when you choose a framework, you trade away some temporary convenience
for long-term flexibility. This may be an appealing bargain, particularly for
short-lived applications as it imposes an immediate sense of structure on an
otherwise blank slate that might very indeed get software built before a
deadline.

However, it's much harder to change your mind about a choice that you never made
yourself. When frameworks make your decisions for you, you very often won't even
realize that a decision has been made at all so it's much harder to identify
problems when the assumptions grounded in that technology choice no longer apply.

Another problem with short-sighted calculations is that it's very hard to know
which applications really are one-offs and which ones will have a surprisingly
enduring lifespan. And when a project "ends", it doesn't usually "end". It's
just that the current contract is up, so any improvements will need to be made
in a separate contract with perhaps a different team at the helm. Extend empathy
to next person to work on some code, because that next person could be you!

An argument I often hear in favor of frameworks is that it helps a big team
rally around some common tools that everybody is familiar with.
This is true, but I often observe the same benefits collaborating with people
who use some of my favorite low-level stream modules like
[through](https://github.com/dominictarr/through)
and [duplexer](http://github.com/Raynos/duplexer)
or when authors use interfaces from node core like `.pipe()`.

To be fair, there is a distinction here between frameworks that are sheer
mudballs of compounded assumptions and frameworks that take the time to
carefully factor out themselves into lots of tiny pieces.
I think an important standard to judge frameworks is how many of the framework's
core abstractions can be used ala-carte without using the framework itself.
If your framework melts away into an informal collection of modules that happen
to work well together but can be easily repurposed by people who don't use the
framework, then you have built something very sublime.

# cooking up some modules

If some problems naturally turn out to
[naturally trend towards monopoly](https://en.wikipedia.org/wiki/Natural_monopoly)
then that is something that we should notice and accept.
However, I expect that the number of these natural monopolies which actually
exist is far lower than many people assume right now.

Tools like [github](https://github.com) and free-for-all package managers
like [npm](https://npmjs.org) continue to remove barriers to entry to such a
point that the fitness landscape of software is a fundamentally different place
than just 10 years ago.

What we need right now is not more frameworks to give a collection of modules a
cohesive "brand" with divisive opinions about what "belongs". Instead, we need
more recipes, articles, talks, and screencasts about how all these pieces fit
together.

We need to be better about how to go from problem A to solution B by way of
modules X, Y, and Z.

There should be more public discussion about what modules to use for which
problems and when.

There should be more meta-analysis too about different fundamental design axioms
at work behind certain clusters of libraries versus other clusters.

Keep publishing modules and learning new facts by experiment of course, but if
we are going to obtain this lofty goal set forth by
[ryan dahl](http://tinyclouds.org/)

```
I want programming computers to be like coloring with crayons and playing with
duplo blocks.
```

then we'll need more than code. We'll need documentation and community!
