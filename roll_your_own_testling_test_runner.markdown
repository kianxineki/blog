<a href="http://testling.com/">Testling</a>
is a browserling product that
<a href="http://catonmat.net">pkrumins</a>
and I put together to make running
automated browser tests super easy.
</p>

<p>
Tests usually look something like this:
</p>

<code>
var test = require('testling');

test('json parse', function (t) {
    t.deepEqual(
        Object.keys({ a : 1, b : 2 }),
        [ 'a', 'b' ]
    );
    t.end();
}); 
</code>

<p>
Then you can run the tests on
<a href="http://testling.com/browsers/">all the browsers we run</a>
using a curl one-liner like this one:
</p>

<code>
curl -sSNT test.js -u mail@substack.net \
  'http://testling.com/?browsers=chrome/16.0,firefox/9.0,safari/5.1,ie/9.0,ie/7.0'
</code>

<p>
and when the code blows up
(in IE7 in this case because it doesn't have Object.keys),
you get a full stack trace!
</p>

<code>
$ curl -sSNT test.js -u mail@substack.net \
  'http://testling.com/?browsers=chrome/16.0,firefox/9.0,safari/5.1,ie/9.0,ie/7.0'
Enter host password for user 'mail@substack.net':
Bundling...  done

chrome/16.0         1/1  100 % ok
firefox/9.0         1/1  100 % ok
safari/5.1          1/1  100 % ok
iexplore/9.0        1/1  100 % ok
iexplore/7.0        0/1    0 % ok
  Error: [object Error]
    at [anonymous]() in /test.js : line: 4, column: 5
    at keys() in /test.js : line: 5, column: 9
    at [anonymous]() in /test.js : line: 3, column: 29
    at test() in /test.js : line: 3, column: 1

  &gt; t.deepEqual(
  &gt;     Object.keys({ a : 1, b : 2 }),
  &gt;     [ 'a', 'b' ]
  &gt; );


total               4/5   80 % ok
</code>

<p>
Wow super great! Except perhaps you don't like how the standard test API looks
and want something more jasmine-esque and bdd-ish.
</p>

<p>
Just write a little wrapper like this:
</p>

<code>
var testling = require('testling');

module.exports = function describe (dname, cb) {
    if (typeof dname === 'function') {
        dcb = dname;
        dname = undefined;
    }
    
    var ix = 0;
    cb(function it (iname, cb) {
        var name = (dname ? dname + ' :: ' : '') + (iname || 'test #' + ix++);
        
        testling(name, function (t) {
            var waiting = false;
            t.wait = function () { waiting = true };
            cb(t);
            if (!waiting) t.end();
        });
    });
};
</code>

<p>
And now you can write your tests like so,
<span class="code">require()</span>ing the bdd wrapper node.js-style:
</p>

<code>
var describe = require('./bdd');

describe('arrays', function (it) {
    it('should map', function (t) {
        t.deepEqual(
            [ 97, 98, 99 ].map(function (c) { return String.fromCharCode(c) }),
            [ 'a', 'b', 'c' ]
        );
    });
    
    it('can do indexOf', function (t) {
        t.equal([ 'a', 'b', 'c', 'd' ].indexOf('c'), 2);
    });
});

describe('tests', function (it) {
    it('can wait', function (t) {
        t.wait();
        
        var start = Date.now();
        setTimeout(function () {
            var elapsed = Date.now() - start;
            t.log(elapsed);
            t.ok(elapsed &gt;= 100);
            t.end();
        }, 100);
    });
});
</code>

<p>
Now you have 2 files but you can still use a one-liner to stitch everything
together:
</p>

<code>
$ tar -cf- bdd.js test.js | curl -sSNT- -u mail@substack.net \
  'testling.com/?browsers=chrome/16.0,firefox/9.0,iexplore/9.0'
Enter host password for user 'mail@substack.net':
Bundling...  done

chrome/16.0         3/3  100 % ok
  Log: 101
firefox/9.0         3/3  100 % ok
  Log: 115
iexplore/9.0        2/3   66 % ok
  Log: 83
  Assertion Error: not ok
    at [anonymous]() in /test.js : line: 24, column: 13
    at ok() in /test.js : line: 24, column: 13
    at [anonymous]() in /test.js : line: 21, column: 29
    at setTimeout() in /test.js : line: 21, column: 9
    at [anonymous]() in /test.js : line: 17, column: 29
    at it() in /test.js : line: 17, column: 5
    at [anonymous]() in /test.js : line: 16, column: 28

  &gt; t.ok(elapsed &gt;= 100);


total               8/9   88 % ok
</code>

<p>
...and strangely enough, IE9 only sleeps for 83 milliseconds when you tell it to
sleep for 100! TYPICAL.
</p>

<div>
<img src="/images/saucer_landing.png">
</div>

<p>
Check out the <a href="http://testling.com/docs/">testling documentation</a>,
<a href="http://browserling.com/#/create">create a browserling account</a>
to use with testling,
and hack up some crazy browser tests and test runners!
</p>

<div>
<a href="/images/browsers/war_of_the_browsers.png"><img src="/images/browsers/war_of_the_browsers_medium.png"></a>
</div>
