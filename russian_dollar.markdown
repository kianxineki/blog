The <a
href="http://javascriptweblog.wordpress.com/2010/04/27/the-russian-doll-principle-re-writing%C2%A0functions%C2%A0at%C2%A0runtime/">Russian
Doll Pattern</a> occurs when functions replace themselves.
The example in that link uses javascript, but this technique should be
applicable to any language with first-class functions and mutable containers.
</p>

<img
    src="/images/nesting-robots.png" alt="nesting robots"
    class="right-art"
    width="200" height="256"
/>

<p>
Even haskell, with its static types and referential transparency, is capable of
emulating this pattern. Here's an example:
</p>

```
-- <a href="http://substack.net/scripts/RussianDoll.hs">RussianDoll.hs</a>
-- Russian Doll principle in haskell
module Main where

import Control.Concurrent.MVar
import Control.Monad (join)

main :: IO ()
main = do
    fn <- newEmptyMVar
    putMVar fn $ do
        putStrLn "First!"
        swapMVar fn (putStrLn "Again!")
            &gt;&gt; return ()

    join $ readMVar fn
    join $ readMVar fn
    join $ readMVar fn
```

<p>
Which when executed prints:
</p>
```
First!
Again!
Again!
```

<p>
An empty MVar is created and bound to fn.
MVars are especially convenient for this example since they have an empty state
built-in.
The first time fn is executed, "First!" is printed, but then the fn MVar is
swapped for a new action that prints "Again!".
</p>

<p>
The inferred type for fn is MVar (IO ()), so join can be used to execute the
IO () action inside the MVar.
</p>

```
ghci&gt; :t join
join :: (Monad m) =&gt; m (m a) -&gt; m a
```

<p>which means</p>

```
:t join (readMVar (undefined :: (MVar (IO ()))))
join (readMVar (undefined :: (MVar (IO ())))) :: IO ()
```

<p>Neat!</p>

<p>
It's usually a better idea in these cases to use the shared mutable state to delegate to functions instead of storing and swapping the functions themselves.
I am however optimistic that this approach could be used in some code to build a more beautiful abstraction.
</p>
