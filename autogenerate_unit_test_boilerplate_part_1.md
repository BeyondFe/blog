title: Autogenerate unit test boilerplate. Part 1.
date: 2017-11-07
---

Most javascript projects I work on don't have any unit tests. I only started adding unit tests to my workflow in the past year but have found them extremely useful. Adding unit tests to an existing project can be a pain. Especially some of the boilerplate code for each unit test. I wanted a tool that would perform four tasks:

1. Parse the javascript source code using [babylon](https://github.com/babel/babylon) to create an [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) or AST. Use [AST Explorer](http://astexplorer.net) for a nice visual of what an AST looks like.
2. Traverse the AST, using [babel-traverse](https://github.com/babel/babel/tree/master/packages/babel-traverse), and look for functions that would require a test.
3. Create a model that stores information on each function that will be tested. This information would include the function name, parameters, any calls to other functions and if it returns a value.
4. Use the model to genreate a new AST that would be the unit test. I'll be using [babel-generator](https://github.com/babel/babel/tree/master/packages/babel-generator).

I only wanted to genreate most of the boilerplate code that is needed for a unit test. Things like importing or requiring modules, setting up any mocks, and creating describe/test blocks for any functions.

The test won't include any expectations. Stuff like this **expect(true).toBe(true)**. I'll leave that for the developer. It can get challenging to understand the purpose of a function by analyizing the AST. I also didn't want this tool to remove the developer from writing a unit test.

<!-- more -->

### Prerequisite
[Babel Handbook](https://github.com/thejameskyle/babel-handbook/blob/master/translations/en/plugin-handbook.md) is great place to get an idea of the all the babel tools I'll be using. It also gives an introduction to the AST and traversing.

### Create Model
Let's create a model to store all the functions and exports that we'll need to create the boilerplate unit test. I'll be referencing this model in code below after traversing the AST.
```javascript
function Model() {
    this.functions = [];
    this.moduleExports = [];
}

Model.prototype.addFunction = function(func) {
    this.functions = [
        ...this.functions,
        func
    ];
};

Model.prototype.addModuleExports = function(moduleExports) {
    this.moduleExports = moduleExports;
};

Model.prototype.getFunctions = function() {
    return this.functions;
};

Model.prototype.getModuleExports = function() {
    return this.moduleExports;
};

module.exports = Model;
```
### Parse source code
To begin I want to parse the source code to create an AST. The variable **ast** stores the parsed code.
```javascript
  const babylon = require("babylon");

  const opts = {
    sourceType: "module"
  };

  const code = `
    function sayHello(name) {
        const greeting = "Hello ";

        return greeting + name;
    }

    module.exports = sayHello;
  `;

  const ast = babylon.parse(code, opts);
```
### Traverse the ast
```javascript
    const traverse = require("babel-traverse").default;
    const t = require("babel-types");
    const Model = require("./src/model");
    const model = new Model();

    let functions = [];
    let moduleExports = null;

    // Traverse the AST.
    traverse(ast, {

        // Find all function declarations.
        FunctionDeclaration(path) {
            functions.push(path);
        },

        // Find module.exports.
        MemberExpression(path) {
            // Check if the object is the type Identifier with
            // the name 'module'.
            if(t.isIdentifier(path.node.object, { name: "module" }) {
                moduleExports = path.parent;
            }
        }
    });

    // Add module.exports to the Model.
    // The function getModuleExportsDetails is defined in the next section.
    const moduleExportDetails = getModuleExportsDetails(moduleExports);
    model.addModuleExports(moduleExportDetails);

    // Loop over functions and add to the Model only if the
    // function is exported.
    functions.forEach((func) => {
        // The function getFunctionDetails is defined in the next section.
        const functionDetails = getFunctionDetails(func);

        const isExported = model
            .getModuleExports()
            .some((moduleExport) => {
                return moduleExport === functionDetails.name;
            });

        // Only add the function if it's exported.
        if(isExported) {
            model.addFunction(functionDetails);
        }
    });
```
I'm only interested in **FunctionDeclarations** and **MemberExpressions**.

Line 4 creates a new model to add functions and exports.

Every time a FunctionDeclaration is located I'll add it to an array. The path parameter refers to the FunctionDeclaration AST that was located during the traversal. Lines 12 - 15.

MemberExpressions will contain the **module.exports**. I'll look for MemberExpressions that contain an object with the name module. Once that is located I'll add the parent which will be an AssignmentExpression. I need the parent in order know what is being exported. I only want to setup tests for what is exported. This can get confusing. Checkout the snippet saved with this example at [AST Explorer](https://astexplorer.net/#/gist/23a0c5dab3ddaf35c2e553c01d632a9d/9fd263f3a6d292745fcd564eebfee2d58cb7384f). It will help getting a visual. Lines 17 -24.

The MemberExpression section will use [babel-types](https://github.com/babel/babel/tree/master/packages/babel-types) to check types of AST nodes. Line 21.

Once the traversal is completed I'll add the exports to the model, iterate over the functions array and only add the functions, to the model, that are exported. Lines 27 - 48.




The function below will get the details from a FunctionDeclaration. I added another traversal to find any calls to other functions and if it returns anything. A note on finding calls to other functions. I'm adding the function name that is being called. For simplicity I'm assuming the call will  be **someFunction()**. Of course there can be calls like **someLibrary.someFunction()** but I'll cover that another time.
```javascript
function getFunctionDetails(ast) {
    let returns = false;
    let mocks = [];

    // Function name.
    const name = ast.node.id.name;

    // Function parameters.
    const params = ast.node.params
        .reduce((acc, param) => {
            acc.push(param.name);
        }, []);

    // Traverse the FunctionDeclaration AST for
    // calls to to other functions and if the function
    // returns a value.
    path.traverse({
        CallExpression: (callExpPath) => {
            mocks.push({
                name: callExpPath.node.callee.name
            });
        },

        ReturnStatement: () => {
            returns = true;
        }
    });

    return {
        name: name,
        params: params,
        mocks: mocks,
        returns: returns
    };
}
```

The function below will get the details from **module.exports**. I'm not handling any named export declarations, **export const foo = "bar"**, or default export declarations, **export default function() {}**, just yet :)

```javascript
function getModuleExportsDetails(node) {
    let moduleExports = [];

    // Checks if the module.exports is defined like this:
    // module.exports = sayHello
    if(t.isIdentifier(node.right)) {
        moduleExports.push(node.right.name);

    // Checks if the module.exports is defined liked this:
    // module.exports = { sayHello: sayHello }
    } else if(t.isObjectExpression(node.right)) {
        node.right.properties.forEach((property) => {
            moduleExports.push(property.key.name);
        });
    }

    return moduleExports;
}
```

Now I have a model that contains what I need to create a boilerplate unit test. This model will be used to create an AST that will be the test file. The model will look like the following:

```javascript
    functions: [
        {
            name: "sayHello",
            params: [ "name" ],
            mocks: [],
            returns: true
        }
    ],
    moduleExports: [ "sayHello" ]
```

This is a simple example. Besides **FunctionDeclarations** there can be **FunctionExpressions**, **ClassMethods**, **ImportDeclarations** etc... The list goes on and on.

[Part 2](https://schempy.com/2017/11/07/autogenerate_unit_test_boilerplate_part_2/) will handle creating an AST from the model that was generated. The AST will be used to generate unit test code.

### Source code
Source code for this example is at [github](https://github.com/schempy/autogenerate-unit-test-boilerplate-files/tree/master/part_01).