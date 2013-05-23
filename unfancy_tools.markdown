The power of a unix system is how unfancy all the pieces need to be.
When every tool constrains itself to solving a narrow problem well using
, it's very
easy to glue together many commands to acheive 


None of this is to say that we shouldn't be using NEW tools, merely that we
should be using tools with sufficiently narrow scope that are easy to get data
into and out of using typical unix idioms. These kinds of narrowly defined tools
afford us the greatest amount of freedom to recombine according to our whims.

"expressiveness"

People use really fancy tools when there are much simpler alternatives
already provided by the system.

# build

Just use bash.

Building is a sequence of commands to generate some output.

make is really good at incrementally compiling only files that changed.

# command dispatch

It seems like when people are hunting for make alternatives they are actually
hunting for alternatives to using make to map `make NAME` to a shell command.

If you are using node there is already a very nice `npm run NAME` command that
will run NAME from your package.json `"scripts"` field.

# commands

make 

# infinite restart

```
while true; do COMMAND; done
```

