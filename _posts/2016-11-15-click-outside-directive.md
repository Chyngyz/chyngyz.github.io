---
layout: post
title: Click outside directive in Angular 2
category: programming
comments: true
---

This is the short post on creating custom directive for triggering event on outside click. It is mostly used in dropdown menus, popups and tooltips. We will be creating an attribute directive that we use like this:

```html
<div class="my-class" (clickOutside)="fireEvent()"></div>
```

As `clickOutside` says, we will use this name as an attribute selector of our directive.

```ts
import { Directive } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  // ...
}
```

Notice that we use square brackets to define attribute directives.

In order to access the current DOM element on which we apply the current directve, we need to inject **ElementRef** in the directive's constructor. (For more detailed info on ElementRef, check the [official docs](https://angular.io/docs/ts/latest/api/core/index/ElementRef-class.html). **Notice the security risk**)

```ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  constructor(private _elementRef : ElementRef) { }
}
```

As our directive will fire an event on click, we need to inject **EventEmitter** and assign an **@Output()** variable as an EventEmitter.

```ts
import { Directive, ElementRef, Output, EventEmitter } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  @Output public clickOutsideEvent = new EventEmitter();
  constructor(private _elementRef : ElementRef) { }
}
```

Notice here that we called our EventEmitter variable **clickOutsideEvent**, which tells that we will be using it in html like this:

```html
<div class="my-class" clickOutside (clickOutsideEvent)="fireEvent()"></div>
```

But if we name it the same as the directive selector, we can achieve the simple syntax removing the extra attribute, and Angular is smart enough to tie the selector and EventEmitter to single one:

```ts
import { Directive, ElementRef, Output, EventEmitter } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  @Output public clickOutside = new EventEmitter();
  constructor(private _elementRef : ElementRef) { }
}
```

As for listening the click events we will be using the **@HostListener** that responds to any given events performed on host element. We pass `document:click` as we want to track all click events on the document and define `['$event.target']` as the second parameter, as we want to pass the event target to the function we call on click.

```ts
import { Directive, ElementRef, Output, EventEmitter, HostListener } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  @Output public clickOutside = new EventEmitter();
  constructor(private _elementRef : ElementRef) { }

  @HostListener('document:click', ['$event.target'])
  public onClick(targetElement) {
      // ...
  }
}
```

And finally we just add the condition that checks if the click event was on the current element or not; and if not, we just fire the function that is bound on our EventEmitter.

```ts
import { Directive, ElementRef, Output, EventEmitter, HostListener } from '@angular/core';

@Directive({
  selector: '[clickOutside]'
})
export class ClickOutsideDirective {
  @Output public clickOutside = new EventEmitter();
  constructor(private _elementRef : ElementRef) { }

  @HostListener('document:click', ['$event.target'])
  public onClick(targetElement) {
    const isClickedInside = this._elementRef.nativeElement.contains(targetElement);
    if (!isClickedInside) {
        this.clickOutside.emit(null);
    }
  }
}
```

That's it.

For more details on creating attribute directive checkout the [official docs](https://angular.io/docs/ts/latest/guide/attribute-directives.html).
