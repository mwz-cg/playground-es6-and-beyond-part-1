# Introduction

JavaScript *used to* be considered one of those "lesser" languages, kind of like PHP, that were deemed unsuitable for "real" programming. After all, yes, JavaScript is dynamically typed (the type of a variable is known/can vary at runtime) and is indecently permissive/forgiving. Here are a few things that you can do without having runtime errors:
- add values of different types (such as numbers and strings),
- use a variable that has not been assigned before,
- reference non-existent properties of an object,
- call a function with less or more arguments than it expects.

This works because JavaScript has implicit type conversion, so it tries to make sense of what it is supposed to do when computing `1 + '2'` for instance. Also, JavaScript has an `undefined` type, which is the type of an expression that is, well, not defined, such as accessing a non-existent property or a missing argument.

JavaScript has been an international standard since 1997, when it was first standardized as ECMAScript. There are several editions of ECMAScript, and the *n*th edition of ECMAScript is generally abbreviated as ES*n*: ES6 means the sixth edition (published in 2015). We will explore in a series of playgrounds the modern features of the language that will make your code more robust, concise, and easier to read.

# The not-so-distant past (ES5)

## Strict mode

Starting with ES5 (2009), JavaScript has a *strict mode* that does just what it says, i.e. it disallows things that are considered too lenient. Among other things, strict mode:

- removes legacy octal notation for numbers and escape sequences in strings,
- restricts the use of `eval` and runs the evaluated code in an isolated environment,
- restricts the use of `arguments` and disables `caller`/`callee` properties,
- makes the `with` statement a syntax error,
- forbids having multiple properties/arguments with the same name,
- no longer coerces `this` to the global object if it is `null` or `undefined`.

This last point is shown below:

```javascript runnable
function succeeds() {
    // in non-strict mode, this is the global object...
    this.console.log('hello');
}

function fails() {
    'use strict';
    // in strict mode, this is undefined here
    this.console.log('fails');
}

succeeds();
fails();
```

TL;DR you should always `'use strict'` in your code to make it cleaner by not using what are mostly legacy features that have cleaner replacements.

## Higher order functions

ES5 also added higher-order functions to arrays. This means that if you wanted to create a new array from another array with values incremented, instead of doing it the old way:

```javascript runnable
var values = [0, 1, 2, 3];
var incremented = [];
for (var i = 0; i < values.length; i++) {
    incremented.push(values[i] + 1);
}

console.log(incremented);
```

You could abstract the iteration away and create a new array using the `map` function:

```javascript runnable
var values = [0, 1, 2, 3];
var incremented = values.map(function (element) {
    return element + 1;
});
console.log(incremented);
```

We will see later in this playground how we can further improve this example.

# The present (ES6)

With the release of ES6 (2015), JavaScript really caught up with other programming languages. This version introduced many changes and new features that make JavaScript far more powerful, while also fixing a few long-standing issues.

## Use `let` and `const`, not `var`

There are two problems with `var`. First, it has function scope even if it is declared in a nested block. This means that outside the `for` loop below, you can still use the variable `i`.

```javascript runnable
function problemWithVar1() {
    for (var i = 0; i < 3; i++) {
    }
    console.log(i);
}

problemWithVar1();
```

This prints `3`. Change `var` to `let` and observe the difference.

The second problem is that within a function two declarations of a `var x` will in fact be the same variable `x`. For instance:

```javascript runnable
function problemWithVar2() {
    var x = 3;
    if (x > 2) {
        var x = 4;
        console.log(x);
    }
    console.log(x);
}

problemWithVar2();
```

What does this print? Try to replace `var` by `let` and see what happens. Move the `let x = 4;` statement before the `if`, and run the code again.

Note how you are allowed to declare a different variable with the same name as long as it is not in the same scope.

`const` behaves similarly to `let` except that it does not let you assign a different value to the variable. This is referred to as an immutable binding: the binding (association of the value to the identifier) is not mutable, though the value itself is, as illustrated by the code below:

