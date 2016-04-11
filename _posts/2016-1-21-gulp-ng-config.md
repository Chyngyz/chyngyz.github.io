---
layout: post
title: Using env variables in AngularJS app using gulp-ng-config
category: programming
comments: false
---

While building an AngularJS app, I encountered that I need my angular constants to be changed according to my development environment. As my app is scaffolded by using [yeoman](http://yeoman.io/ 'yeoman'), the [generator-gulp-angular](https://github.com/Swiip/generator-gulp-angular "generator-gulp-angular") to be precise, I had to find the gulp module that could create the constants module according to my gulp processes. Luckily I found several npm packages that solve my issue, but the one I used was [gulp-ng-config](https://github.com/ajwhite/gulp-ng-config 'gulp-ng-config').

All I did was to create `config.json` with the proper constants:

```json
{
  "local": {
    "EnvironmentConfig": {
      "API": "http://localhost:8888/api"
    }
  },
  "production": {
    "EnvironmentConfig": {
      "API": "http://myproduction.com/api"
    }
  }
}
```
and the gulp task that converted the `config.json` into `config.js` angular module, that I inject into my app later on.
The gulp tasks are:

```js
'use strict';

var path = require('path'),
    gulp = require('gulp'),
    conf = require('./conf'),
    gulpNgConfig = require('gulp-ng-config');

gulp.task('config', function () {
  gulp.src(path.join(conf.paths.src, '/app/config.json'))
    .pipe(gulpNgConfig('vip.config', {
      environment: 'local'
    }))
    .pipe(gulp.dest(path.join(conf.paths.src, '/app/')))
});

gulp.task('config:build', function () {
  gulp.src(path.join(conf.paths.src, '/app/config.json'))
    .pipe(gulpNgConfig('vip.config', {
      environment: 'production'
    }))
    .pipe(gulp.dest(path.join(conf.paths.src, '/app/')))
});
```

I added the above tasks before the certain task fires:

```js
'use strict';

gulp.task('serve', ['config','watch'], function () {
  ...
});

gulp.task('serve:dist', ['config:build','build'], function () {
  ...
});

gulp.task('serve:e2e', ['config','inject'], function () {
  ...
});

gulp.task('serve:e2e-dist', ['config:build','build'], function () {
  ...
});

gulp.task('build', ['config:build', 'html', 'fonts', 'other']);
```

And the results for serving and checking locally, the `config` task gave me

```js
angular.module('vip.config', [])
.constant('EnvironmentConfig', {
  "API":"http://localhost:8888/api"
});
```

while the build and serving dist gave

```js
angular.module('vip.config', [])
.constant('EnvironmentConfig', {
  "API":"http://myproduction.com/api"
});
```

There are quite a lot of options in the package, like wrapping the module into IIFE, or choosing the service whether it is `constant` or `value` and many others. Check them out at [https://github.com/ajwhite/gulp-ng-config](https://github.com/ajwhite/gulp-ng-config 'https://github.com/ajwhite/gulp-ng-config');
