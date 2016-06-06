---
layout: post
title: Angular + Webpack
category: programming
comments: true
---

Learning [Webpack](https://webpack.github.io/) lead me in writing my own starter boilerplate for angular 1 projects. I understand that there are too many of this kind of starter projects, but writing it, I gained a lot. Especially in depth configuration of Webpack both for devepelopment and production.

The [Angular-Webpack-Starter](https://github.com/Chyngyz/angular-webpack-starter) consists of:

* ES6 Support using Babel
* Dev-server with live-reload
* Production optimization
* Serve Production
* Inject dependencies using [ng-annotate](https://github.com/huston007/ng-annotate-loader)
* File Structure according best practices and angular style guide
* Flexible configuration of index.html by using EJS(LoDash/Underscore Templates)
* Separate files for dev and production Webpack Configs.
* Third party libs included: UI-Router, jQuery, MomentJS, Bootstrap, Ionicons.


### Quick start

```bash
# clone our repo
$ git clone https://github.com/Chyngyz/angular-webpack-starter my-app

# change directory to your app
$ cd my-app

# install the dependencies with npm
$ npm install

# start the server
$ npm start

# build into dist folder
$ npm run build

# Serve dist
$ npm run serve:dist
```

go to [http://localhost:3000](http://localhost:3000) in your browser.

### Webpack.config.js

This is config file that runs on `npm start` command:

```js
'use strict';

// Dev server port
var DEV_SERVER_PORT = 3000;

var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  context: __dirname + '/src',
  entry: './app/app.js',
  output: {
    path: __dirname + '/dist',
    publicPath: 'http://localhost:' + DEV_SERVER_PORT + '/',
    filename: 'bundle.js'
  },
  devtool: 'eval',
  devServer: {
    contentBase: './src',
    port: DEV_SERVER_PORT,
    historyApiFallback: {
      index: 'index.html'
    }
  },
  module: {
    preLoaders: [],
    loaders: [
      {
        // Inject dependencies, convert ES6 to ES5 by using Babel
        test: /\.js$/,
        loader: 'ng-annotate?add=true&map=false!babel',
        exclude: '/node_modules/'
      },
      {
        // Row loader to load html as inline templates.
        test: /\.html$/,
        loader: 'raw',
        exclude: '/node_modules/'
      },
      {
        // Load css and inject in index.html in head
        test: /\.css$/,
        loader: 'style!css',
        exclude: '/node_modules/'
      },
      {
        // Convert scss to css and inject in index.html in head
        test: /\.scss$/,
        loader: 'style!css?sourceMap!sass?sourceMap',
        exclude: '/node_modules/'
      },
      {
        // Load images
        test: /\.(png|jpg|jpeg|gif|ico)$/,
        loader: 'file'
      },
      {
        // Load fonts, if file size is less than 10KB,
        // it will be converted and loaded as DataURI
        test: /\.((woff2?|svg)(\?v=[0-9]\.[0-9]\.[0-9]))|(woff2?|svg)$/,
        loader: "url?limit=10000"
      },
      {
        // Load fonts as file
        test: /\.((ttf|eot)(\?v=[0-9]\.[0-9]\.[0-9]))|(ttf|eot)$/,  
        loader: "file"
      }
    ]
  },
  plugins: [
    // Hot Module for live-reload
    new webpack.HotModuleReplacementPlugin(),
    // Define global PRODUCTION CONST that can be used anywhere in the app
    new webpack.DefinePlugin({
      PRODUCTION: false
    }),
    // Define third party libs
    new webpack.ProvidePlugin({
      $: 'jquery',
      moment: 'moment'
    }),
    // Configure index.ejs for dev mode
    new HtmlWebpackPlugin({
      ngApp: 'app',
      title: 'Angular + Webpack',
      mobile: true,
      template: './index.ejs',
      favicon: 'favicon.ico',
      baseHref: '/',
      inject: false,
      devServer: 'http://localhost:' + DEV_SERVER_PORT
    })
  ]
};
```


### Webpack.prod.config.js

This is config file that runs on `npm run build` command:

```js
'use strict';

var webpack = require('webpack');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  context: __dirname + '/src',
  entry: './app/app.js',
  output: {
    path: __dirname + '/dist',
    filename: 'assets/js/bundle.js'
  },
  devtool: 'cheap-module-source-map',
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'ng-annotate?add=true&map=false!babel',
        exclude: '/node_modules/'
      },
      {
        test: /\.html$/,
        loader: 'raw',
        exclude: '/node_modules/'
      },
      {
        // Take out css into file
        test: /\.scss$/,
        loader: ExtractTextPlugin.extract(
          'style',
          'css?sourceMap!resolve-url!sass?sourceMap=true&sourceMapContents=true',
          {
            publicPath: '../../'
          }
        ),
        exclude: '/node_modules/'
      },
      {
        test: /\.(png|jpg|jpeg|gif|ico)$/,
        loader: 'file?name=assets/images/[hash].[ext]'
      },
      {
        test: /\.((woff2?|svg)(\?v=[0-9]\.[0-9]\.[0-9]))|(woff2?|svg)$/,
        loader: "url?limit=10000&name=assets/fonts/[name].[hash].[ext]"
      },
      {
        test: /\.((ttf|eot)(\?v=[0-9]\.[0-9]\.[0-9]))|(ttf|eot)$/,  
        loader: "file?name=assets/fonts/[name].[hash].[ext]"
      }
    ]
  },
  plugins: [
    // Define PRODUCTION CONST
    new webpack.DefinePlugin({
      PRODUCTION: true
    }),
    // Third party libs
    new webpack.ProvidePlugin({
      $: 'jquery',
      moment: 'moment'
    }),
    // Don't emit error in assets
    new webpack.NoErrorsPlugin(),
    // Avoids duplication of code chunks
    new webpack.optimize.DedupePlugin(),
    // UglifyJS
    new webpack.optimize.UglifyJsPlugin(),
    // Configure index.ejs according to production
    new HtmlWebpackPlugin({
      ngApp: 'app',
      title: 'Angular + Webpack',
      template: './index.ejs',
      inject: false,
      favicon: 'favicon.ico',
      mobile: true,
      baseHref: '/',
      minify: {
        collapseWhitespace: true,
        conservativeCollapse: true,
        minifyCSS: true,
        minifyJS: true
      },
      googleAnalytics: {
        trackingId: 'UA-XXXX-XX',
        pageViewOnLoad: true
      },
    }),
    // Take out css into [name].[hash].css
    new ExtractTextPlugin('assets/styles/[name].[hash].css'),
    // Copy static assets
    new CopyWebpackPlugin([{
      from: __dirname + '/src/assets',
      to: __dirname + '/dist/assets'
    }, {
      from: __dirname + '/src/favicon.ico',
      to: __dirname + 'dist/favicon.ico'
    }], {
      ignore: ['*.scss','fonts/**/*']
    })

  ]
};
```

### To Do

* Add tests support
