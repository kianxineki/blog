Just off the plane from the first ever <a href="http://nodeconf.com/">nodeconf</a>
where I
<a href="http://substack.net/doc/dnode_slides_nodeconf.pdf">gave a talk</a>
about <a href="https://github.com/substack/dnode">dnode</a>
culminating in a
<a href="http://github.com/substack/rap-battle">live dnode-powered markov rap battle</a>,
I headed down to
<a href="http://sf.musichackday.org/2011/">Music Hack Day, San Francisco</a>
with <a href="https://github.com/marak">Marak</a> to represent node.js.
</p>

<p>
Music Hack Day is a 24 hour hacking contest for musical hacks.
I got in late but stayed the whole night and managed to hack up
<a href="https://github.com/substack/singsong">singsong</a>,
interface on top of festival's little-known singing voice synthesizer plugin.
You can click on the staff to plop notes down and then use the text boxes
below to associate bits of text with them.
</p>

<p>
<img src="/images/singsong.png" width="788" height="531">
</p>

<p>
Sample output:
<a href="http://substack.net/audio/singsong.ogg">[ ogg ]</a>
 | 
<a href="http://substack.net/audio/singsong.wav">[ wav ]</a>
</p>

<p>
It's not quite ideal since you've got to break up the words by syllables
yourself. I wrote 
<a href="https://github.com/substack/node-rhyme">a lib</a>
that can do this automatically but didn't hack it in.
Good enough for 24 hours at least.
<a href="https://github.com/substack/singsong">Check it out on github.</a>
</p>
