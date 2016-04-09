title: Default export value in Babel 6.x
date: 2016-04-08
---

In ECMAScript 6 creating a module whose default export is a class would look like the following: 

```js
export default class MyClass {
	getMessage() {
		return 'hello';
	}
}
```

Using Babel 6.x to transpile this would look like the following:

```js
'use strict';

Object.defineProperty(exports, "__esModule", {
	value: true
});

// More transpilation stuff goes here.

// This is the important line!
exports.default = MyClass;
```

<!-- more -->

If I wanted to require and use this module, in node, I would do the following:
```js
var MyClass = require('./MyClass');

var myClass = new MyClass();
console.log(myClass.getMessage()); // Should print out `hello`
```

Running that would produce an error:
```js
TypeError: MyClass is not a function
```
 This stummped me for a bit until I viewed the transpiled file and noticed the **exports** statement.
```js
exports.default = MyClass;
```

In order to require and use this module, in node, correctly I would do:
```js
var MyClass = require('./MyClass');

var myClass = new MyClass.default();
console.log(myClass.getMessage()); // Will print out `hello`
```






