# Path-to-RegExp

> Turn an Express-style path string such as `/user/:name` into a regular expression.

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]
[![Dependency Status][david-image]][david-url]
[![License][license-image]][license-url]
[![Downloads][downloads-image]][downloads-url]

## Installation

```
npm install path-to-regexp --save
```

## Usage

```javascript
var pathToRegexp = require('path-to-regexp')

// pathToRegexp(path, keys, options)
// pathToRegexp.parse(path)
// pathToRegexp.compile(path)
```

- **path** A string in the express format, an array of strings, or a regular expression.
- **keys** An array to be populated with the keys present in the url.
- **options**
  - **sensitive** When `true` the route will be case sensitive. (default: `false`)
  - **strict** When `false` the trailing slash is optional. (default: `false`)
  - **end** When `false` the path will match at the beginning. (default: `true`)

```javascript
var keys = []
var re = pathToRegexp('/foo/:bar', keys)
// re = /^\/foo\/([^\/]+?)\/?$/i
// keys = [{ name: 'bar', prefix: '/', delimiter: '/', optional: false, repeat: false, pattern: '[^\\/]+?' }]
```

### Parameters

The path has the ability to define parameters and automatically populate the keys array.

#### Named Parameters

Named parameters are defined by prefixing a colon to the parameter name (`:foo`). By default, this parameter will match up to the next path segment.

```js
var re = pathToRegexp('/:foo/:bar', keys)
// keys = [{ name: 'foo', ... }, { name: 'bar', ... }]

re.exec('/test/route')
//=> ['/test/route', 'test', 'route']
```

#### Suffixed Parameters

##### Optional

Parameters can be suffixed with a question mark (`?`) to make the entire parameter optional. This will also make any prefixed path delimiter optional (`/` or `.`).

```js
var re = pathToRegexp('/:foo/:bar?', keys)
// keys = [{ name: 'foo', ... }, { name: 'bar', delimiter: '/', optional: true, repeat: false }]

re.exec('/test')
//=> ['/test', 'test', undefined]

re.exec('/test/route')
//=> ['/test', 'test', 'route']
```

##### Zero or more

Parameters can be suffixed with an asterisk (`*`) to denote a zero or more parameter match. The prefixed path delimiter is also taken into account for the match.

```js
var re = pathToRegexp('/:foo*', keys)
// keys = [{ name: 'foo', delimiter: '/', optional: true, repeat: true }]

re.exec('/')
//=> ['/', undefined]

re.exec('/bar/baz')
//=> ['/bar/baz', 'bar/baz']
```

##### One or more

Parameters can be suffixed with a plus sign (`+`) to denote a one or more parameters match. The prefixed path delimiter is included in the match.

```js
var re = pathToRegexp('/:foo+', keys)
// keys = [{ name: 'foo', delimiter: '/', optional: false, repeat: true }]

re.exec('/')
//=> null

re.exec('/bar/baz')
//=> ['/bar/baz', 'bar/baz']
```

#### Custom Match Parameters

All parameters can be provided a custom matching regexp and override the default. Please note: Backslashes need to be escaped in strings.

```js
var re = pathToRegexp('/:foo(\\d+)', keys)
// keys = [{ name: 'foo', ... }]

re.exec('/123')
//=> ['/123', '123']

re.exec('/abc')
//=> null
```

#### Unnamed Parameters

It is possible to write an unnamed parameter that is only a matching group. It works the same as a named parameter, except it will be numerically indexed.

```js
var re = pathToRegexp('/:foo/(.*)', keys)
// keys = [{ name: 'foo', ... }, { name: '0', ... }]

re.exec('/test/route')
//=> ['/test/route', 'test', 'route']
```

### Parse

The parse function is exposed via `pathToRegexp.parse`. This will yield an array of strings and keys.

```js
var tokens = pathToRegexp.parse('/route/:foo/(.*)')

console.log(tokens[0])
//=> "/route"

console.log(tokens[1])
//=> { name: 'foo', prefix: '/', delimiter: '/', optional: false, repeat: false, pattern: '[^\\/]+?' }

console.log(tokens[2])
//=> { name: 0, prefix: '/', delimiter: '/', optional: false, repeat: false, pattern: '.*' }
```

**Note:** This method only works with strings.

### Compile ("Reverse" Path-To-RegExp)

Path-To-RegExp exposes a compile function for transforming an express path into valid path. Confusing enough? This example will straighten everything out for you.

```js
var toPath = pathToRegexp.compile('/user/:id')

var result = toPath({ id: 123 })

console.log(result)
//=> "/user/123"
```

**Note:** The generated function will throw on any invalid input. It will execute all necessary checks to ensure the generated path is valid. This method only works with strings.

### Working with Tokens

Path-To-RegExp exposes the two functions used internally that accept an array of tokens.

* `pathToRegexp.tokensToRegExp(tokens, options)` Transform an array of tokens into a matching regular expression.
* `pathToRegexp.tokensToFunction(tokens)` Transform an array of tokens into a path generator function.

## Compatibility with Express <= 4.x

Path-To-RegExp breaks compatibility with Express <= 4.x in a few ways:

* RegExp special characters can now be used in the regular path. E.g. `/user/(\\d+)`
* All RegExp special characters can now be used inside the custom match. E.g. `/:user(.*)`
* No more support for asterisk matching - use an explicit parameter instead. E.g. `/(.*)`
* Parameters can have suffixes that augment meaning - `*`, `+` and `?`. E.g. `/:user*`
* Strings aren't interpreted as literal regexp strings - no more non-capturing groups, lookaheads, lookbehinds or nested matching groups (but you can still pass a regexp manually)

## Live Demo

You can see a live demo of this library in use at [express-route-tester](http://forbeslindesay.github.com/express-route-tester/).

## License

MIT

[npm-image]: https://img.shields.io/npm/v/path-to-regexp.svg?style=flat
[npm-url]: https://npmjs.org/package/path-to-regexp
[travis-image]: https://img.shields.io/travis/pillarjs/path-to-regexp.svg?style=flat
[travis-url]: https://travis-ci.org/pillarjs/path-to-regexp
[coveralls-image]: https://img.shields.io/coveralls/pillarjs/path-to-regexp.svg?style=flat
[coveralls-url]: https://coveralls.io/r/pillarjs/path-to-regexp?branch=master
[david-image]: http://img.shields.io/david/pillarjs/path-to-regexp.svg?style=flat
[david-url]: https://david-dm.org/pillarjs/path-to-regexp
[license-image]: http://img.shields.io/npm/l/path-to-regexp.svg?style=flat
[license-url]: LICENSE.md
[downloads-image]: http://img.shields.io/npm/dm/path-to-regexp.svg?style=flat
[downloads-url]: https://npmjs.org/package/path-to-regexp
