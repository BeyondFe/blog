title: Autogenerate unit test boilerplate. Part 2.
date: 2017-11-07
---

[Part 1](https://schempy.com/2017/11/07/autogenerate_unit_test_boilerplate_part_1/) deals with parsing a javascript file to get an AST, looking for **FunctionDeclarations**, **MemberExpressions** and generating a model. The model will be used to generate a new AST that will represent the boilerplate code for a unit test. This article will deal with generating the boilerplate unit test AST and code.

The source code I want to generate a unit test will be the following:
```javascript
function sayHello(name) {
    const greeting = "Hello ";

    return greeting + name;
}

module.exports = sayHello;
```

<!-- more -->

The model that is generated from the source code will look like the following:

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

I'll be using two modules to create the unit test AST and code:

1. [bable-types](https://github.com/babel/babel/tree/master/packages/babel-types) used to create AST nodes.
2. [babel-generator](https://github.com/babel/babel/tree/master/packages/babel-generator) used to generate code from the AST.

I want to create a test block for each function in the model. From the source code above I want the test to look like the following:

```javascript
describe("Testing", () => {
    test("testing sayHello", () => {
        const name = null;
        const result = sayHello(name);
    });
});
```

Note line 3 there's a variable for the function parameter **name**. I do this so the developer can assign values to each function parameter. Every function parameter will have a variable defined this way.


### Iterating over functions in model
First I have to iterate over all the functions in the model. Then I can create a test block for each function. The code below will create an array of test blocks. A test block will be for every function in the model. I'm using the [reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce?v=a) method to iterate over the functions. Inside the reduce method I will

1. Create an array of function parameters.
2. Create an array of variables There will be one variable for every function parameter.
3. Create the function call.
4. Create the test block.

Once iterating is completed I'll add the test block array to a description block.
```javascript
const testBlockArray = model
    .getFunctions()
    .reduce((acc, functionDetails) {
        // Create tests here...
        // 1. Create function parameters.
        // 2. Create a variable for every function paramter.
        // 3. Create function call.
        // 4. Create test block.

        // Add test to the accumulator

        // return the accumulator
        return acc;
    }, []);

// Add test block array to the descrition block...
```

### Creating function parameters
All function parameters will be defined as an [Identifier](https://github.com/babel/babel/tree/master/packages/babel-types#identifier). Below I'm creating an array of function parameters that's iterating over the params array from the model.

```javascript
const functionParameters = functionDetails.params
    .reduce((acc, param) => {
        acc.push(t.identifier(param));

        return acc;
    }, []);
```

### Creating variables for every function parameter
Every function parameter will have a variable defined within the test block. The variable will have the same name as the parameter and assigned a null value. This will also loop over the params array from the model. Below I'm creating an array of variables.
```javascript
const variables = functionDetails.params
    .reduce((acc, param) => {
        const variables = t.variableDeclaration(
            "const",
            [
              t.variableDeclarator(
                t.identifier(param),
                t.nullLiteral()
              )
            ]
          );

          acc.push(variable);

          return acc;
    }, []);
```

### Creating the function call
```javascript
const functionName = functionDetails.name;
const functionCall = t.variableDeclaration(
    "const",
    [
        t.variableDeclarator(
            t.identifier("result"),
            t.callExpression(
                t.identifier(functionName),
                functionParamters
            )
        )
    ]
);
```
The function call is a [CallExpression](https://github.com/babel/babel/tree/master/packages/babel-types#callexpression) in AST terms. The call will contain the function name and parameters. The function also returns a value. I'll create a [VariableDeclaration](https://github.com/babel/babel/tree/master/packages/babel-types#variabledeclaration), named result, that holds the return value of the function call, line 6.

Line 9 contains the **functionParameters** array the was created above.


### Creating a test block
Now I can create the test block. It's an [ExpressionStatement](https://github.com/babel/babel/tree/master/packages/babel-types#expressionstatement) containing a [CallExpression](https://github.com/babel/babel/tree/master/packages/babel-types#callexpression) containing a [ArrowFunctionExpression](https://github.com/babel/babel/tree/master/packages/babel-types#arrowfunctionexpression).
```javascript
const testBlockStatement = [...variables, functionCall ];
const testBlockText = "Testing " + functionDetails.name;

const testBlockAst = t.expressionStatement(
    t.callExpression(
        t.identifier("test"),
        [
            t.stringLiteral(testBlockText),
            t.arrowFunctionExpression(
                [],
                t.blockStatement(testBlockStatement)
            )
        ]
    )
);

// We are still in the reduce function here.
// Add the test block to the accumulator of
// the reduce method.
```
The ArrowFunctionExpression will have the variables and the function call. The array, **testBlockStatement**, line 1, is any array of the variables and the function call that wlll be used in the ArrowFunctionExpression in line 9.


### Creating a describe block
I'll wrap the all the test blocks within a describe block. The describe block is also a [ExpressionStatement](https://github.com/babel/babel/tree/master/packages/babel-types#expressionstatement) containing a [CallExpression](https://github.com/babel/babel/tree/master/packages/babel-types#callexpression) containing a [ArrowFunctionExpression](https://github.com/babel/babel/tree/master/packages/babel-types#arrowfunctionexpression). The ArrowFunctionExpression will have the all the test blocks.
```javascript
const describeBlock = t.expressionStatement(
    t.callExpression(
        t.identifier("describe"),
        [
            t.stringLiteral("Some Description..."),
            t.arrowFunctionExpression(
                [],
                t.blockStatement(testBlockArray)
            )
        ]
    )
);
```
Line 8 refers to the **testBlockArray** created above.

### Generate the unit test code
Finally we can generate a unit test.

```javascript
const babelGenerate = require("babel-generator").default;

const programNode = t.program([ describeBlock ], []);
const output = babelGenerate(programNode, {});

// The unit test code!!!
console.log(output.code);
```

Line 3 creates a [Program](https://github.com/babel/babel/tree/master/packages/babel-types#program) AST node that contains the variable **describeBlock** from the code above. Line 4 uses the [babel-generator](https://github.com/babel/babel/tree/master/packages/babel-generator) module to generate the unit test code. I'm passing an empty object of [options](https://github.com/babel/babel/tree/master/packages/babel-generator#options) to babel-generator. The **output** variable will be an object having a property **code** which is the unit test code. I'm only ouputing the code to the console but you can save this to a file.

Next I'll explore FunctionDeclarations that contain a call to another module and mocking/spying that call.

### Source code
Source code for this this example is at [github](https://github.com/schempy/autogenerate-unit-test-boilerplate-files/tree/master/part_02).









