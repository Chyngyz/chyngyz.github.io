---
layout: post
title: How to SVG Sprite
category: programming
comments: true
---

SVG nowadays is very popular, and there great resources out there on how to generate SVG sprites and use them. I put the links to them below the post. Here I just want to show how I use SVG sprites.

So first of all, I need to generate SVG sprite. Here is the sprite container.

```html
<svg style="display:none;">
  <symbol id="twitter-icon" viewBox="0 0 32 32">
    <path d="M32 6.076c-1.177 …" fill="#000000"></path>
  </symbol>

 <!-- remaining icons here -->
  <symbol id="facebook-icon" viewBox="0 0 32 32">
   <!-- icon contents -->
   <path d="M16 0q6.6 0 11.3 4 …" fill="#000000"></path>
  </symbol>

  <!-- etc. -->
</svg>
```

I make sure that all SVG elements' viewports are same so that there is no viewport differences between them, otherwise you have a headache fixing via CSS. So make sure all SVG element paths are taken from same viewport dimensions. The `fill` attribute is the default color, and maybe omitted if needed. The color can be applied in CSS.

The next step is to link to particular SVG icon to view it:

```html
<svg class="twitter-icon">
  <use xlink:href="#twitter-icon"></use>
<svg>
```
Here the `#twitter-icon` is the reference to the particular id of `symbol` in SVG sprite above.

For small number of SVG elements I prefer to put the SVG sprite inline in the document. But if it is large enough containing dozens of SVG icons, it is better to take SVG sprite into external file and reference to individual icon like this:

```html
<svg class="twitter-icon">
  <use xlink:href="path/to/icons.svg#twitter-icon"></use>
<svg>
```

The best part is that class of containing SVG icon can be easily styled, for instance adding some hover effects:

```css
.twitter-icon {
  display: inline-block;
  width: 2em;
  height: 2em;
  fill: #ccc;
  transition: all .2s cubic-bezier(.4,0,.2,1);
}

.twitter-icon:hover {
  fill: #55acee;
}
```

The disadvantage of SVG sprite is that it is not supported in **IE**. Even IE9+ don't fully support this technique. No version of IE supports referencing an external SVG in `<use>`.

But fortunately there is a plugin called [svg4everybody](https://github.com/jonathantneal/svg4everybody 'svg4everybody') that fills this gap in IE. For details, refer to the plugin’s Github repository.

So this is it, the live examples of SVG sprite use are social icons on the left. Checkout some great resources from the links below:

* [Icon System with SVG Sprites](https://css-tricks.com/svg-sprites-use-better-icon-fonts/ 'svg4evIcon System with SVG Spriteserybody')
* [An Overview of SVG Sprite Creation Techniques](http://24ways.org/2014/an-overview-of-svg-sprite-creation-techniques/ 'An Overview of SVG Sprite Creation Techniques')
* [CSS-Tricks Screencast #136: SVG is for Everybody](http://www.youtube.com/watch?v=w83XRCkMtHQ 'CSS-Tricks Screencast #136: SVG is for Everybody')
