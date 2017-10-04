title: Aurelia Testing With Jest
date: 2017-05-20
---

Setting up [Jest](https://facebook.github.io/jest/) to test your Aurelia components is pretty easy. The Aurelia team has a great collection of [starter kits](https://github.com/aurelia/skeleton-navigation) that can get you setup using Aurelia. I started with the esnext-webpack version and stripped out some dependencies.

The Aurelia team also has a [aurelia-testing](https://github.com/aurelia/testing) which I haven't used. For this example I only used [Jest](https://facebook.github.io/jest/).

Full source code for this example [aurelia-testing-jest](https://github.com/schempy/aurelia-testing-jest)

Let's start!

### Directory structure
```
.
|__src
|  |__app.js
|  |__app.html
|  |__main.js
|
|__test
|  |__jest-pretest.js
|  |__unit
|     |__app.spec.js
|
|__package.json
```
All tests are in ``test/unit`` directory and end in ``.spec.js``. For this example I'll be testing the ``src/app.js`` component.


### Setup
1. Add a ``jest`` key to ``package.json``. This will be used for configuration.
2. Modify ``.babelrc`` to use ES6 features.
3. Create a setup file for Jest.
4. Create a npm script that will run the tests.

#### Step 1
Adding a [jest configuration](https://facebook.github.io/jest/docs/configuration.html#content) to the ``package.json`` file. If you want to use this method there must be a ``jest`` key in the JSON file.
test
If not you can [set the configuration on the command line](https://facebook.github.io/jest/docs/cli.html#content).
```json
"jest": {
  "modulePaths": [
    "<rootDir>/src",
    "<rootDir>/node_modules"
  ],
  "moduleFileExtensions": [
    "js",
    "json"
  ],
  "transform": {
    "^.+\\.jsx?$": "babel7-jest"
  },
  "testRegex": "\\.spec\\.(ts|js)x?$",
  "setupFiles": [
    "<rootDir>/test/jest-pretest.js"
  ],
  "testEnvironment": "node",
  "moduleNameMapper": {
    "aurelia-(.*)": "<rootDir>/node_modules/$1"
  }
}
```


#### Step 2
Modify ``.babelrc`` to use ES6 features within the test files.
```json
"env": {
  "test": {
    "presets": ["env"],
    "plugins": [
      "transform-class-properties",
      "transform-decorators"
    ]
  }
}
```

#### Step 3
Create a setup file for Jest. This will allow me to use Aurelia in the tests. The setup file is located in ``test/jest-pretest.js``. This file can be named anything you'd like. Configure jest to use this setup file in the configuration under ``setupFiles.js``, line 14 Step 1.
```javascript
import 'aurelia-polyfills';
import {Options} from 'aurelia-loader-nodejs';
import {globalize} from 'aurelia-pal-nodejs';
import * as path from 'path';
Options.relativeToDir = path.join(__dirname, 'unit');
globalize();
```

#### Step 4
Adding a npm script that will run the tests. I added the ``--verbose`` option because I like feedback when running the tests.
```
"scripts": {
  "test": "jest --verbose"
}
```

Now the good stuff!

### The Aurelia component
```javascript
import {inject} from 'aurelia-framework';
import {EventAggregator} from 'aurelia-event-aggregator';

@inject(EventAggregator)
export class App {

  constructor(eventAggregator) {
    this.heading = 'Testing Aurelia With Jest';
    this.ea = eventAggregator;
  }

  fireEvent() {
    this.ea.publish('event-fired');
  }
}
```
The component sets a ``heading`` variable that will be binded to an html element and a ``fireEvent`` function. I wanted to include a dependency to inject into this component so  I can demonstrate how to use mocks.

### The HTML Template
```html
<template>
  <h2>${heading}</h2>
  <form>
    <button
      class="btn btn-default" click.delegate="fireEvent()">
    Fire Event
    </button>
  </form>
</template>
```
The template has a placeholder for the binding ``heading`` defined in the component and a button that will fire off a ``fireEvent`` function also defined in the component.


### The Testing script
```javascript
import {EventAggregator} from 'aurelia-event-aggregator';
jest.mock('aurelia-event-aggregator', () => ({
  EventAggregator: {
    publish: jest.fn()
  }
}));

import {App} from '../../src/app';

describe('App Component', () => {
  let app;

  beforeEach(() => {
    app = new App(EventAggregator);
  });

  test('constructor is defined', () => {
    expect(app.constructor).toBeDefined();
  });

  test('fire event', () => {
    app.fireEvent();

    expect(EventAggregator.publish).toHaveBeenCalledTimes(1);
  })
});
```
There are two tests:
1. Check if a constructor is defined.
2. Make sure the ``EventAggreator.publish`` funtion has been called during the ``fireEvent`` function.

Note there is a ``beforeEach`` function that will create a new component for each test that is run.

The test to check if a constructor is defined is simple. No need for an explaination.

The ``EventAggreator.publish`` test requires some explaination. Notice after I import the ``EventAggregator`` module I create a mock using the ``jest.mock`` syntax. This tells Jest to use the mock anytime ``EventAggregator`` is used. The function that I will mock is ``publish``. The ``jest.fn()`` will keep track of all calls to the ``publish`` function is called along with arguments. You can add as many mock functions as you'd like.

The documentation for [manual mocks](https://facebook.github.io/jest/docs/manual-mocks.html#content) recommends creating a ``__mocks__`` subdirectory adjacent to the module but I prefer to define the mocks within my test file.

Anytime you want to check if a function was called or called with specific parameters that function needs to be a mock. This one got me. I was checking if a function was called but did not mock the iimplementation. Duh!?


### Running the test.
Running ``npm test`` will start running the tests.


Full source code for this example [aurelia-testing-jest](https://github.com/schempy/aurelia-testing-jest)

