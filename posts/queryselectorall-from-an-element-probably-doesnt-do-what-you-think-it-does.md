<!--
.. title: querySelectorAll from an element probably doesn't do what you think it does
.. slug: queryselectorall-from-an-element-probably-doesnt-do-what-you-think-it-does
.. date: 2015-08-21 12:11:23 UTC-07:00
.. tags: css
.. category:
.. link:
.. description:
.. type: text
-->

Modern browsers have APIs called `querySelector` and `querySelectorAll`. They
find one or more elements matching a CSS selector. I'm assuming basic
familiarity with CSS selectors: how you select elements, classes and ids. If
you haven't used them, start with the [MDN introduction][csssel].

Imagine the following HTML page:

```html
<!DOCTYPE html>
<html>
<body>
    <img id="outside">
    <div id="my-id">
        <img id="inside">
        <div class="lonely"></div>
        <div class="outer">
            <div class="inner"></div>
        </div>
    </div>
</body>
</html>
```

`document.querySelectorAll("div")` returns a `NodeList` of all of the `<div>`
elements on the page. `document.querySelector("div.lonely")` returns that
single lonely div.

`document` supports both [`querySelector`][dqs] and
[`querySelectorAll`][dqsa], letting you find elements in the entire
document. Elements themselves also support both [`querySelector`][eqs] and
[`querySelectorAll`][eqsa], letting you query elements that are children of a
given element.  `document.querySelector("#my-id").querySelectorAll("img")`
will find images that are children of `#my-id`. In the sample HTML page above,
it will find `<img id="inside">` but not `<img id="outside">`.

With that in mind, what do you think these two lines might do?

```javascript
document.querySelectorAll("#my-id div div");
document.querySelector("#my-id").querySelectorAll("div div");
```

You might reasonably expect them to be the same. After all, one asks `div`
elements inside `div` elements inside `#my-id`, and the other asks for `div`
elements inside `div` elements that are *children* of `#my-id`. I certainly
expected them to be the same. However, when you look at [this JSbin][jsbin]
you'll see that they're not:

```javascript
document.querySelectorAll("#my-id div div").length === 1;
document.querySelector("#my-id").querySelectorAll("div div").length === 3;
```

What the heck is going on here?

It turns out that [`element.querySelectorAll`][eqsa] doesn't start matching
elements from `element`. Instead, it searches the `document` for elements
matching the query, and *then* filters for elements that are children of the
element. So, we're seeing three `div` elements: `div.lonely`, `div.outer`,
`div.inner`; because they match the `div div` selector and because they're all
descendants of `#my-id`.

The trick to remembering this is that CSS selectors for `querySelector(All)`
always start from the document, never from an element.

I think this API is surprising, and the front-end engineers I've asked seem to
agree with me. This is, however, not a bug; it's definitely how the spec
claims it should work, and how it works in Firefox, Chrome and Safari.

[csssel]: https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/Selectors
[dqs]: https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector
[dqsa]: https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll
[eqs]: https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelector
[eqsa]: https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelectorAll
[jsbin]: http://jsbin.com/hineco/edit?html,js,output
[spec]: https://dom.spec.whatwg.org/#dom-parentnode-queryselectorall
