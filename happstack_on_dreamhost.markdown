I had been meaning to give happstack a closer look for far too long and decided
the best way to force myself to get around to putting together something
interesting would be to get some cheap webhosting.

I've installed the ghc and cabal toolchain in non-root user accounts before, so
I figured anything that offers shell accounts would be sufficient for my
purposes. On IRC, pkrumins suggested Dreamhost, which he uses for catonmat.net.
Dreamhost has shell accounts and looked cheap enough, so I registered
substack.net under the cheapest "happy hosting" account.

Once I got a shell, I downloaded the binary build of ghc for x86_64 into my home
directory, extracted it, and configured it

```
./configure --prefix=$HOME/prefix && make install
```

Unfortunately, the system strip command used by the installer was incompatible
with the binaries. Downloading and installing the latest version of GNU binutils
into my home directory with the strip in my $PATH, I tried to install ghc again
with exactly the same errors as before.

Frustratingly, ghc's installer hard-codes "/usr/bin/strip" instead of checking
$PATH for it. I replaced every hard-coded /usr/bin/strip occurence to the
location of my own version of strip from binutils with this one-liner:

```
perl -pi -e's{/usr/bin/strip}{$ENV{HOME}/prefix/bin/strip}g' \
    `grep '/usr/bin/strip' -lR .`
```

`make install` succeeded after this.

I downloaded and extracted cabal-install, ran bootstrap.sh, and had cabal up and
running. Unfortunately, while installing many packages with cabal, Dreamhost's
memory checker would kill the process for using too much memory. Whenever this
happened, I would go into the directory of the troublesome package in
~/.cabal/packages/hackage.haskell.org/, then I would

```
tar xzf $PKG.tar.gz
cd $PKG
ghc --make Setup.hs && ./Setup configure --user && ./Setup build && ./Setup install
```

If the compile was killed, I just typed ./Setup build again until it finally
finished all the way through. In this manner I was able to get most of the
modules necessary to run happstack installed.

To install packages which depended on libraries and include files, I modified
~/.cabal/config by uncommenting and setting:

```
extra-include-dirs: $PREFIX/include
extra-lib-dirs: $PREFIX/lib
```

Not all modules will actually respect these directives, however. I had to hack
away at some cabal files to get all the modules I needed running.

FileManip was super annoying to get installed. I finally went into its source
and replaced "Control.Exception" with "Control.OldException", which did the
trick.

Running happstack applications in the usual way where the service provides its
own http server would only work on a Dreamhost Private Server account, which
costs more money. Dreamhost does support CGI and FastCGI on the cheapest kind of
account, however. I recommend using the plain CGI interface that
Happstack.Server.FastCGI provides through Network.FastCGI, since errors show up
properly in error.log and process management is much simpler.

While non-trivial to get running, this web experiment proves that it is at the
very least possible to run Happstack applications on cheap hosting providers
such as Dreamhost with little more than a shell account and CGI support. The
source to substack.net is available on github, if you'd like to poke around!
