> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [piccalil.li](https://piccalil.li/blog/a-modern-css-reset/)

A Modern CSS Reset
==================

_1st of October 2019_

Categories
----------

*   [CSS](/category/css)

I think about and enjoy very boring CSS stuff—probably much more than I should do, to be honest. One thing that I’ve probably spent too much time thinking about over the years, is CSS resets.

In this modern era of web development, we don’t really need a heavy-handed reset, or even a reset at all, because CSS browser compatibility issues are much less likely than they were in the old IE 6 days. That era was when resets such as [normalize.css](https://github.com/necolas/normalize.css/) came about and saved us all heaps of hell. Those days are gone now and we can trust our browsers to behave more, so I think resets like that are probably mostly redundant.

A reset of sensible defaults [permalink](#heading-a-reset-of-sensible-defaults)
-------------------------------------------------------------------------------

I still like to reset stuff, so I’ve been slowly and continually tinkering with a reset myself over the years in an obsessive [code golf](https://en.wikipedia.org/wiki/Code_golf) manner. I’ll explain what’s in there and why, but before I do that, here it is in its entirety:

Code language

CSS

Copy to clipboard

```
/* Box sizing rules */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* Remove default margin */
body,
h1,
h2,
h3,
h4,
p,
figure,
blockquote,
dl,
dd {
  margin: 0;
}

/* Remove list styles on ul, ol elements with a list role, which suggests default styling will be removed */
ul[role='list'],
ol[role='list'] {
  list-style: none;
}

/* Set core root defaults */
html:focus-within {
  scroll-behavior: smooth;
}

/* Set core body defaults */
body {
  min-height: 100vh;
  text-rendering: optimizeSpeed;
  line-height: 1.5;
}

/* A elements that don't have a class get default styles */
a:not([class]) {
  text-decoration-skip-ink: auto;
}

/* Make images easier to work with */
img,
picture {
  max-width: 100%;
  display: block;
}

/* Inherit fonts for inputs and buttons */
input,
button,
textarea,
select {
  font: inherit;
}

/* Remove all animations, transitions and smooth scroll for people that prefer not to see them */
@media (prefers-reduced-motion: reduce) {
  html:focus-within {
   scroll-behavior: auto;
  }
  
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}


```

### Breaking it down

We start with box-sizing. I just flat out reset all elements and pseudo-elements to use `box-sizing: border-box`.

Code language

CSS

Copy to clipboard

```
*,
*::before,
*::after {
  box-sizing: border-box;
}


```

Some people think that pseudo-elements should `inherit` box sizing, but I think that’s silly. If you want to use a [different box-sizing value](https://css-tricks.com/almanac/properties/b/box-sizing/), set it explicitly—at least that’s what I do, anyway. I wrote about box-sizing more over on [CSS From Scratch](https://cssfromscratch.com/posts/bite-sized-basics-box-sizing/).

Code language

CSS

Copy to clipboard

```
/* Remove default margin */
body,
h1,
h2,
h3,
h4,
p,
li,
figure,
figcaption,
blockquote,
dl,
dd {
  margin: 0;
}


```

After box-sizing, I do a blanket reset of `margin`, where it gets set by the browser styles. This is all pretty self-explanatory, so I won’t get into it too much.

Code language

CSS

Copy to clipboard

```
html:focus-within {
  scroll-behavior: smooth;
}


```

The “resetting” is now mostly done, so the first thing I do for core styles is set smooth scrolling. This was previously set right on the `html` element, but [recent updates](https://css-tricks.com/fixing-smooth-scrolling-with-find-on-page/) have resulted in this being updated to only apply smooth scrolling when there is `:focus-within` the `html` element.

FYI

FYI

FYI

FYI

I like that this has been updated recently. I never even considered how searching in-page could become so problematic ([you should definitely read the post](https://css-tricks.com/fixing-smooth-scrolling-with-find-on-page/)). Big thanks to [David Darnes](https://twitter.com/DavidDarnes) for [doing the leg-work on a PR, too](https://github.com/hankchizljaw/modern-css-reset/pull/36).

Code language

CSS

Copy to clipboard

```
body {
  min-height: 100vh;
  text-rendering: optimizeSpeed;
  line-height: 1.5;
}


```

Next up: body styles. I keep this really simple. It’s useful for the `<body>` to fill the viewport, even when empty, so I do that by setting the `min-height` to `100vh`.

I only set two text styles. I set the `line-height` to be `1.5` because the default `1.2` just isn’t big enough to have accessible, readable text. I also set `text-rendering` to `optimizeSpeed`. Using `optimizeLegibility` makes your text look nicer, but can have serious performance issues such as [30 second loading delays](https://marco.org/2012/11/15/text-rendering-optimize-legibility), so I try to avoid that now. I do sometimes add it to sections of microcopy though.

Code language

CSS

Copy to clipboard

```
ul[role='list'],
ol[role='list'] {
  list-style: none;
}


```

I only reset `list-style` where a list element has a `role=["list"]` attribute. This assists with some [accessibility issues, expertly explained by Scott](https://www.scottohara.me/blog/2019/01/12/lists-and-safari.html).

Code language

CSS

Copy to clipboard

```
a:not([class]) {
  text-decoration-skip-ink: auto;
}


```

For links without a class attribute, I set `text-decoration-skip-ink: auto` so that the underline renders in a much more readable fashion. This could be set on links globally, but it’s caused one or two conflicts in the past for me, so I keep it like this.

Code language

CSS

Copy to clipboard

```
img {
  max-width: 100%;
  display: block;
}


```

Good ol’ fluid image styles come next. I set images to be a block element because frankly, life is too short for that weird gap you get at the bottom, and realistically, images—especially with work I do—tend to behave like blocks.

Code language

CSS

Copy to clipboard

```
input,
button,
textarea,
select {
  font: inherit;
}


```

Another thing I’ve finally been brave enough to set as default is `font: inherit` on input elements, which as a shorthand, [does exactly what it says on the tin](https://css-tricks.com/almanac/properties/f/font/). No more tiny (mono, in some cases) text!

Code language

CSS

Copy to clipboard

```
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}


```

Last, and by no means least, is a single `@media` query that resets animations, transitions and scroll behaviour if the [user prefers reduced motion](https://css-tricks.com/introduction-reduced-motion-media-query/). I like this in the reset, with [specificity trumping](https://hankchizljaw.com/wrote/css-specificity-and-the-cascade/) `!important` selectors, because most likely now, if a user doesn’t want motion, they won’t get it, regardless of the CSS that follows this reset.

ℹ️ **Update**: _Thanks to [@atomiks](https://github.com/atomiks), this has [been updated](https://github.com/hankchizljaw/modern-css-reset/pull/6) so it doesn’t break JavaScript events watching for `animationend` and `transitionend`._

**Updated: November 6th, 2020 with list rules.**

Wrapping up [permalink](#heading-wrapping-up)
---------------------------------------------

That’s it, a very tiny reset that makes my life a lot easier. If you like it, you can use it yourself, too! You can find it on [GitHub](https://github.com/hankchizljaw/modern-css-reset) or [NPM](https://www.npmjs.com/package/modern-css-reset).

* * *

🇯🇵 If you would prefer to read this article in Japanese, head [over here](https://coliss.com/articles/build-websites/operation/css/a-modern-css-reset.html), where [コリス](https://twitter.com/colisscom) has very kindly translated it for me.

🇷🇺 If you would prefer to read this article in Russian, head [over here](https://medium.com/@stasonmars/%D1%81%D0%BE%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B8%CC%86-%D1%81%D0%B1%D1%80%D0%BE%D1%81-css-f5816963c82b), where [Stas](https://twitter.com/stassonmars) has very kindly translated it for me.

🇪🇸 If you would prefer to read this article in Spanish, head [over here](https://super-simple.net/blog/un-css-reset-moderno/), where [superSimple](https://twitter.com/supersimplenet) has very kindly translated it for me.

 [![](https://assets.codepen.io/174183/ko-fi.jpg?width=1077&height=327&format=auto) Enjoying this content? It’d be really appreciated if you support me on Ko-Fi by buying me a coffee.](https://ko-fi.com/Y8Y54XOHL)