Here are some of my thoughts on writing tiny reusable modules that each do just
one thing.
These notes started out life as a recent email response, then
[turned into a gist](https://gist.github.com/substack/5075355),
and now has moved to my blog.

***

If some component is reusable enough to be a module then the
maintenance gains are really worth the overhead of making a new
project with separate tests and docs. Splitting out a reusable
component might take 5 or 10 minutes to set up all the package
overhead but it's much easier to test and document a piece that is
completely separate from a larger project. When something is easy, you
do it more often.
 
I've observed the biggest gains for splitting something out into a
separate module when I'm completely stuck mucking around in a larger
application. When I isolate the problem down to the scope of a small
module, it's much easier to see how that small piece fits in with the
greater application objective.
 
Another technique is to pick tools and approaches that don't involve a
lot of boilerplate in the first place. My usual flow for writing a new
module is:
 
0. do a bunch of `npm search`es and ask around on irc to see what similar
modules already exist. If something adequate already exists then you can skip
the rest of the steps!

1. hack up some tiny (`<10` line) example file that does just enough to
start experimenting with the api
 
2. start fleshing out an index.js file required by the example file
 
3. make incremental changes to the index file to get the example
further along, updating the example as necessary to reflect changes
 
4. once the example pretty much works, copy it into test/
 
5. `npm install tap` or `npm install tape` for modules I want to test
in the browser. I like simple, imperative tests that I can adapt from
simple examples.
 
6. add some assertions around the example code
 
7. [`pkginit`](https://github.com/substack/pkginit) to generate a
package.json (or you could just use `npm init`)
 
8. copy the example file into a readme.markdown since I really like
when packages have code in their readmes when I'm looking for modules
to use
 
9. add a blurb at the top of the readme.markdown, document the methods
(there are only a few to document usually), add a license and install
instructions
 
10. create a github repo
 
11. add [travis](http://travis-ci.org) and/or [testling-ci](http://testling.com)
github hooks as appropriate
 
12. `git push` and `npm publish`
 
With practice, I run through all these steps very quickly now. Not all
of these are strictly necessary and sometimes I'll skip a step but
this is mostly the procedure I use for new modules.
 
As much as possible, I try to build large-scale projects using lots of
tiny modules so I just repeat this process whenever I need some
reusable component that doesn't yet exist in quite the form I need it
to exist. As more modules are published to npm I expect I won't need
to write so many modules but there will always be room for new stuff.
 
When applications are done well, they are just the really
application-specific, brackish residue that can't be so easily
abstracted away. All the nice, reusable components sublimate away onto
github and npm where everybody can collaborate to advance the commons.