```javascript runnable
const obj = {x: 3};
console.log(obj);

// allowed
obj.x++;
console.log(obj);

// not allowed
obj = {x: 5};
console.log(JSON.stringify(obj));
```

## Use arrow functions instead of anonymous functions

As you already know, JavaScript has long had first-class support for functions: you can pass functions as arguments to other functions and create functions dynamically. However, the `function ()` syntax can feel a bit clunky, and so-called arrow functions are a nice improvement in syntax. More than that though, by reducing visual burden, they also enable a way of thinking that is traditionally found in functional programming languages.

Our ES5 code for mapping an array by incrementing all values was:

```javascript runnable
var values = [0, 1, 2, 3];
var incremented = values.map(function (element) {
    return element + 1;
});
console.log(incremented);
```

Compare this to the ES6 version:

```javascript runnable
const values = [0, 1, 2, 3];
const incremented = values.map(element => element + 1);
console.log(incremented);
```

An arrow function can either return an expression directly (as is the case here), or have a normal body with statements if it begins with `{`. This means that if you want to return an object directly you should wrap it in parentheses. An arrow function with 0 parameters or more than 1 parameter must have a list of parameters in parentheses:

```javascript
const add = (x, y) => x + y;
```

Interestingly, with arrow functions it becomes much more feasible to *currify* a function, i.e. transform the `add` function into this:

to this:
```javascript
const add = x => y => x + y;
```

What is the difference? You can now use *partial application* to increment elements with `add` as follows:

```javascript runnable
const add = x => y => x + y;

const values = [0, 1, 2, 3];
const incremented = values.map(add(1));
console.log(incremented);
```

This pattern is used in practice, for instance a middleware in Redux will look like this:

```javascript
function myMiddleware() {
    return next => action => {
        // do something with action
        // ...

        // Call the next dispatch method in the middleware chain.
        return next(action)
    }
}
```

### Arrow functions and this

Another advantage of arrow functions is that they keep the existing value of `this`. For instance, the following ES5 code does not work:

```javascript runnable
'use strict';

var incrementer = {
    sum: 0,
    computeSum: function(values) {
        values.forEach(function(value) {
            this.sum += value;
        });
    }
};

incrementer.computeSum([1, 2, 3, 4]);
```

That's because in strict mode, `this` is actually `undefined` in the inner function. There are several workarounds: you could create a `self` variable outside of the `forEach` that equals `this`, you could pass `this` as an additional argument to `forEach`, you could `bind` the inner function to `this`, or you can just use an arrow function in ES6:

```javascript runnable
'use strict';

let incrementer = {
    sum: 0,
    computeSum: values => values.forEach(value => this.sum += value)
};

incrementer.computeSum([1, 2, 3, 4]);
console.log(incrementer.sum);
```

## Use default arguments

One of JavaScript's great strength is its flexibility when calling functions. There is no such thing as a function signature: you can call a function with fewer or more arguments than it declares. Optional arguments have been supported since day one, but before ES6 they required boilerplate code such as:

```javascript runnable
function cons(item, list) {
    // one of:
    list = list || [];
    // or:
    // if (arguments.length == 1 || list === undefined) { list = []; }
    list.unshift(item);
    return list;
}

console.log(cons(1, cons(2, cons(3))));
console.log(cons(1, cons(2, cons(3, undefined))));
```

Now instead you can simply use default parameters:

```javascript runnable
function cons(item, list = []) {
    list.unshift(item);
    return list;
}

console.log(cons(1, cons(2, cons(3))));
console.log(cons(1, cons(2, cons(3, undefined))));
```

The default value of a parameter can be another parameter that is declared before it:

```javascript runnable
const assert = require('assert');

function multiply(a, b = a) {
    return a * b;
}

assert(multiply(3, 4) == 12);
assert(multiply(5) == 25);
```

Finally, `arguments` reflect the arguments that were given to the function, not the arguments that are available after default values have been set:

```javascript runnable
const assert = require('assert');

function multiply(a, b = 5) {
    assert arguments.length == 1 && arguments[1] === undefined;
    return a * b;
}

assert(multiply(3) == 15);
```
