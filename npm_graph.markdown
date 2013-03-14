The node.js community has been growing like crazy with 995 packages on
<a href="http://npmjs.org">npm</a> at the time of this writing.
</p>

<code>
$ npm --loglevel=silent ls latest | wc -l
995
</code>

<p>
For fun I graphed the dependencies between npm modules with
<a href="https://github.com/glejeune/node-graphviz">graphviz</a>.
Here are the results with the fdp and twopi graphing algorithms:
</p>

<p>
<a href="http://substack.net/images/npmdep/npm-fdp.png">
<img src="http://substack.net/images/npmdep/npm-fdp-prev.png" width="499" height="835">
</a>
</p>

<p>
<a href="http://substack.net/images/npmdep/npm-twopi.png">
<img src="http://substack.net/images/npmdep/npm-twopi-prev.png" width="499" height="497">
</a>
</p>

<p>
<a href="http://github.com/substack/npmdep">Check out the code on github</a>
Or <span class="code">npm install npmdep</span>!
</p>
