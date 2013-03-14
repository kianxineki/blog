Introducing <a href="http://github.com/substack/node-heatmap">node-heatmap</a>,
a nifty node.js module for making pretty heatmaps.

You can even
<a href="https://github.com/substack/node-heatmap/blob/master/example/web/main.js">use node-heatmap in the browser</a>
with <a href="https://github.com/substack/node-browserify">browserify</a>.
</p>

<p>
Here's an example of using heatmap in node with some random values biased
towards the center of the canvas:
</p>

<code>
var heatmap = require('heatmap');

var heat = heatmap(500, 500, { radius : 30 });
for (var i = 0; i < 5000; i++) {
    var rho = Math.random() * 2 * Math.PI;
    var z = Math.pow(Math.random(), 2) * 200;

    var x = 250 + Math.cos(rho) * z;
    var y = 250 + Math.sin(rho) * z;

    heat.addPoint(x, y);
}
heat.draw();

var fs = require('fs');
fs.writeFileSync('blob.png', heat.canvas.toBuffer());
</code>

<p>
When the example calls <span class="code">.addPoint(x,y)</span>, heatmap draws a
rectangle with a radial gradient and outwardly diminishing alpha mask so that
the alpha values can pile up as nearby points are added. With Canvas this is
surprisingly easy to do:
</p>

<code>
Heat.prototype.addPoint = function (x, y, radius) {
    var ctx = this.alphaCanvas.getContext('2d');
    
    var g = ctx.createRadialGradient(x, y, 0, x, y, radius);
    var a = 1 / 10;
    
    g.addColorStop(0, 'rgba(255,255,255,' + a + ')');
    g.addColorStop(1, 'rgba(255,255,255,0)');
    
    ctx.fillStyle = g;
    ctx.fillRect(x - radius, y - radius, radius * 2, radius * 2);
};
</code>

<p>
I got the idea to use a secondary canvas's alpha channel with radial gradients
from <a href="https://github.com/pa7/heatmap.js">heatmap.js</a>,
which is yet another heatmap module which may be of interest.
</p>

<p>
Later when it comes time to <span class="code">.draw()</span> onto the primary
canvas, I just use
<a href="https://github.com/harthur/color-convert">color-convert</a>
to convert an
<a href="http://en.wikipedia.org/wiki/HSL_and_HSV">HSL</a>
coordinate to rgb.
<a href="http://en.wikipedia.org/wiki/HSL_and_HSV">HSL</a>
is super neat because the hue is just an angle, so a loop of
<span class="code">i += 30</span> from 0 to 360 degrees hits all the rainbow
colors along the way.

All <span class="code">.draw()</span> needs to do is
<a href="https://github.com/substack/node-heatmap/blob/3338a4f4244cca13f0ab07555984ddebba8bd7c7/index.js#L79-L80">map the composite alpha value as the HSL
hue</a>
at every pixel onto the RGB of the primary canvas.
Easy!
</p>

<p>
And here's what the final product from the example code at the start looks like:
</p>

<div>
<img src="/images/heatmap.png" width="500" height="500">
</div>

<p>
Radical.
<a href="http://github.com/substack/node-heatmap">Check out the code on github</a>
and <a href="http://npmjs.org">npm install heatmap</a>!
</p>

<p>
<a href="http://github.com/substack/mrcolor">Mr. Color</a> approves of this
module.
</p>

<div>
<img src="/images/mrcolor.png" width="390" height="799">
</div>
