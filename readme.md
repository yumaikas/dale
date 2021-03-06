# dale

> "Don't write that stuff until you've tried to live without it and fully understand why you need it." -- [James Hague](http://prog21.dadgum.com/187.html)

dale is a tool for iterating over arrays, objects and other values. Why did I write this instead of using [underscore](https://github.com/jashkenas/underscore) or [lodash](https://github.com/lodash/lodash)? Well, because I want a very small library that only contains the looping constructs that I always need, no more.

Small as it is, dale is superior to writing `for (var a in b)` in the following respects:

1. It can iterate an input that's neither an array nor an object - this input is interpreted as being an array with one element - except that when it is `undefined`, it will be interpreted as an empty array.

   ```javascript

   [1, 2, 3]          // interpreted as [1, 2, 3]

   {a: 1, b: 2, c: 3} // interpreted as {a: 1, b: 2, c: 3}

   []                 // interpreted as []

   {}                 // interpreted as {}

   'hola!'            // interpreted as ['hola!']

   3                  // interpreted as [3]

   null               // interpreted as [null]

   undefined          // interpreted as []

   ```

2. Array iterator variables (the `a` in `for (var a in b)` whenever `b` is an array) are numbers instead of strings, so you don't have to remember to use `parseInt` to do math with the iterator.

   ```javascript

   var input = ['a', 'b', 'c'];

   for (var i in input) {
      console.log ('Element #' + (i + 1) + ' is ' + input [i]);
   }

   // This loop will print:
   //    Element #01 is a
   //    Element #11 is b
   //    Element #21 is c

   dale.go (input, function (v, k) {
      console.log ('Element #' + (k + 1) + ' is ' + v);
   });

   // This function will print:
   //    Element #1 is a
   //    Element #2 is b
   //    Element #3 is c

   ```

   This also is the case for `arguments` pseudo-arrays:

   ```javascript
   var argumentsTest = function (A, B, C) {
      dale.go (arguments, function (v, k) {
         console.log ('Element #' + (k + 1) + ' is ' + v);
      });
   }

   // The function invocation below will print:
   //    Element #1 is a
   //    Element #2 is b
   //    Element #3 is c

   argumentsTest ('a', 'b', 'c');
   ```

3. Provides functions (`stop` and `stopNot`) that allow you to exit the loop prematurely if a certain value is returned by an iteration. This allows for code that's more clear as well as more efficient.

   ```javascript

   var input = [1, 2, 'clank', 4];

   var output = [];

   for (var i in input) {
      if (typeof (input [i]) !== 'number') break;
      else output.push (input [i] * 10);
   }

   // output will be [10, 20]

   output = [];

   dale.stop (input, false, function (v, k) {
      if (typeof (v) !== 'number') return false;
      else output.push (v * 10);
   });

   // output will be [10, 20]

   ```

4. When iterating an object, by default dale will only take into account the keys that are not inherited. This means that when iterating objects you never again have to do a `hasOwnProperty` check. This default can be overriden [by passing an extra argument](https://github.com/fpereiro/dale#inherited-properties).

5. It is functional, so you can invoke dale functions within object literals to generate parts of them in a very compact and elegant way. In other words, loops become expressions instead of statements. This is probably the greatest advantage of them all.

   ```javascript

   var data = {
      key: 'value',
      key2: [1, 2, 3],
      key3: dale.go ([1, 2, 3, 4], function (v) {
         return v * 10;
      })
   }

   // data.key3 will be equal to [10, 20, 30, 40]

   ```

## Current status of the project

The current version of dale, v6.0.1, is considered to be *stable* and *complete*. [Suggestions](https://github.com/fpereiro/dale/issues) and [patches](https://github.com/fpereiro/dale/pulls) are welcome. Besides bug fixes or performance improvements, there are no future changes planned.

dale is part of the [ustack](https://github.com/fpereiro/ustack), a set of libraries to build web applications which aims to be fully understandable by those who use it.

## Installation

dale is written in Javascript. You can use it in the browser by sourcing the main file.

```html
<script src="dale.js"></script>
```

Or you can use this link to the latest version - courtesy of [jsDelivr](https://jsdelivr.com).

```html
<script src="https://cdn.jsdelivr.net/gh/fpereiro/dale@7e1be108aa52beef7ad84f8c31649cfa23bc8f53/dale.js"></script>
```

And you also can use it in node.js. To install: `npm install dale`

dale should work in any version of node.js (tested in v0.8.0 and above). Browser compatibility has been tested in the following browsers:

- Google Chrome 15 and above.
- Mozilla Firefox 3 and above.
- Safari 4 and above.
- Internet Explorer 6 and above.
- Microsoft Edge 14 and above.
- Opera 10.6 and above.
- Yandex 14.12 and above.

The author wishes to thank [Browserstack](https://browserstack.com) for providing tools to test cross-browser compatibility.

<a href="https://www.browserstack.com"><img src="https://bstacksupport.zendesk.com/attachments/token/kkjj6piHDCXiWrYlNXjKbFveo/?name=Logo-01.svg" width="150px" height="33px"></a>

## Functions

dale consists of eight functions.

### `dale.go`

`dale.go`, for vanilla iteration:
   - Takes an `input` and a `function`.
   - For each `element` in the `input`, it executes the `function`, passing the `element` and the `key` of the element as arguments and pushes the result into an `output` array.
   - Returns the `output` array.
   - Returns an array where each element is the result of each `function` application.
   - If `function` is not a function, an error will be printed and `dale.go` will return `false`.

The general idea of the function is quite similar to that of `Array.map`, but the function accepts inputs of any type.

```javascript

dale.go ([1, 2, 3], function (v) {return v + 1})
// returns [2, 3, 4]

dale.go ({a: 1, b: 2, c: 3}, function (v) {return v + 1})
// returns [2, 3, 4]

dale.go (1, function (v) {return v + 1})
// returns [2]

dale.go ({a: 1, b: 2, c: 3}, function (v, k) {return k + v})
// returns ['a1', 'b2', 'c3']

```

Notice that `dale.go` always returns an array with zero or more elements. It will only return an array of zero elements if its `input` is either an empty object, an empty array, or `undefined`.

```javascript

dale.go ([], function (v, k) {return v + 1})
// returns []

dale.go ({}, function (v, k) {return v + 1})
// returns []

dale.go (undefined, function (v, k) {return v + 1})
// returns []

```

### `dale.fil`

`dale.fil`, for iteration that filters out some results:
   - Takes an `input`, a `filterValue` and a `function`.
   - Just like `dale.go`, it iterates over the `input`. If the result of this application is **not** equal to the `filterValue`, it is pushed onto the `output` array.
   - Returns the `output` array.
   - If `function` is not a function, an error will be printed and `dale.fil` will return `false`.

```javascript

dale.fil ([{id: 1}, {id: 8}, {id: 14}], undefined, function (v) {
   if (v.id < 10) return v;
});
// returns [{id: 1}, {id: 8}]

var members = [
   {name: 'Pepe', active: true},
   {name: 'Dimitri', active: false},
   {name: 'Helmut', active: true}
];

dale.fil (members, undefined, function (v) {
   if (v.active) return {name: v.name};
});
// returns [{name: 'Pepe'}, {name: 'Helmut'}]

```

Notice that `dale.fil` always returns an array with zero or more elements.

### `dale.keys`

`dale.keys`, for returning the keys of an input (almost always an object):
   - Takes an `input`.
   - Applies the following `function` to the input: `function (v, k) {return k}` and pushes each result to the `output` array.
   - Returns the `output` array.
   - If `function` is not a function, an error will be printed and `dale.keys` will return `false`.

The general idea of this function is quite similar to that of `Object.keys`, but the function accepts inputs of any type.

```javascript

dale.keys ({'foo': true, 'bar': false, 'hip': undefined})
// returns ['foo', 'bar', 'hip']

```

Notice that `dale.keys` always returns an array with zero or more elements (each of them the `keys` of the elements within the `input`).

### `dale.stop`

`dale.stop`, for stopping the iteration when finding a certain value:
   - Takes an `input`, a `stop value` and a `function`.
   - Just like `dale.go`, it iterates over the `input`. Two things can happen:
      - If the result of this application **is equal** to the `stop value`, the result is returned and no further iteration is performed.
      - If the result of this application **is not equal** to the `stop value`, the iteration continues.
   - If the `input` is iterated completely without finding the `stop value`, the result of the last application is returned.
   - If the `input` has zero elements (because it is an empty object, empty array, or `undefined`, `dale.stop` returns `undefined`.
   - If `function` is not a function, an error will be printed and `dale.stop` will return `false`.

This function, just like `dale.stopNot` below, has two qualities that distinguish it from the other functions:
- It can stop the iteration before reaching the end of the `input`.
- It returns a single result, instead of an array of results.

```javascript

var isNumber = function (value) {
   if (typeof (value) === 'number') return true;
   else return false;
}

dale.stop ([2, 3, 4],       false, isNumber)    // returns true
dale.stop ([2, 'trois', 4], false, isNumber)    // returns false
dale.stop ([],              true,  isNumber)    // returns undefined
dale.stop (undefined,       true,  isNumber)    // returns undefined

```

### `dale.stopNot`

`dale.stopNot` is the complementary function to `dale.stop`. The only difference is that it stops when it finds a value that is **not** equal to the comparison value (which we name `stopNot value`).

```javascript

var returnIfNotNumber = function (value) {
   if (typeof (value) === 'number') return true;
   else return value;
}

dale.stopNot ([2, 3, 4],       true, returnIfNotNumber)    // returns true
dale.stopNot ([2, 'trois', 4], true, returnIfNotNumber)    // returns 'trois'
dale.stopNot ([],              true, returnIfNotNumber)    // returns undefined

```

### `dale.acc`

`dale.acc` is a function for *accumulating* results into one and then returning it. This function:
   - Takes an `input` as its first argument - it can be of any type.
   - Takes an optional `firstValue`, which can be used as the initial value for the accumulator. Otherwise, the first element of `input` will be considered as the `firstValue`. Notice that the former variant is akin to `fold`, whereas the latter is akin to `reduce`.
   - Takes a `function`, the function that accumulates two values.
   - If `function` is not a function, an error will be printed and `dale.acc` will return `false`.

The general idea of this function is quite similar to that of `Array.fold` and `Array.reduce`, but the function accepts inputs of any type.

```javascript
   dale.acc ([1, 2, 3], function (a, b) {return a + b}); // returns 6

   dale.acc ({x: 1, y: 2, z: 3}, function (a, b) {return a + b}); // returns 6

   dale.acc ([1, 2, 3], function (a, b) {return a * b}); // returns 6

   dale.acc ([2, 3], 1, function (a, b) {return a + b}); // returns 6

   dale.acc (['A', 'B', 'C'], function (a, b) {return a + b}); // returns 'ABC';
```

### `dale.obj`

`dale.obj` is like `dale.go`, but it returns an object instead. This function:
   - Always returns an object (let's name it `output`).
   - Takes an `input`, an optional `base object`, and a `function`. If a `base object` is passed that is not an object, an error will be printed.
   - For each `element` in the `input`, it executes the `function`, passing the `element` and the `key` of the element. This function application generates a `result`.
   - If `result` is an array with two elements (`result [0]` and `result [1]`), the key `result [0]` will be set to `result [1]` in either the `base object` or a new object (if no `base object` is provided).
   - If `result` is `undefined`, `output` will remain unchanged.
   - If `result` is neither an array of length 2 nor `undefined`, an error will be printed and `dale.obj` will return `false`.
   - If `function` is not a function, an error will be printed and `dale.obj` will return `false`.

```javascript

var members = [
   {name: 'Pepe', age: 68, active: true},
   {name: 'Dimitri', age: 34, active: false},
   {name: 'Helmut', age: 42, active: true}
];

dale.obj (members, function (v) {
   if (! v.active) return;
   return [v.name, v.age];
});
// returns {Pepe: 68, Helmut: 42}

var base = {
   Fritz: 46,
   Sigfrid: 24
}

dale.obj (members, base, function (v) {
   if (! v.active) return;
   return [v.name, v.age];
});
// returns {Fritz: 46, Sigfrid: 24, Pepe: 68, Helmut: 42}

dale.obj ([], function (v) {
   return [v, v];
});
// returns {}

dale.obj (members, function (v) {
   return /thisisinvalid/
}));
// returns false and prints the following error: `fun passed to dale.obj must return undefined or an array of length 2 but instead returned a value of type regex`

dale.clog (output = dale.obj (members2, function (v) {
   return [1, 2, 3];
}));

// returns false and prints the following error: `fun passed to dale.obj must return undefined or an array of length 2 but instead returned an array of length 0`
```

Notice that `dale.obj` always returns an object with zero or more elements, unless one of the invocations to `fun` returns an invalid value.

One important point: if you pass a `base object`, the original object will be modified. This means that if you want to preserve the original object, you must either copy it first or avoid using `dale.obj`.

```javascript

console.log (base);
// `base` will be equal to {Fritz: 46, Sigfrid: 24, Pepe: 68, Helmut: 42}

```

### `dale.times`

`dale.times` is a function that generates an array of numbers - it is an indirect equivalent of a `while` loop, since it creates an array with consecutive integers `[1, 2, ..., n - 1, n]` which can then be passed to any of the other functions - most often, `dale.go`.

The only required parameter is `times`, which can be either a positive integer or 0, and which states the length of the returned array.

```javascript
dale.times (3);
// returns [1, 2, 3];

dale.go (dale.times (3), function (v) {return v + 1});
// returns [2, 3, 4];
```

The default starting value for the first element of the array returned by `dale.times` is `1`. However, this can be changed by passing a second argument, `start`, to `dale.times`. `start` can be either an integer or a float.

```javascript
dale.times (3, 0);
// returns [0, 1, 2];

dale.times (3, 0.5);
// returns (0.5, 1.5, 2.5);
```

The default increment used by `dale.times` is `1`. You can change it by passing a third parameter, `step`, which can be an integer or a float.

```javascript
dale.times (3, 1, 2);
// returns [1, 3, 5];

dale.times (3, 0, -1);
// returns (0, -1, -2);

dale.times (3, 0, 0);
// returns [0, 0, 0];
```

Here's how you can use `dale.times` within the context of other functions.

```javascript
   // returns [2, 3, 4]
   dale.go (dale.times (3), function (v) {return v + 1}));

   // returns [2, 4]
   dale.fil (dale.times (4), undefined, function (v) {
      if (v % 2 === 0) return v;
   });

   // returns {1: 2}
   dale.obj (dale.times (2), function (v, k) {
      if (v % 2 === 0) return [k, v];
   });

   // returns true
   dale.stop (dale.times (2), false, function (v, k) {
      return v % 3 !== 0;
   });

   // returns false
   dale.stop (dale.times (4), false, function (v, k) {
      return v % 3 !== 0;
   });

   // returns 15
   dale.acc (dale.times (5), function (a, b) {return a + b});
```

If `dale.times` receives invalid inputs, it will print an error message and return `false`.

## Inherited properties

By default, dale functions iterate an object, it will only iterate the keys that belong to the object directly, ignoring inherited keys.

```javascript
   var o1 = {foo: 42}
   var o2 = Object.create (o1);  // o2 inherits from o1

   dale.keys (o1); // returns ['foo']
   dale.keys (o2); // returns []

   dale.go (o1, function (v) {return v});       // returns [42]
   dale.go (o2, function (v) {return v});       // returns []
```

If you want dale functions to iterate the inherited properties of an object, pass `true` as the last argument to the function.

```javascript
   dale.keys (o2, true)                         // returns ['foo']
   dale.go (o2, function (v) {return v}, true); // returns [42]
```

## Logging

For compatibility with old browsers, dale defines and uses `dale.clog`, a function that will default to `console.log` and, if `console.log` is absent, will emit an `alert` statement instead.

## Performance

dale is necessarily slower than a `for` loop, since it consists of a functional wrapper on top of a `for` loop. Besides its features, dale's emphasis on code succintness (which is achieved by having a single variadic function generating the main loop) probably adds an extra performance hit.

The benchmark I used is included in `example.js` - to execute it just run that file, or open it in a browser. The benchmark attempts to make many iterations without almost any computation, to focus on the raw speed of the underlying loop (be it a real loop or dale's layer on top of it).

Testing different versions of node.js and the supported browsers, here's some (very approximate) performance factors:

```
Iterating arrays:

for:  1x
dale: 2x-9x

Iterating objects:

for:                                    1x
dale:                                   2x-10x
dale, without the hasOwnProperty check: 2x-8x
```

This means that dale takes roughly twice when iterating arrays and up to *ten times* more time when iterating objects. Although significant, I believe this is a worthy price to pay for the ease of expression and the facilities provided by dale - especially since many of these facilities have to be inserted into the loops anyway, hence bringing down the speed of a raw `for` loop.

The modern the javascript engine, the better will dale's performance be relative to that of a `for` loop.

## Source code

The complete source code is contained in `dale.js`. It is about 160 lines long.

Below is the annotated source.

```javascript
/*
dale - v6.0.1

Written by Federico Pereiro (fpereiro@gmail.com) and released into the public domain.

Please refer to readme.md to read the annotated source.
*/
```

### Setup

We wrap the entire file in a self-executing anonymous function. This practice is commonly named [the javascript module pattern](http://yuiblog.com/blog/2007/06/12/module-pattern/). The purpose of it is to wrap our code in a closure and hence avoid making the local variables we define here to be available outside of this module. A cursory test indicates that local variables exceed the scope of a script in the browser, but not in node.js. This means that this pattern is useful only on the browser.

```javascript
(function () {
```

Since this file must run both in the browser and in node.js, we define a variable `isNode` to check where we are. The `exports` object only exists in node.js.

```javascript
   var isNode = typeof exports === 'object';
```

This is the most succinct form I found to export an object containing all the public members (functions and constants) of a javascript module. Note that, in the browser, we use the global variable `dale` to export the library.

```javascript
   if (isNode) var dale = exports;
   else        var dale = window.dale = {};
```

The `type` function below is <del>copypasted</del> taken from [teishi](https://github.com/fpereiro/teishi). This is because I needed dale when writing teishi more than I needed teishi when writing dale. Thus, I decided that teishi should depend on dale. And I don't know if I can elegantly cross-reference both libraries, taking just what I need and avoiding circular dependencies.

The purpose of `type` is to create an improved version of `typeof`. The improvements are three:

- Distinguish between types of numbers: `nan`, `infinity`, `integer` and `float` (all of which return `number` in `typeof`).
- Distinguish between `array`, `date`, `null`, `regex` and `object` (all of which return `object` in `typeof`).
- Fix quirks of old browsers.

For the other types that `typeof` recognizes successfully, `type` will return the same value as `typeof`.

`type` takes a single argument (of any type, naturally) and returns a string with its type.

The possible types of a value can be grouped into three:
- *Values which `typeof` detects appropriately*: `boolean`, `string`, `undefined`, `function`.
- *Values which `typeof` considers `number`*: `nan`, `infinity`, `integer`, `float`.
- *values which `typeof` considers `object`*: `array`, `date`, `null`, `regex` and `object`.

If you pass `true` as a second argument, `type` will distinguish between *plain objects* (ie: object literals) and other objects. If you pass an object that belongs to a class, `type` will return the lowercased class name instead.

The clearest example of this is the `arguments` object:

```javascript
type (arguments)        // returns 'object'
type (arguments, true)  // returns 'arguments'
```

Before we define `type`, we define `argdetect`, a local variable that will be `true` in most javascript engines. In Internet Explorer 8 and below, however, it is not possible to get the type of the prototype of an `arguments` pseudo-array, hence the definition of this variable (which will be used in `type` and also once more later).

The variable gets its value from a self-execution anonymous function. We need to do this since the `arguments` pseudo-array is only defined in the context of a function.

```javascript
   var argdetect = (function () {return Object.prototype.toString.call (arguments).match ('arguments')}) ();
```

Below is the function.

```javascript
   var type = function (value, objectType) {
      var type = typeof value;
      if (type === 'function') return Object.prototype.toString.call (value).match (/regexp/i) ? 'regex' : 'function';
      if (type !== 'object' && type !== 'number') return type;
      if (value instanceof Array) return 'array';
      if (type === 'number') {
         if      (isNaN (value))      return 'nan';
         else if (! isFinite (value)) return 'infinity';
         else if (value % 1 === 0)    return 'integer';
         else                         return 'float';
      }
      if (value === null) return 'null';
      type = Object.prototype.toString.call (value).replace ('[object ', '').replace (']', '').toLowerCase ();
      if (type === 'array' || type === 'date') return type;
      if (type === 'regexp') return 'regex';
      if (objectType) return argdetect ? type : (type (value.callee) === 'function' ? 'arguments' : type);
      return 'object';
   }
```

We define `dale.clog`, a function that invokes `console.log` where available and uses the browser's `alert` native method otherwise.

```javascript
   try {
      dale.clog = console.log.bind (console);
   }
   catch (error) {
```

If we're inside the `catch`, it means that `console` might not be supported. It could also be the case that `bind` is not supported (which happens on browsers without full ES5 support.

```javascript
      dale.clog = function () {
```

We construct a string with the message to be printed.

```javascript
         var output = dale.go (arguments, function (v) {return v === undefined ? 'undefined' : v}).join (' ');
```

If `window.console` is available, we invoke it and print `output`; otherwise, we invoke `alert`. There's nothing else to do, so we close the function and the `catch` block.

```javascript
         window.console ? window.console.log (output) : alert (output);

      }
   }
```

### The main function

All eight functions of dale have many common elements. As a result, I've factored out the common elements in the function `make` below (short for `make function`).

`make` function receives a `what` argument (can be any of `'go'`, `'obj'`, `'fil'`, `'stop'`, `'stopNot'`). It will then return the corresponding dale function.

`dale.keys`, `dale.times` and `dale.acc` are wrappers around the other functions, so the function below is actually concerned with the other five functions.

```javascript
   var make = function (what) {
      return function (input, second, third, fourth) {
```

For `dale.go`, the arguments received must be `input`, `fun` and an optional `inherit` flag, which must be `true`. We also set `output` to an empty array. Finally, we create a variable `index` to keep track of how many items we added to `output` array - this is strictly for performance purposes.

```javascript
         if      (what === 'go')              var fun = second, inherit = third  === true, output = [], index = 0;
```

For `dale.fil`, the arguments received must be `input`, `second` (which will be the value that must be filtered out), `fun` and an optional `inherit` flag. As with `dale.go`, we set `output` to an empty array and `index` to `0`.

```javascript
         else if (what === 'fil')             var fun = third,  inherit = fourth === true, output = [], index = 0;
```

The case below corresponds both to `dale.stop` and `dale.stopNot`, which receive `input`, `second` (the value which must be compared to see if the loop has to stop), `fun` and an optional `inherit` flag. We define `output` to be `undefined`.

```javascript
         else if (what !== 'obj')             var fun = third,  inherit = fourth === true, output;
```

If we're here, we're dealing with `dale.obj`. We will consider the case where we receive a base object between `input` and `fun`. In this case, we set `output` to the argument between `input` and `fun`. As with the cases above, we recognize the `inherit` flag.

```javascript
         else if (type (second) === 'object') var fun = third,  inherit = fourth === true, output = second;
```

Finally, we consider the case where `dale.obj` doesn't receive a base object. In this case, we accept `input`, `fun`, `inherit` and initialize `output` to an empty object.

```javascript
         else                                 var fun = second, inherit = third  === true,                     output = {};
```

We check the type of the arguments. Since `input` and `second` can be anything (except for the case of `dale.obj`, for which we have already checked the type of `second`), we just need to check that `fun` is indeed a function.

If `fun` is not a function, we log an error and return `false`.

```javascript
         if (type (fun) !== 'function') {
            dale.clog (((what === 'go' || (what === 'obj' && type (second) !== 'object')) ? 'Second' : 'Third') + ' argument passed to dale.' + what + ' must be a function but instead is', fun, 'with type', type (fun));
            return false;
         }
```

For any dale function, if the `input` is `undefined`, we return the default `output`. Notice that in this case, the function returns without executing the `fun` even once.

```javascript
         if (input === undefined) return output;
```

We define a local function `inner`, which we'll execute for each item of `output` we are iterating. This function takes a single argument, `result`, which will be the result of applying each element of `output` to `fun`.

If `inner` returns `true`, this will mean that the iteration process (defined below) will be stopped.

```javascript
         var inner = function (result) {
```

For the case of `dale.go`, or the case of `dale.fil` when `result` is not equal to `second`, we append `result` into `output`. As we do this, we increment `index` (which is only defined for these two cases).

```javascript
            if      (what === 'go')   output [index++] = result;
            else if (what === 'fil') {
               if (result !== second) output [index++] = result;
            }
```

For the case of `dale.obj`:

- If `result` is `undefined`, no key will be set. We do nothing.
- If `result` is not an `array` of length 2, we set `output` to `false` and print an error message through `dale.clog`.
- Otherwise, we set the key `result [0]` of `output` to `result [1]`.

```javascript
            else if (what === 'obj') {
               if (result === undefined) return;
               if (type (result) !== 'array' || result.length !== 2) {
                  dale.clog (type (result) === 'array' ? ('fun passed to dale.obj must return undefined or an array of length 2 but instead returned an array of length ' + result.length) : ('fun passed to dale.obj must return undefined or an array of length 2 but instead returned a value of type ' + type (result)));
                  output = false;
                  return true;
               }
               output [result [0]] = result [1];
            }
```

For the case of `dale.stop` and `dale.stopNot`, we set `output` to `result`. If we're in `dale.stop` and `result` is equal to `second`, we return `true` to break the loop. We do the same with `dale.stopNot` when `result` is *not* equal to `second`. Otherwise, we return `false`, which will do nothing.

```javascript
            else {
               output = result;
               return what === 'stop' ? result === second : result !== second;
            }
```

We close `inner`.

```javascript
         }
```

We save the type of `input` in a local variable `inputType`.

```javascript
         var inputType = type (input);
```

If `input` is an object, we want to check whether this is an `arguments` object, which we want to treat like an array. To ascertain this we use `Object.prototype.toString`. We use this method instead of our `type` function simply for performance purposes. If it is indeed an `arguments` object, we will convert it into an array.

Note that if `argdetect` is false (which will happen in IE 8 and below), we check whether `input.callee` is a function to ascertain whether this is an `arguments` object. This is a workaround since there seems to be no straightforward way to detect whether a certain object is an `arguments` object for these particular browsers.

```javascript
         if (inputType === 'object') {
            if (Object.prototype.toString.call (input) === '[object Arguments]' || (! argdetect && type (input.callee) === 'function')) inputType = 'array', input = [].slice.call (input);
         }
```

We're ready to iterate `input`! We first tackle the case where `inputType` is an array.

```javascript
         if (inputType === 'array') {
```

We use a `for` loop to iterate over `input`.

```javascript
            for (var key = 0; key < input.length; key++) {
```

We call `inner` passing it `result`. `result` will be the result of invoking `fun` with two arguments: `input [key]`, which is the element being iterated, plus the key. Notice that we apply `parseInt` to the key. This is because javascript returns stringified numeric iterators (`'0'`, `'1'`, `'2'`...) when looping an array, instead of numeric keys.

Notice also that the `fun` receives the `value` as the first argument and the `key` as the second. This inversion is useful since usually the `fun` needs the `value` but not the `key`. In this case, with this argument ordering you can write `function (v) {...}` instead of `function (k, v) {...}`.

If `inner` returns `true`, the iteration will be stopped through a `break` statement.

There's nothing else to do, so we close the loop and the case.

```javascript
               if (inner (fun (input [key], parseInt (key)))) break;
            }
         }
```

If `input` is an object:

```javascript
         else if (inputType === 'object') {
```

We iterate `input` with a `for ... in` loop.

```javascript
            for (var key in input) {
```

If two conditions are met simultaneously, we skip the current `key`, by issuing a `continue` statement. These conditions are:
- `inherit` is not set.
- `input` has `key` as an inherited property.

We also check that `input` has the `hasOwnProperty` method. Since `input` is an object, you would think that it has the `hasOwnProperty` method. However, when iterating a DOM element's [attributes](https://developer.mozilla.org/en-US/docs/Web/API/Element/attributes) or [style](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style) object in Internet Explorer 8 and below, the method will be absent despite these objects behaving like plain objects. For this reason, we add this extra check. If the method is absent, we won't skip the key since we have no way of ascertaining whether the object actually contains the key or inherited it.

```javascript
               if (! inherit && input.hasOwnProperty && ! input.hasOwnProperty (key)) continue;
```

We now invoke `inner` with the result of invoking `fun` with two arguments (the element being iterated, plus its key). If this invocation returns `true`, we stop the loop with a `break` statement.

There's nothing else to do, so we close the loop and the case.

```javascript
               if (inner (fun (input [key], key))) break;
            }
         }
```

If `input` is neither an array nor an object, we invoke `inner` a single time, passing `input` itself as its first argument and `0` as its second. We pass `0` as the second argument because if `input` is neither an array nor an object, we consider it to be an array of one element only - hence, the key of its only element will be `0`.

There's no point in recording whether the invocation of `inner` returned `true`, since we don't need to interrupt an iteration of a single element.

```javascript
         else inner (fun (input, 0));
```

At this point, we're done. The return value we need is at `output`. We return it, then close the function returned by `make` and `make` itself.

```javascript
         return output;
      }
   }
```

### The eight functions

We create each of the dale functions. `dale.keys` is simply a lambda function that passes `input` and `inherit` to `dale.go`, using a `fun` that only returns its `key`.

```javascript
   dale.go      = make ('go');
   dale.fil     = make ('fil');
   dale.obj     = make ('obj');
   dale.stop    = make ('stop');
   dale.stopNot = make ('stopNot');
   dale.keys      = function (input, inherit) {return dale.go (input, function (v, k) {return k}, inherit)};
```

`dale.times` is meant as a replacement for while loops where we iterate through integers. This function takes `times` (the number of iterations), an optional `start` parameter to use as the first element of the array and an optional `step` parameter to specify the increment used.

```javascript
   dale.times   = function (steps, start, step) {
```

We validate and initialize the inputs:

- `steps` has to be an integer larger than or equal to 0. If it's equal to 0, we immediately return an empty array.
- `start` is either undefined or an integer or a float. If undefined, we set it to 1.
- `step` is either undefined or an integer or a float. If undefined, we set it to 1.

If any of these conditions is violated, we print an error message and return `false`.

```javascript
      if (steps === 0) return [];
      if (type (steps) !== 'integer' || steps < 0) {
         dale.clog ('steps must be a positive integer or zero.');
         return false;
      }
      if (start === undefined) start = 1;
      else if (type (start) !== 'integer' && type (start) !== 'float') {
         dale.clog ('start must be an integer or float.');
         return false;
      }
      if (step  === undefined) step  = 1;
      else if (type (step) !== 'integer'  && type (step)  !== 'float') {
         dale.clog ('step must be an integer or float.');
         return false;
      }
```

We initialize `output` to an array with one element, `start`.

```javascript
      var output = [start];
```

The while loop to end most while loops.

```javascript
      while (i < steps) {
```

We increment `start` by `step` and then immediately push it onto `output`. We then close the while loop.

```javascript
         start += step;
         output.push (start);
      }
```

We return `output` and close the function.

```javascript
      return output;
   }
```

`dale.acc` is a function for *accumulating* results into one, using a function that takes an accumulator (result of previous operations) and a new item on each iteration. It provides the functionality of the functional operations commonly named `reduce` and `fold`.

While this function could be possibly inserted into the main function that implements most of the other functions, I felt that its particularities would make the main function more complex and slower, so I opted to write it as a standalone function that calls `dale.go` in its inner loop.

This function takes up to four arguments.

- The first argument is always `input`, which can be of any type.
- `acc` is the initial value for the accumulator. If `acc` is omitted, we consider the first element of `input` to be `acc`.
- `fun` is the function that does the *reduction* operation on two arguments.
- `inherit` is a flag that determines whether to iterate or not the inherited elements of `input`.

```javascript
   dale.acc = function (input, second, third, fourth) {
```

To determine whether it was passed or not, we count the amount of arguments passed. `fun` (the following argument) is required, but the `inherit` flag is also optional. To note whether `inherit` is passed we check whether the last argument is boolean (since `fun` cannot be a boolean).

Now, if `inherit` is not passed, then if there's 3 arguments we will consider `acc` to be the second one (with `fun` being the third). If, however, `inherit` is passed, we'll consider `acc` to be passed if there's 4 arguments.

```javascript
      var hasAcc  = arguments.length === (type (arguments [arguments.length - 1]) === 'boolean' ? 4 : 3);
      var acc     = hasAcc ? second : undefined;
      var fun     = hasAcc ? third  : second;
      var inherit = hasAcc ? fourth : third;
```

We check whether `fun` is of the right type. Otherwise, we print an error and return `false`.

```javascript
      if (type (fun) !== 'function') {
         dale.clog ('fun must be a function but instead is of type', type (fun));
         return false;
      }
```

We initialize a variable `first` that will tell us whether we are in the first element of `input` or not - this is necessary for the case where `input` is an object.

```javascript
      var first = true;
```

We invoke `dale.go` on our `input` and a function.

```javascript
      dale.go (input, function (v, k) {
```

If no accumulator was passed and we're iterating the first element of `input`, we set `first` to `false` (to mark that we have already processed it) and then we set `acc` to the value of this element.

```javasript
         if (! hasAcc && first) {
            first = false;
            return acc = v;
         }
```

In every other case, we set `acc` to the value returned by `fun` when it is passed `acc` and the element currently being iterated.

```javascript
         acc = fun (acc, v);
```

We close the inner loop. Notice we pass the `inherit` flag to `dale.go`.

```javascript
      }, inherit);
```

We return `acc` and close the function.

```javascript
      return acc;
   }
```

We close the module.

```javascript
}) ();
```

## License

dale is written by Federico Pereiro (fpereiro@gmail.com) and released into the public domain.
