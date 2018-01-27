title: Generate unit test boilerplate with platelunch
date: 2018-01-28
---

[Platelunch](https://www.npmjs.com/package/platelunch) is a tool that will generate unit tests based on source files.
I created it because a lot of projects have zero unit tests and this is a quick way to genreate the boilerplate unit
test code.

Currently [jest](https://facebook.github.io/jest/) is the only testing framework that is supported. More to come!

### CLI

Pass in the directory where the source files are located and unit test files will be generated for every '.js' file that's
found.

```js
platelunch --test-framework jest "src/**/*.js"
```

 I'm passing a glob in the example above but you can have it generate only one unit test file.

```js
platelunch --test-framework jest "src/some-file.js"
```

A directory \_\_tests\_\_ will be created at the root of the project that contain all the unit tests. You'll have to modify
the unit test files to include any mocking. Only one test per function or class method is created.

Full source code is available on [github](https://github.com/schempy/platelunch).
