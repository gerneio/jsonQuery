# jsonQuery
This is a fork of 'jsonQuery' from https://github.com/harpreetkhalsagtbit/jsonQuery which was copied and updated for browser/node support from Dojox.json from https://github.com/maqetta/dojox/blob/master/json/query.js

## Install

No npm of this repo just yet

# How to Use?
Although the code has been modified from Dojox.json, it's usability is still the same as detailed here: https://dojotoolkit.org/reference-guide/1.10/dojox/json/query.html

My fork simple adds additional features and usage enhancements as detailed below.

**Example:**

Please look to the original repo and Dojox reference document for typical detailed usage. I will cover functionality that my fork has improved on or added below. **NOTE: My testing environment is Chrome with the latest V8 engine. My production environment is ClearScript which also uses the V8 engine. My main goal is to support my environment and use cases.**

Using this dataset:
```js
// require/import jsonQuery.js

var data = {
  grouped_people: {
    'friends': [
      {name: 'Steve', country: 'N Z', age: 1},
      {name: 'Jane', country: 'US', age: 2},
      {name: 'Mike', country: 'AU', age: 3},
      {name: 'Mary', country: 'N Z', age: 4},
    ],
    'enemies': [
      {name: 'Evil Steve', country: 'AU', age: 5},
      {name: 'Betty', country: 'N Z', age: 6}
    ]
  }
};
```

Evalutating the following queries:
```js
// Basic query
var res = jsonQuery.query("$..name", data);
console.log(res); // ["Steve", "Jane", "Mike", "Mary", "Evil Steve", "Betty"]

// Simplified
jsonQuery.setObjProto(); // Only need to call once
var res = data.query("$..name");
console.log(res); // ["Steve", "Jane", "Mike", "Mary", "Evil Steve", "Betty"]

// Further simplified
var pdata = jsonQuery.proxyIndexer(data); // (ES6 ONLY) Called once per object that it needs to be used on; must reference the produced proxied object
var res = pdata["$..name"];
console.log(res); // ["Steve", "Jane", "Mike", "Mary", "Evil Steve", "Betty"]

// Lambda and regular function support
var res = pdata["$..[?country].map(e => e.country)"];
console.log(res); // ["N Z", "US", "AU", "N Z", "AU", "N Z"]

// Prototyping/overloading support
Array.prototype.min = function() { return Math.min.apply(_, this); }
Array.prototype.max = function() { return Math.max.apply(_, this); }

var min = pdata["$..age.min()"];
var max = pdata["$..age.max()"];
console.log(min, max); // 1 6

// Lambda & Prototyping with LINQJS
var _EnumerableImpl = Enumerable.From([]).constructor; // LINQJS

Object.setPrototypeOf(Array.prototype, _EnumerableImpl.prototype);

_EnumerableImpl.prototype.AverageA = function(a) { return this.Select(a || (e => e)).Average(e => parseInt(e, 10) || null) };
_EnumerableImpl.prototype.SumA = function(a) { return this.Select(a || (e => e)).Sum(e => parseInt(e, 10) || null) };

var avgAge = pdata["$..age.AverageA()"]);
var sumAge = pdata["$..age.SumA()"]);
var ordNms = pdata["$..[?country='N Z'].OrderBy(e => e.age).ToArray()[=name]"];
console.log(avgAge, sumAge, ordNms); // 3.5 21 ["Steve", "Mary", "Betty"]
```


# Changelog Notes
Commit: https://github.com/gerneio/jsonQuery/commit/28dce7e90efb3d0a79c4c7ed4b9cf096352d7890

Implemented a rudimentary logging system, specifically an array (queue) for the following: jsonQuery._log.func & jsonQuery._log.query. In the future need to add amt to keep or implement a tokenized session retrieval for the logs and/or add a parameter to enable console logging per jsonQuery.query() call. In the mean time the logs can be cleared on demand with jsonQuery._log.resetLogs().

Implemented full support for ES6 lambdas as well as normal functions. I.E. Array.map(e => e.prop) or Array.map(function(e) { return e.prop }). Everything between the parenthesis will not be further interpreted, so valid JS code is expected. This was implemented specifically for the usage in combination with prototyped functions from LINQJS (I.E. https://jsfiddle.net/x7g0zf19/1/): obj["$..prop.Sum()"].

Cleaned up arguments usage. Previously a hard limit of 9 args could be passed and used in the format '$#' where # was a digit between 1 and 9. Now you can pass any amount of arguments (for whatever reason). The function definition is generated dynamically based on the # of arguments passed to the query function.

Added support for automatically setting Object.prototype.query() to be bound to automatically calling jsonQuery.query() via jsonQuery.setObjProto() (only needs called once per JS instance), so that the query method can easily be called on any object with little code: obj.query("$..prop"). This is taken to the next level with jsonQuery.proxyIndexer() (ES6 only) which implements a proxy over an object. All the proxy does is monitor/track the usage of any property GET checking to see IF the requested property is not present AND the character '$' is present at the beginning of the property string, then it will perform that jsonQuery.query() for you using the specified property name: obj["$..query"]. This is the simplest way to implement and effectively use the jsonQuery library, since the whole point is to use as little syntax as possible to get the desired result, yet still have the ability to do complex operations, which can still be pretty simple.

Lots of code cleanup, spacing and a few rewrites for ES6 syntax. Ideally the whole thing will be converted to ES6 sugar-syntax since it is much more readable and typically cleaner. A transpiler can be used to convert for ES5 or older compatibility (proxyIndexer() only supported on ES6).

# Todo

* Remove $obj reference and use only $.
* Further code cleanup and transpilation to ES6
* Investigate possibility of parental references
* Attempt to implement escape characters for keywords and/or to prevent entire sequences of code from being processed by the regex rules (similar to how strings and lambdas/funcs are currently handled)
* Implement a regex.replace engine/system that can be put into a modularized sequence for clearer visibility & code readability.

# Goals

As stated above, the goals of this repo are to add feature and usage enhancements inline with my use cases and environment (ClearScript & V8). These are my primary focuses:

* Simplicity: Make it easy to call jsonQuery.query() from any object or scenario.
* Capability: Ensure that even some of the most complicated tasks are possible
* One liners: Need I say more?
* Customizability & control: Implement handlers, such as security control, which is already loosely implemented, as well as other configurable options such as logging, context, and prototyping.
* ES6 Support: Implement any feature that makes it easier to write one liner
