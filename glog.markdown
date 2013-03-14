This blog just got a new look, thanks in part to
[glog](https://github.com/substack/glog),
a new markdown-powered, git-push blog server I whipped up a while back for the 
[new browserify website](http://browserify.org/).

glog uses [pushover](https://github.com/substack/pushover) to speak git over
http, so you can attach git-based markdown blogging into pretty much any web
server. You can try it yourself, just do:

```
$ git clone http://substack.net/blog.git
```

to download all the blog articles on my new website!
When I want to publish a new article, I just need to commit a new markdown file
to the blog repo, then:

```
$ glog publish article.markdown 'article title goes here'
$ git push publish master --tags
```

I'm hoping that this new blogging platform will cause me to blog more frequently
in tinier tidbits now that it's much easier for me to author, publish, and
revise content.

Check out the source of
[this very blog](https://github.com/substack/substack.net)
and the 
[new browserify website](https://github.com/substack/browserify-website)
for examples of how to use glog in your own site.
