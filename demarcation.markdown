If you don't see the point after investigating, that probably means that these tools don't solve problems you personally have. Go with your intuition and don't use them until you feel like you have the problem that they are trying to solve. What might actually be very useful in your current situation is to use a module system to organize your frontend code. Module systems allow you to create structure and organization that can be reused while leveraging the work of tens of thousands of other people in a simple way. I'm personally a fan of npm and node, so I wrote browserify so I could use the node style of doing modules in the browser too and even reuse many of the same modules in both my backend and frontend code. Module systems provide many of the touted architectural benefits of MVC frameworks without as many opinions about how you should structure your files and a much better strategy for interoperability with third-party code.

There are many demarcation issues that come up when modules try to do many unrelated things instead of doing exactly one thing well:
turf wars - what belongs and what doesn't belong? In your other post you mention
underscore, which has extremely arbitrary ideas about belongs in core. There's
_.template() but no _.maxBy(). The steady state for modules is to do one thing
or to do every possible thing. On the maintainer side, policing very arbitrary
scope boundaries creates burn out constantly re-hashing ad-hoc justifications
with appeals to personal aesthetics.


discoverability - It's really hard to find and reuse code if it's buried in some big collection of utilities. The other stuff will either get in your way or you won't even discover that the big module does the thing you want in the first place because it doesn't come up in the npm search or github search output.
hard to contribute - the bigger a project becomes, the harder it gets for newcomers to submit meaningful contributions. There is more process, more surface area, and more opportunity to break things when projects attain a certain scale.
granular versioning - If you use a grab bag utility library that lumps a bunch of features together, the semantic versioning becomes less useful. Did that breaking change in a major version bump only affect some function you weren't even using? You've got to dive in and inspect the changelog yourself. With smaller modules, it's much more clear what you need to update when you upgrade a library version.
For some more context, here's how these arguments have played out before: https://github.com/substack/node-mkdirp/issues/17

https://github.com/jashkenas/backbone/issues/2888
