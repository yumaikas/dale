# dale

> "Don't write that stuff until you've tried to live without it and fully understand why you need it." -- [James Hague](http://prog21.dadgum.com/187.html)

dale is a tool for iterating over arrays and objects. Why did I write this instead of using [underscore](https://github.com/jashkenas/underscore) or [lodash](https://github.com/lodash/lodash)? Well, because I want a very small library that only contains the looping constructs that I always need, no more.

Small as it is, dale is superior to writing ```for (var a in b)``` in the following respects:

1. It can iterate an input that's neither an array nor an object - this object can be interpreted as being an array with either zero or one elements (depending on whether it is undefined or not).
2. Array iterator variables (the ```a``` in ```for (var a in b)```) are numbers instead of strings, so you don't have to recur to ```parseInt``` to do math with the iterator.
3. Allows you to exit the loop prematurely if a certain value is returned by an iteration. This allows for code that's more clear as well as more efficient.
4. It is a function, so you can invoke it within object literals to generate parts of them in a very compact and elegant way. This is probably the greatest advantage of them all.

## Installation

dale is written in Javascript. You can use it in the browser by sourcing the main file.

```html
<script src="dale.js"></script>
```

And you also can use it in node.js. To install: `npm install dale`

## Functions

dale consists of just three functions.

### dale.do

The main function is `dale.do`. This function does the following:

- It takes exactly two arguments
   1. A `value`, which can be any javascript element (object, array, string, number, regex, boolean, function, undefined).
   2. A `function`.
- If the `function` is not a function, an error will be informed through `console.log` and `dale.do` will return false.
- If the input is valid, `dale.do` always returns an array. The following cases arise:
   1. If `value` is undefined, an empty array or an empty object, then an empty array is returned.
   2. If `value` is a single non-undefined value (such as true, a string or a number), the `function` is executed once, receiving `value` as its sole parameter. In this case, `dale.do` returns an array with the result of `function (value)` as its sole element.
   3. If `value` is an array or object with one or more keys/elements, the `function` is executed once per key/element, receiving as arguments the value first and the iterator second (which is a number in the case of an array and a string in the case of an object).

*Examples:*

```javascript
dale.do (undefined, some_func)             // returns []

dale.do ([], some_func)                    // returns []

dale.do ({}, some_func)                    // returns []

dale.do ('dale viteh', some_func)          // returns [some_func ('dale viteh')]

dale.do ([1, 2, 3], function (v) {return v + 1})
// returns [2, 3, 4]

dale.do ({a: 1, b: 2, c: 3}, function (v, k) {return k + v})
// returns ['a1', 'b2', 'c3']

dale.do ([1, 2, 3], function (v, k) {return v + ' is element #' + k})
// returns ['1 is element #0', '2 is element #1', '3 is element #2']
```

Notice that for each iteration, the value is passed as the first argument to the function and the key as the second. This is because values are used more often than keys, so many times you can just omit the keys argument when writing the `function`.

### dale.stop\_on

`dale.stop_on` is a function very similar to `dale.do`. The main difference is that it can stop the iterative process when it finds a specific value.

- It receives an extra argument. Hence, the input to `dale.stop_on` is:
   1. A `value` (same than `dale.do`)
   2. A `stop_on_value` (can be any value).
   3. A `function` (same than `dale.do`).
- As in `dale.do`, if it receives `undefined` or an empty array/object as `value`, it will return an empty array.
- As in `dale.do`, the `function` is executed once for each element in the value.
- If the result is equal to the `stop_on_value`, `dale.stop_on` stops and returns the `stop_on_value`.
- If the `stop_on_value` was not found, the *last* result of the iteration is returned.

*Examples:*

```javascript
function is_number (value) {
   if (typeof (value) === 'number') return true;
   else return false;
}

dale.stop_on (undefined, true, is_number)           // returns undefined
dale.stop_on ([], true, is_number)                  // returns undefined
dale.stop_on (['hey', 2, 'ho'], true, is_number)    // returns true
dale.stop_on (['hey', 'hi', 'ho'], true, is_number) // returns false
```

### dale.times

The function `dale.times` is an useful shorthand for passing an array of consecutive numbers as the `value` of `dale.do`.

*Examples*:

```javascript
dale.times (3, function (v) {return v})       // returns [1, 2, 3]
dale.times (3, function (v) {return 'a' + v}) // returns ['a1', 'a2', 'a3']
```

## Source code

The complete source code is contained in `dale.js`. It is about 130 lines long.

## License

lith is written by Federico Pereiro (fpereiro@gmail.com) and released into the public domain.
