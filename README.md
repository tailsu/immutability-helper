immutability-helper
===

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Downloads][downloads-image]][downloads-url]

Immutable data with maximum object reuse

This is a drop in replacement for [`react-addons-update`](https://facebook.github.io/react/docs/update.html)

### What is this?

In many libraries and frameworks, the DOM output is a function of some piece of state. As an example, in React the React Element created is just based off of `props` and/or `state`:

```js
// React
class MyApp extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'Moshe' };
  }
  render() {
    return <div>Hello {this.state.name}!</div>;
  }
}
```

If we wanted to make a change to the `state` object we would normally do something like
`state.name = 'World'` or in React `this.setState({ name: 'World' })`. This is a mutation
to the `state` object. Mutations tend to create hard to follow code. If the state starts
getting passed around to many different functions, changes can happen in those functions which
make the code a nightmare to keep track of.

A solution to this is to use something called immutable data. What that means is that you never
change the data once created (think [deep-freeze](https://github.com/substack/deep-freeze))
and instead create new objects that have the correct pieces changed. Imagine this was the state

```js
const state = {
  user: {
    name: 'Moshe',
    buildPieces: ['babel', 'webpack', 'eslint'],
    editors: ['sublime', 'atom']  
  },
  pagesRead: ['README.md']
};
```

If I wanted to change the `user.name` I could do something like this:

```js
const nextState = {
  user: {
    name: 'Dan',
    buildPieces: state.user.buildPieces,
    editors: state.user.editors
  },
  pagesRead: state.pagesRead
}
```

As you can see, there tends to be a lot of repetition when doing this. This tool
enables you to write the above as follows:

```js
var update = require('immutability-helper');

// or when using es modules
import update from 'immutability-helper';

const nextState = update(state, {
  name: {$set: 'Dan'}
});
```

The `{$name: ...}` is known as a command and follows the format that
[mongo uses](http://docs.mongodb.org/manual/core/crud-introduction/#query)

## Available commands

  * `{$push: array}` `push()` all the items in `array` on the target.
  * `{$unshift: array}` `unshift()` all the items in `array` on the target.
  * `{$splice: array of arrays}` for each item in `arrays` call `splice()` on the target with the parameters provided by the item.
  * `{$set: any}` replace the target entirely.
  * `{$merge: object}` merge the keys of `object` with the target.
  * `{$apply: function}` passes in the current value to the function and updates it with the new
  returned value.

## Examples

### Simple push

```js
var initialArray = [1, 2, 3];
var newArray = update(initialArray, {$push: [4]}); // => [1, 2, 3, 4]
```
`initialArray` is still `[1, 2, 3]`.

### Nested collections

```js
var collection = [1, 2, {a: [12, 17, 15]}];
var newCollection = update(collection, {2: {a: {$splice: [[1, 1, 13, 14]]}}});
// => [1, 2, {a: [12, 13, 14, 15]}]
```
This accesses `collection`'s index `2`, key `a`, and does a splice of one item starting from index `1` (to remove `17`) while inserting `13` and `14`.

### Updating a value based on its current one

```js
var obj = {a: 5, b: 3};
var newObj = update(obj, {b: {$apply: function(x) {return x * 2;}}});
// => {a: 5, b: 6}
// This is equivalent, but gets verbose for deeply nested collections:
var newObj2 = update(obj, {b: {$set: obj.b * 2}});
```

### (Shallow) merge

```js
var obj = {a: 5, b: 3};
var newObj = update(obj, {$merge: {b: 6, c: 7}}); // => {a: 5, b: 6, c: 7}
```

#### Extending

The main difference this has with `react-addons-update` is that
you can extend this to give it more functionality:

```js
update.extend('$addtax', function(tax, original) {
  return original + (tax * original);
});
const state = { price: 123 };
const withTax = update(state, {
  price: {$addtax: 0.8},
});
assert(JSON.stringify(withTax) === JSON.stringify({ price: 221.4 });
```

Note that `original` in the function above is the original object, so if you plan making a
mutation, you must first shallow clone the object. Another option is to
use `update` to make the change `return update(original, { foo: {$set: 'bar'} })`

If you don't want to mess around with the globally exported `update` function you can make a copy and work with that copy:

```js
import { newContext } from 'immutability-helper';
const myUpdate = newContext();
myUpdate.extend('$foo', function(value, original) {
  return 'foo!';
});
```

[npm-image]: https://img.shields.io/npm/v/immutability-helper.svg?style=flat-square
[npm-url]: https://npmjs.org/package/immutability-helper
[travis-image]: https://img.shields.io/travis/kolodny/immutability-helper.svg?style=flat-square
[travis-url]: https://travis-ci.org/kolodny/immutability-helper
[coveralls-image]: https://img.shields.io/coveralls/kolodny/immutability-helper.svg?style=flat-square
[coveralls-url]: https://coveralls.io/r/kolodny/immutability-helper
[downloads-image]: http://img.shields.io/npm/dm/immutability-helper.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/immutability-helper
