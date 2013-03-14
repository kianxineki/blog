The haskell
<a href="http://hackage.haskell.org/package/gd-3000.4.0">gd module</a>
provides bindings to a small but useful subset of <a
href="http://www.libgd.org/">libgd</a>.

It's a nice enough module and I am grateful that
<a href="http://bringert.net/">Björn Bringert</a>
took the time to put it together but it's just not... <i>functional</i> enough
for my tastes. Consider setPixel.
</p>

<code>
setPixel :: Point -&gt; Color -&gt; Image -&gt; IO ()
</code>

<p>
This function takes a point, a color, and an image, returning the
<a href="http://en.wikipedia.org/wiki/Unit_type">unit type</a> from the IO
monad, which means that somewhere inside of Image there must lie a mutable
reference. As good as GHC's garbage collection and algebraic trickery are for
optimizing away many sorts of unused intermediate state,
I can understand the appeal of these in-place updates, especially
considering how much the underlying C library caters to this programming style.
</p>

<p>
Not all of the actions are performed in-place, however.
<p>

<code>
rotateImage :: Int -&gt; Image -&gt; IO Image
</code>

<p>
The rotateImage function creates and returns a new Image type which has been
rotated some integer number of degrees. However, this return type is still
wrapped up in the IO monad and threading this Image type around looks tedious.
Luckily, there is a more functional way to handle both types of updates efficiently
while providing a handy way to automatically thread the state at the same time.
</p>

<code>
-- <a href="http://substack.net/scripts/haskell-gd/State.hs">State.hs</a>
module Graphics.GD.State where

import Control.Monad.State.Lazy (State(..),modify,execState)
import qualified Graphics.GD as GD

data GDCmd = SetPixel GD.Point GD.Color

data GD' = GD' { gdCmds :: [GDCmd] }
type GD a = State GD' a
</code>

<p>
Here, the GD type lets us create State monads that operate on GD' data.
The GD' type contains the image and a list of operations to perform on the
image. For simplicity, only SetPixel is defined. With these types, we can write
a helper function consCmd that adds a command to the gdCmds of the state and a
function setPixel that registers a SetPixel action.
</p>

<code>
consCmd :: GDCmd -&gt; GD ()
consCmd cmd = modify $ \gd -&gt; gd { gdCmds = cmd : (gdCmds gd) }

setPixel :: GD.Point -&gt; GD.Color -&gt; GD ()
setPixel = (consCmd .) . SetPixel
</code>

<p>
Finally, a newImage function is defined that takes a size and GD () action
and executes the commands in the IO monad using runCmd.
</p>

<code>
newImage :: GD.Size -&gt; GD () -&gt; IO GD.Image
newImage size f = do
    im &lt;- GD.newImage size
    -- run each of the commands for the image
    mapM_ (flip runCmd $ im) $ gdCmds $ execState f $ GD' []
    return im

runCmd :: GDCmd -&gt; GD.Image -&gt; IO ()
runCmd (SetPixel pt c) = GD.setPixel pt c
</code>

<p>
With just these pieces, it's possible to build something useful!
This program creates a file noise.png filled with random color values.
</p>

<code>
-- <a href="http://substack.net/scripts/haskell-gd/Main.hs">Main.hs</a>
module Main where 

import Graphics.GD.State
import qualified Graphics.GD as GD
import System.Random (newStdGen,randomRs)
import Control.Monad (mapM_,liftM2)

main = do
    g &lt;- newStdGen
    let (w,h) = (400,300)
    (GD.savePngFile "noise.png" =&lt;&lt;) . newImage (w,h) 
        $ mapM_ (uncurry setPixel)
        $ zip (liftM2 (,) [0..w-1] [0..h-1]) -- all the pixel coordinates
        $ map fromInteger $ randomRs (0,256^3-1) g -- random colors
</code>

<br>
<img src="/images/haskell-gd/noise.png" width="400" height="300">

<p>
The code here is just enough to demonstrate a use of the state monad to
create a mini-domain specific language on top of another library.
I forked dancor's <a href="http://github.com/dancor/haskell-gd/">haskell-gd</a>
to include a more complete version of this Graphics.GD.State module.
You can <a href="http://github.com/substack/haskell-gd">check out the code
here</a>.
</p>

<p>
With the extended module, here's code that 
<a href="http://github.com/substack/haskell-gd/blob/master/examples/gradient.hs"
>computes a gradient</a>:
</p>

<code>
import Graphics.GD.State

main = (savePngFile "gradient.png" =&lt;&lt;) . newImage (400,300) $ do
    (w,h) &lt;- getSize
    eachPixel $ \(x,y) -&gt;
        let
            r = ((128 * x) `div` w) + ((128 * y) `div` h)
            g = 127 + ((128 * x) `div` w)
            b = 127
        in rgb r g b
</code>

<br>
<img src="/images/haskell-gd/gradient.png" width="400" height="300">

<p>
And this one draws a
<a href="http://github.com/substack/haskell-gd/blob/master/examples/circle.hs"
>circle and a line</a>:
</p>

<code>
import Graphics.GD.State
 
main = (savePngFile "circle.png" =&lt;&lt;) . newImage (400,300) $ do
    (w,h) &lt;- getSize

    fill $ rgb 100 63 127 -- dark purple background

    drawArc
        (w `div` 2,h `div` 2) -- centered
        (180,180) -- (width,height)
        0 360 -- a circle
        (rgb 255 255 255) -- white

    drawLine (0,0) (w-1,h-1) (rgb 127 255 127)
</code>

<br>
<img src="/images/haskell-gd/circle.png" width="400" height="300">

<p>
This might seem like a lot of effort for something that doesn't matter very
much, but writing beautiful code is important.
Clean, abstract interfaces save precious mental horsepower for the bigger,
harder problems.
</p>

<p>
"Beauty is a consequential thing, a product of solving problems correctly."
-- <a href="http://www.greatbuildings.com/buildings/Cary_House.html"
>Joseph Esherick</a>
</p>
