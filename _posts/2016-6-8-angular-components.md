---
layout: post
title: Компоненты в Ангуляре 1.5^
category: programming
comments: true
---

С новой версии ангуляра 1.5^, появился новый метод `.component()`, который позволяет писать компоненты похожие на компоненты в ангуляре 2, что в дальнейшем позволит легче перейти на ангуляр 2.

Компоненты в ангуляре 1.5^ очень похожи на директивы(тоже компоненты по сути), но есть отличия.

Во первых если сравнить вызов метода с директивами:

```js
.directive(name, fn)

.component(name, options)
```

`name` - имя компонента и соответсвенно является именем элемента для вызова, вторым параметром является `options` (опции) в виде объекта. Здесь важно заметить, что компоненты могут являться только элементами при вызове, тогда как директива имеет гибкость быть элементом, атрибутом или классом.

## `scope` и `bindToController` в `bindings`

В компонентах область скоупа изолированная, а поток данных для связывания с котроллером проходит через новое свойство `bindings`, в директивах данные проходят через `bindToController`:

```js
.directive('myDirective', function(){
    return {
      scope: {},
      bindToController: {
           data: '='
      }
    }
})

.component('myComponent', {
  bindings: {
    data: '='
  }
})

```

## $ctrl

Так же в компонентах появилось имя по умолчанию `$ctrl` на свойство `controllerAs` для вызова в шаблонах.

```js
.component('myComponent', {
  bindings: {
    data: '='
  },
  template: "<p ng-bind='$ctrl.data.name'></p>"
})
```

но опять же можно изменить имя указав значение `controllerAs`:

```js
.component('myComponent', {
  bindings: {
    data: '='
  },
  controllerAs: 'vm',
  template: "<p ng-bind='vm.data.name'></p>"
})
```

Так же в ангуляре 1.5 можно писать имя контроллера (если используется контроллер) в директивах и в компонентах так:

```js
controller: "myCtrl as vm"

```

что убирает надобность в использовании `controllerAs`.

## Stateless компоненты

Если заметили в примере выше мы использовали в компоненте только `bindings` и `template`, не делая никаких манипуляций с данными. Такие компоненты называется stateless компоненты. Такие компоненты используются для передачи данных в шаблоны, что позволяет строить полноценное компонентно-ориентированное приложение.

```js
.component('myComponent', {
  bindings: {
    data: '='
  },
  template: [
    '<div class="container">',
      '<h1 ng-bind="$ctrl.data.title"></h1>',
      '<ul>',
        '<li ng-repeat="item in $ctrl.data.list track by $id(item)" ng-bind="item"></li>',
      '</ul>',
    '</div>'
  ].join('')
})
```

Так же в ангуляра 1.5^ в компонентах так и в директивах, свойство `template` может являться функцией которая принимает `$element` и `$attrs` параметры и возвращает обязательно шаблон в виде `string`:

```js
.component('myComponent', {
  template: function($element, $attrs) {
    // мелкие DOM манипуляции с использованием $element и $attrs

    return [
      '<div class="container">',
        '<h1 ng-bind="$ctrl.title"></h1>',
      '</div>'
    ].join('');
  }
})
```

## Когда не стоит использовать компоненты:

* стоит не использовать компоненты если нужно большие DOM манипуляции, использование обработчиков событий, у компонентов нет `link`, поэтому стоит использовать директивы
* если нужно использовать классы или атрибуты

## Соурс код на компоненты ангуляра 1.5.6 версии:

```js
this.component = function registerComponent(name, options) {
  var controller = options.controller || function() {};

  function factory($injector) {
    function makeInjectable(fn) {
      if (isFunction(fn) || isArray(fn)) {
        return function(tElement, tAttrs) {
          return $injector.invoke(fn, this, {$element: tElement, $attrs: tAttrs});
        };
      } else {
        return fn;
      }
    }

    var template = (!options.template && !options.templateUrl ? '' : options.template);
    var ddo = {
      controller: controller,
      controllerAs: identifierForController(options.controller) || options.controllerAs || '$ctrl',
      template: makeInjectable(template),
      templateUrl: makeInjectable(options.templateUrl),
      transclude: options.transclude,
      scope: {},
      bindToController: options.bindings || {},
      restrict: 'E',
      require: options.require
    };

    // Copy annotations (starting with $) over to the DDO
    forEach(options, function(val, key) {
      if (key.charAt(0) === '$') ddo[key] = val;
    });

    return ddo;
  }

  // Copy any annotation properties (starting with $) over
  // to the factory and controller constructor functions
  // These could be used by libraries such as the new component router
  forEach(options, function(val, key) {
    if (key.charAt(0) === '$') {
      factory[key] = val;
      if (isFunction(controller)) controller[key] = val;
    }
  });

  factory.$inject = ['$injector'];

  return this.directive(name, factory);
};
```
