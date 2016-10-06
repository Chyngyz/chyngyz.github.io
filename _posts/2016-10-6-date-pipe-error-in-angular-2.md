---
layout: post
title: Date and Currency pipe error in Angular 2
category: programming
comments: true
---

Recently while working on [Rekrut Project](http://rekrut.kg), I encountered a strange error of my app crashing on some browsers. Digging in the past commits, I finally found that the problem was with my *date pipe* that I used to format my dates.

Digging into the problem, I found that Angular 2 uses the new [Internationalization(Intl)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) standard built into JavaScript and the browser, which allows to automatically display the appropriate currency and date format to users in different geolocations, which is COOL! But the problem is that not all browsers support it. According to [caniuse.com](http://caniuse.com/#search=intl), we see that our app will crash on mobile browsers of
* Android <= 4.3,
* on Safari and Chrome on IOS <= 9.3
* and on IE<=10 and Safari <= 9.1 on desktops.

To fix the compatability of Intl we should provide polyfill to those browsers. One of the easiest ways to do this, is to grab a CDN link from [polyfill.io](https://polyfill.io/v2/docs/) and paste it into our `index.html` like this:

```html
<script src="https://cdn.polyfill.io/v2/polyfill.min.js?features=Intl.~locale.ru"></script>
```

The advantage of using polyfill.io is that it grabs the required polyfill only on those browsers that need it.
But if you don't want to rely on third party CDNs you can include it into your bundle by installing the required polyfill like this:

```bash
npm install intl --save
```

and importing it in your app:

```js
import 'intl';
import 'intl/locale-data/jsonp/ru.js';
```

That's it, now you have full support for all browsers in displaying proper formatted date and currency.

### Update

Just yesterday (05.10.2016) [Webkit announced](https://webkit.org/blog/6978/javascript-internationalization-api/) of their full support of JavaScript Internationalization API.
