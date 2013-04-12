Tiny reusable components are great for organizing and reusing front-end code but
some dependencies like jquery will make a tiny frontend module much bigger and
more cumbersome than it needs to be.

Unfortunately introducing a dependency on a big library like jquery in your
frontend code can make it very unattractive for consumers of your module.

As programmers build up applications up from lots of tiny components, if every
component included a vendored jquery version, the total javascript size quickly
baloons into multi-megabytes. When the jquery version gets pegged, version
incompatibilities arise that likewise create redundancy issues. When the jquery
version is left to float or its use is demanded externally in the documentation,
this also does not scale well and makes it cumbersome and error-prone to use
tiny pieces.

Getting away from jquery is attractive even if you have an existing application
where its use is more controlled so that it's only ever used once. When your
code isn't coupled to jquery, it's much easier to spin out the reusable pieces
so you can publish them to package repositories like npm. When it's harder to
split out reusable pieces, those pieces tend to calcify and become tightly
coupled to your application.

But jquery is genuinely very useful and familiar to lots of folks! Fortunately
browsers have taken notice and the DOM itself is a pretty friendly place too
these days.

Here is a list of common jquery features and their raw DOM replacements:

---

To query for elements based on css selectors, instead of:

    $('.foo .bar')

to get a single element, do:

    document.querySelector('.foo .bar')

and to get a list of elements do:

    document.querySelectorAll('.foo .bar')

There's also `element.querySelector{,All}()` if you want the search to begin at an existing element. querySelector and querySelectorAll work in IE8 and up.

---

To register events, in IE9 and up just do:

    element.addEventListener('click', fn)

or for IE8 you can do:

    element.attachEvent('onclick', fn)

---

To set attributes, instead of

    $(element).attr(key, value)

do

    element.setAttribute(key, value)

This works in all browsers.

---

To set css styles, instead of:

    $(element).css(name, value)

you can do:

    element.style[name] = value

This should work in all browsers but note that you'll sometimes need to attach the element to the document body before its `style` property will be set. You can `if (!element.style) element.style = {}` to get around that.

---

To create nodes, instead of:

    var div = $('<div>')

you can do:

    var div = document.createElement('div')

---

To append elements, instead of:

    $(a).append(b)

do:

    a.appendChild(b)

This works in all browsers.

---

To prepend elements, instead of:

    $(a).prepend(b)

do:

    a.insertBefore(b, a.childNodes[0])

This works in all browsers.

---

To update html, instead of:


    $(element).html(htmlString)

do:

    element.innerHTML = htmlString

This works in all browsers.

---

To set the text content of nodes without html tag interpolation, instead of:

    $(element).text(textString)

you can do:

    element.textContent = textString

which works in IE9+ or you can do:

    element.textContent = element.innerText = textString

or you can do:

    element.appendChild(document.createTextNode(textString))

which should both work in all browsers.

---

To remove an element, instead of:

    $(element).remove()

you can do:

    element.parentNode.removeChild(element)

which should work in all browsers.

---

To empty out the inner contents of an element, instead of:

    $(element).empty()

the easiest thing is just to do:

    element.innerHTML = ''

This works in all browsers.

---

For fancy things like animation libraries make sense but for basic stuff like
simple dom manipulation the browser already provides you with some really good
primitives.
