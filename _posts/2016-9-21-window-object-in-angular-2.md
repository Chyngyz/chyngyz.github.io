---
layout: post
title: Window object in Angular 2
category: programming
comments: true
---

Example of using window object in Angular 2:

```ts
// window.service.ts

import { OpaqueToken } from '@angular/core';

export const WindowService = new OpaqueToken('WindowService');

```

We use `OpaqueToken` to prevent name collisions in providers, detailed info is [here](http://blog.thoughtram.io/angular/2016/05/23/opaque-tokens-in-angular-2.html).

```ts
// app.module.ts

@NgModule({
  imports: [ ... ],
  declarations: [ ... ],
  providers: [
    { provide: WindowService, useValue: window }
  ],
  bootstrap: [ AppComponent ]
})
export class AppModule { }
```
here we pass the `window` as `useValue`, thus making available global `window` object as service.

As we use string type tokens, we need to use `@Inject` decorator:

```ts
import { Inject } from '@angular/core';

@Component({
  ...
  ...
})
export class MyComponent {
  constructor(@Inject(WindowService) private _window: window) { }
  ...
}
```

[More about @Inject](https://angular.io/docs/ts/latest/guide/dependency-injection.html#!#dependency-injection-tokens).
