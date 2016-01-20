title: Angular2 With Typescript And Webpack
date: 2016-01-19
---

I'm just starting to get into [angular2](https://www.npmjs.com/package/angular2) with [TypeScript](https://www.npmjs.com/package/typescript). I decided to give the [angular2 quickstart application](https://angular.io/docs/ts/latest/quickstart.html) a try. At first glance I wanted to use [webpack](https://www.npmjs.com/package/webpack) instead of [systemjs](https://www.npmjs.com/package/systemjs) for module loading.

To begin here is my package.json file:
```js
{
  "name": "angular2-typescript-webpack",
  "version": "1.0.0",
  "description": "",
  "scripts": {
    "build": "webpack",
    "start": "webpack-dev-server"
  },
  "license": "ISC",
  "devDependencies": {
    "ts-loader": "^0.7.2",
    "tsd": "^0.6.5",
    "typescript": "^1.7.5",
    "webpack": "^1.12.11",
    "webpack-dev-server": "^1.14.1"
  },
  "dependencies": {
    "angular2": "^2.0.0-beta.1",
    "es6-promise": "^3.0.2",
    "es6-shim": "^0.33.13",
    "reflect-metadata": "^0.1.2",
    "rxjs": "^5.0.0-beta.0",
    "zone.js": "^0.5.10"
  }
}
```

I have [typescript](https://www.npmjs.com/package/typescript) and [tsd](https://www.npmjs.com/package/tsd) installed globally **npm install typescript tsd -g** but included them in the devDependencies section just in case some people don't want to install those globally. You can run **tsd init** then **tsd query angular2 --action install** to have angular2 TypeScript definition files.

I included [webpack](https://www.npmjs.com/package/webpack) and [webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server). There are two [npm scripts](https://docs.npmjs.com/misc/scripts), build and start, that use webpack and webpack-dev-server respectively.

<!-- more -->

 Webpack requires a configuration file named webpack.config.js. The [webpack configuration details](https://webpack.github.io/docs/configuration.html) contain an explaination of all the configuration options. Here is what I included:

```js
var webpack = require("webpack");

module.exports = {
  entry: {
    "vendor": "./app/vendor",
    "app": "./app/boot"
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
    "node_modules"
  ],
  "files": [
    "typings/angular2/angular2.d.ts",
    "typings/tsd.d.ts",
    "app/app.component.ts",
    "app/boot.ts",
    "app/vendor.ts"
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
