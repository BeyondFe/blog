title: Angular2 With Typescript And Webpack
date: 2016-01-19
updated: 2016-09-17
---

I'm just starting to get into [angular2](https://www.npmjs.com/package/angular2) with [TypeScript](https://www.npmjs.com/package/typescript). I decided to give the [angular2 quickstart application](https://angular.io/docs/ts/latest/quickstart.html) a try. At first glance I wanted to use [webpack](https://www.npmjs.com/package/webpack) instead of [systemjs](https://www.npmjs.com/package/systemjs) for module loading.

To begin here is my package.json file:
```json
{
  "name": "angular2-typescript-webpack",
  "version": "1.3.0",
  "scripts": {
    "postinstall": "typings install",
    "build": "webpack",
    "start": "webpack-dev-server"
  },
  "dependencies": {
    "@angular/common": "2.0.0",
    "@angular/compiler": "2.0.0",
    "@angular/core": "2.0.0",
    "@angular/forms": "2.0.0",
    "@angular/http": "2.0.0",
    "@angular/platform-browser": "2.0.0",
    "@angular/platform-browser-dynamic": "2.0.0",
    "@angular/router": "3.0.0",
    "reflect-metadata": "^0.1.3",
    "rxjs": "5.0.0-beta.12",
    "zone.js": "^0.6.23"
  },
  "devDependencies": {
    "ts-loader": "0.8.2",
    "typescript": "1.8.10",
    "typings": "1.3.0",
    "webpack": "1.13.0",
    "webpack-dev-server": "1.14.1"
  }
}
```

I have [typescript](https://www.npmjs.com/package/typescript) and [typings](https://www.npmjs.com/package/typings) installed globally **npm install typescript typings -g** but included them in the devDependencies section just in case some people don't want to install those globally.

You'll need a file named typings.json to install a es6-shim

```json
{
  "globalDependencies": {
    "es6-shim": "github:DefinitelyTyped/DefinitelyTyped/es6-shim/es6-shim.d.ts#7de6c3dd94feaeb21f20054b9f30d5dabc5efabd"
  }
}
```

You can now run **typings install**

I included [webpack](https://www.npmjs.com/package/webpack) and [webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server). There are two [npm scripts](https://docs.npmjs.com/misc/scripts), build and start, that use webpack and webpack-dev-server respectively.

<!-- more -->

 Webpack requires a configuration file named webpack.config.js. The [webpack configuration details](https://webpack.github.io/docs/configuration.html) contain an explaination of all the configuration options. Here is what I included:

```js
var webpack = require("webpack");

module.exports = {
  entry: {
    "vendor": "./app/vendor",
    "app": "./app/main"
  },
  output: {
    path: __dirname,
    filename: "./dist/[name].bundle.js"
  },
  resolve: {
    extensions: ['', '.js', '.ts']
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {
        test: /\.ts/,
        loaders: ['ts-loader'],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"./dist/vendor.bundle.js")
  ]
}
```
I will be splitting [vendor and app code](https://webpack.github.io/docs/code-splitting.html#split-app-and-vendor-code) which will create two bundles. One bundle will contain vendor code like polyfills and other libraries like [lodash](https://github.com/lodash/lodash). The other bundle will contain my application code. Lines 5-6 define the entry point for two bundles that will be created.

Line 26 uses the CommonsChunkPlugin to remove all modules in the vendor bundle from the app bundle.

Line 15 will create source maps. More on this later.

Lines 18-22 will use a [loader](https://webpack.github.io/docs/loaders.html) for my typescript files. The loader will transform the typescript files to ES5 JavaScript. There is a configuration file, tsconfig.json, for the typescript complier that guides the compiler as it generates the JavaScript files.

Here is the tsconfig.json file
```js
{
  "compilerOptions": {
    "target": "ES5",
    "module": "commonjs",
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "removeComments": false,
    "noImplicitAny": false
  },
  "exclude": [
    "node_modules",
    "typings/main",
    "typings/main.d.ts"
  ],
  "compileOnSave": false,
  "buildOnSave": false,
}
```

The [TypeScript Wiki](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) contains all the details for the configuration file.

One of the options you should note is on line 6 that will generate source maps. Ok here we go with the source maps again. One of the neat things with this setup is that [source maps](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/) will be generated. They help when debugging JavaScript with chrome devtools.

After running **npm run build** then **npm run start** goto ```http://localhost:8080``` in a browser (I use chrome). Once the page is loaded open devtools and click on the **Sources** tab. Drill down on the list item labeled **webpack** on the left-side. You'll notice there are the original typescript files! Click on one and check for yourself. You can set breakpoints and start digging in! Here is what your screen should look like:

![alt text](https://raw.githubusercontent.com/schempy/angular2-typescript-webpack/master/images/angular2-typescript-webpack-source-maps.png)

This is a good starting point for me. From here I can start developing an application.

Full source code for this example [angular2-typescript-webpack](https://github.com/schempy/angular2-typescript-webpack)

Thanks to [dachibro](https://github.com/dachibro) and [H-e-r-m](https://github.com/hermanfransen) for their contributions to the github repository!
