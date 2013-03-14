Instead of doing any real work tonight, I wrote
<a href="https://github.com/substack/npmtop">npmtop</a>!
npmtop is a silly program that ranks npm contributors by the number of packages
they've writen.
</p>

<code>
$ npmtop
rank   percent   packages   author
----   -------   --------   ------
#  1    3.05 %      28      tjholowaychuk
#  2    2.83 %      26      samuraijack
#  3    2.29 %      21      gozala
#  4    1.96 %      18      isaacs
#  5    1.96 %      18      creationix
#  6    1.74 %      16      substack
#  7    1.63 %      15      kriskowal
#  8    1.52 %      14      marak
#  9    1.41 %      13      coolaj86
# 10    1.41 %      13      pkrumins
# 11    1.20 %      11      masylum
# 12    1.09 %      10      TooTallNate
# 13    0.98 %       9      davglass
# 14    0.98 %       9      mikebannister
# 15    0.98 %       9      cloudhead
</code>

<p>
You can see where you stand individually in the rankings:
</p>

<code>
$ npmtop substack
rank   percent   packages   author
----   -------   --------   ------
#  6    1.74 %      16      substack
</code>

<p>
Here are the top 50, why not:
</p>

<code>
$ npmtop 50
rank   percent   packages   author
----   -------   --------   ------
#  1    3.05 %      28      tjholowaychuk
#  2    2.83 %      26      samuraijack
#  3    2.29 %      21      gozala
#  4    1.96 %      18      creationix
#  5    1.96 %      18      isaacs
#  6    1.74 %      16      substack
#  7    1.63 %      15      kriskowal
#  8    1.52 %      14      marak
#  9    1.41 %      13      coolaj86
# 10    1.41 %      13      pkrumins
# 11    1.20 %      11      masylum
# 12    1.09 %      10      TooTallNate
# 13    0.98 %       9      indexzero
# 14    0.98 %       9      mikebannister
# 15    0.98 %       9      Tim-Smart
# 16    0.98 %       9      davglass
# 17    0.98 %       9      cloudhead
# 18    0.87 %       8      nathan
# 19    0.87 %       8      cohara87
# 20    0.87 %       8      chrisdickinson
# 21    0.76 %       7      tmpvar
# 22    0.76 %       7      TrevorBurnham
# 23    0.65 %       6      felixge
# 24    0.65 %       6      bnoguchi
# 25    0.65 %       6      fedor.indutny
# 26    0.65 %       6      mikeal
# 27    0.65 %       6      caolan
# 28    0.65 %       6      mape
# 29    0.65 %       6      astro
# 30    0.54 %       5      kof
# 31    0.54 %       5      weepy
# 32    0.54 %       5      andris
# 33    0.54 %       5      akaspin
# 34    0.54 %       5      s3u
# 35    0.54 %       5      ncb000gt
# 36    0.54 %       5      rauchg
# 37    0.54 %       5      bradleymeck
# 38    0.54 %       5      gf3
# 39    0.54 %       5      as-jpolo
# 40    0.54 %       5      naitik
# 41    0.54 %       5      aredridel
# 42    0.44 %       4      weaver
# 43    0.44 %       4      technoweenie
# 44    0.44 %       4      bartt
# 45    0.44 %       4      dandean
# 46    0.44 %       4      sjs
# 47    0.44 %       4      jamescarr
# 48    0.44 %       4      afelix
# 49    0.44 %       4      aconbere
# 50    0.44 %       4      broofa
</code>

<p>
That is a lot of packages!
</p>

<div style="text-align: center">
<img src="/images/npmtop.png" width="510" height="684">
</div>

<code>
npm install npmtop
</code>

<p>
<a href="https://github.com/substack/npmtop">get it on github</a>
</p>

