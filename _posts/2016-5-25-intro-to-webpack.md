---
layout: post
title: Введение в Webpack
category: programming
comments: true
---

Хочу рассказать вкратце про такую классную вещь как [Webpack](https://webpack.github.io/ 'Webpack'). Webpack – один из самых мощных и гибких инструментов для сборки frontend. Он позволяет писать модульный JS код прямо из коробки и поддерживает  AMD, CommonJS и даже ES6 модули с использованием Babel. Он умеет загружать любые файлы, делая всяческие манипуляции с ними, дает гибкость в настройках, удобство и хорошую скорость.

Рассмотрим поближе:

Для начало установим `webpack` через `npm`.

```
npm install webpack -g
```

Webpack имеет свой `cli`, и может работать прямо из консоли. Указываем `entry` файл (app.js)  и имя `output` файла (bundle.js)

```
webpack ./app.js ./bundle.js
```

Также можно создать конфигурационный файл и вызывать webpack с настройками указанными в ней. По дефолту webpack смотрит на `webpack.config.js`, но конечно можно поменять название и указать в флаге webpack нужный путь к файлу:

```
webpack —config customWebpackConfig.js
```

Детальнее про доступные флаги можно посмотреть при вызове webpack с флагом `-h` (—help).

Рассмотрим `webpack.config.js`:

```js
module.exports = {
  entry : "./app.js",
  output : {
    filename : "bundle.js"
  }
}
```
Здесь указываем `entry` файл `app.js`, который при выводе - `output` образует `bundle.js`. И если запустить в консоли просто `webpack` то мы получим тот же результат.



Если запустить с флагом `—watch`, webpack будет ждать изменений в файле и срабатывать каждый раз как выявить их. Или добавить ее в `webpack.config.js`:

```js
watch: true
```

## Webpack-dev-server

Если нужно поднять локальный сервер, то webpack имеет свой webpack-dev-server, который ставиться через npm:

```
npm install webpack-dev-server -g
```

и вызвать `webpack-dev-server` который в свою очередь вызовет  webpack и поднимет локальный веб-сервер по умолчанию на 8080 порту:

```
http://localhost:8080/
```
Можно также указать на какую папку поднять веб-сервер с флагом `—content-base`.

```
webpack-dev-server --content-base path/
```

Так же можно настроить output в `webpack.config.js`:

```js
output : {
    path: __dirname + 'build/js',
    publicPath: "/public/js/",
    filename : "bundle.js",
}
```

Это означает, что при сборке `bundle.js` окажется в `build/js/bundle.js`, а когда webpack-dev-server поднимет локальный веб-сервер `bundle.js`, будет доступен по `publicPath` пути, то есть в `index.html`:

```html
<script src="public/js/bundle.js"></script>
```


## Hot Module Replacement

Позволяет обновлять, удалять и добавлять модули в реальном режиме, т.е. даже без перезагрузки страницы (тем самым сохранив её состояние). Для этого нужно добавить флаг `--hot`:

```
webpack-dev-server --hot
```

или опять же ее добавить в `webpack.config.js` как плагин:

```js
plugins: [
  new webpack.HotModuleReplacementPlugin()
]
```

## Loaders

Webpack поддерживает только нативный JS код, и если вам нужно писать на ES6, CoffeeScript, TypeScript и тд. то они могут быть использованы с использованием загрузчиков (loaders).

Рассмотрим пример поддержки `ES6` c использованием `babel-loader`:

```
npm install babel-core babel-loader babel-preset-es2015    --save-dev
```

Также нужно создать `.babelrc` и указать там нужный пресет:

```js
//.babelrc
{
  "presets" : "es2015"
}
```

Теперь можем писать ES6 и транспайлить ее в ES5.

```js
// greeting.js
class Greeting{
  constructor() {
    this.name = "Webpack";
  }

  greet () {
    console.log(`Hi ${this.name}`);
  }
}

export default Greeting;
```

```js
// app.js

import Greeting from './greeting';

let a = new Greeting();

a.greet();

```

Кстати загрузчики можно писать без `-loader`, то есть `babel-loader` можно написать вкратце `babel`.

Добавим поддержку стилей, картинок и шрифтов.

```js
module : {
  loaders : [
    {
      test: /\.js$/,
      exclude: /node_modules/,
      loader: "babel"
    },
    {
      test: /\.css$/,
      loader: 'style!css',
      exclude: '/node_modules/'
    },
    {
      test: /\.scss$/,
      loader: 'style!css!sass?sourceMap',
      exclude: '/node_modules/'
    },
    {
      test: /\.(png|jpg|jpeg|gif|svg|woff|woff2|ttf|eot)$/,
      loader: 'file',
      exclude: '/node_modules/'
    }
  ]
}
```

Здесь важно заметить, что очередь команд в `loader`:

```js
{
  test: /\.scss$/,
  loader: 'style!css!sass?sourceMap',
  exclude: '/node_modules/'
}
```

сначала отработает конвертация `scss` файлов в `css`, который в свою очередь передается `style-loader` который инжектит ее в `index.html`.

## Plugins

Плагины дают возможность еще больше расширить процесс сборки, делая  разные изменения на лету. Для подключения добавим пункт `plugins` с несколькими широко популярными плагинами:

```js
plugins: [
  // Убирает дубликаты кода
  new webpack.optimize.DedupePlugin(),

  // Минифицирует код с помошью UglifyJS
  new webpack.optimize.UglifyJsPlugin(),

  // Позволяет загружать библиотеки с bower
  new BowerWebpackPlugin({
    modulesDirectories: ['bower_components']
  })
]
```

`Webpack.config.js` целиком:

```js
var webpack = require('webpack');

module.exports = {
  entry: './src/app.js',
  output: {
    path: __dirname + '/js/',
    publicPath: '/public/',
    filename: 'bundle.js'
  },
  watch: true,
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel',
        exclude: '/node_modules/'
      },
      {
        test: /\.css$/,
        loader: 'style!css',
        exclude: '/node_modules/'
      },
      {
        test: /\.scss$/,
        loader: 'style!css!sass?sourceMap',
        exclude: '/node_modules/'
      },
      {
        test: /\.(png|jpg|jpeg|gif|svg|woff|woff2|ttf|eot)$/,
        loader: 'file',
        exclude: '/node_modules/'
      }
    ]
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.UglifyJsPlugin(),
    new BowerWebpackPlugin({
      modulesDirectories: ['bower_components']
    })
  ]
}
```

### Ссылки для более глубокого изучения:

* [Документация Webpack](https://webpack.github.io/docs/ 'Документация Webpack')
* [7 бед — один ответ](https://habrahabr.ru/post/245991/ '7 бед — один ответ')
* [Webpack для Single Page App](https://habrahabr.ru/post/265801/ 'Webpack для Single Page App')
* [Скринкаст Webpack](https://learn.javascript.ru/screencast/webpack 'Скринкаст Webpack')
