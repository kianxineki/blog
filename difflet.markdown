Introducing <a href="https://github.com/substack/difflet">difflet</a>,
a handy node.js module for computing pretty object diffs!
</p>

<p>
Just plug in some initial options and the objects you want to compare!
</p>

<code>
var difflet = require('difflet');
var s = difflet({ indent : 2, comment : true }).compare(
    { z : [6,7], a : 'abcdefgh', b : [ 31, 'xxt' ] },
    { x : 5, a : 'abdcefg', b : [ 51, 'xxs' ] }
);
console.log(s);
</code>

<p>
and you'll get a colored and annotated object expressing the differences between
the 2 objects you passed in:
</p>

<div>
<img src="http://substack.net/images/screenshots/difflet_object_comments.png"
width="433" height="164">
</div>

<p>
Deleted elements between the first and second object are shown in red and
commented if comments are turned on.
New object show up in green and updated objects show in blue.
The comments show what the previous value was, if any.
</p>

<p>
You can set a bunch of options to adjust the formatting including comma-first
output. You can even generate HTML output:
</p>

<code>
var difflet = require('difflet');
var ent = require('ent');

var tags = {
    inserted : '&lt;span class="g"&gt;',
    updated : '&lt;span class="b"&gt;',
    deleted : '&lt;span class="r"&gt;',
};
var diff = difflet({
    start : function (t, s) {
        s.write(tags[t]);
    },
    stop : function (t, s) {
        s.write('&lt;/span&gt;');
    },
    write : function (buf, s) {
        s.write(ent.encode(buf))
    },
});

var prev = {
    yy : 6,
    zz : 5,
    a : [1,2,3],
    fn : function qq () {}
};
var next = {
    a : [ 1, 2, 3, [4], "z", /beep/, new Buffer([0,1,2]) ],
    fn : 'I &lt;3 robots',
    b : [5,6,7]
};
diff(prev, next).pipe(process.stdout);
</code>

<p>
which generates some HTML output that you can stuff into a browser:
</p>

<code>
$ node example/html.js 
{&amp;quot;a&amp;quot;:[1,2,3,&lt;span class="g"&gt;[4]&lt;/span&gt;,&lt;span class="g"&gt;&amp;quot;z&amp;quot;&lt;/span&gt;,&lt;span class="g"&gt;/beep/&lt;/span&gt;,&lt;span class="g"&gt;&amp;lt;Buffer 00 01 02&amp;gt;&lt;/span&gt;],&amp;quot;fn&amp;quot;:&lt;span class="b"&gt;&amp;quot;I &amp;lt;3 robots&amp;quot;&lt;/span&gt;,&lt;span class="g"&gt;&amp;quot;b&amp;quot;:[5,6,7]&lt;/span&gt;,&lt;span class="r"&gt;&amp;quot;yy&amp;quot;:6,&amp;quot;zz&amp;quot;:5&lt;/span&gt;}
</code>

<p>
Plus,
<a href="http://catonmat.net">pkrumins</a> and I just rolled out difflet output
for <a href="http://testling.com">testling</a> to make debugging
<span class="code">t.deepEqual()</span> statements on big objects easier:
</p>

<div>
<img src="http://substack.net/images/screenshots/testling_using_difflet.png"
width="681" height="766">
</div>

<p>
<a href="https://github.com/isaacs/node-tap">node-tap</a> is now using
difflet too in 0.2.1!
</p>

<div>
<img src="http://substack.net/images/screenshots/tap_using_difflet.png"
width="681" height="766">
</div>

<p>
<a href="https://github.com/substack/difflet">Check out the code on github</a>
or with <a href="http://npmjs.org">npm</a> do:
</p>

<code>
npm install difflet
</code>
