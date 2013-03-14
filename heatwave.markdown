This weekend I took part in
<a href="http://nodeknockout.com">node knockout</a>
along with 
<a href="http://catonmat.net">Peteris Krumins</a>,
<a href="http://rook2pawn.com">David Wee</a>,
and <a href="http://jesusabdullah.github.com/">Josh Holbrook</a>
on
<a href="http://nodeknockout.com/teams/replicants">team replicants</a>.
</p>

<p>
Our entry, <a href="http://heatwave.nodejitsu.com/">heatwave</a>,
uses <a href="http://github.com/substack/node-heatmap">node-heatmap</a>
to draw a heatmap over your code in realtime to show the parts that are most
active as the code executes.
</p>

<p>
Heatwave accomplishes this trick using
<a href="http://github.com/substack/node-bunker">bunker</a>
to add hooks around every expression.
When these hooks get called, the heatmap gets updated!
</p>

<p>
<img src="/images/heatwave/code.png">
</p>

<p>
Check out the <a href="http://heatwave.nodejitsu.com">live demo</a>
and if you like what you see,
<a href="http://nodeknockout.com/teams/replicants">vote for us</a>!
</p>
