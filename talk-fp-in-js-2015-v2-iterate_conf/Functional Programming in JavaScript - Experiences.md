
# Functional Programming in JavaScript - Experiences

<p class="biggg">Jakub Holý - Iterate conference 8/2015</p>

## Survey:

* JavaScript devs?
* Lo-dash / Underscore / etc.?
* Immutable.js etc.?
* React?
* Functional programming?

# 1. FP: What & Why?

```javascript
var localIp = _.chain(os.networkInterfaces())
   .values()
   .flatten()
   .filter(_.partial(supportsProtocol, "IPv4"))
   .map("address")
   .first()
   .value();
```

### NOTES

This is the embodiment of FP to me - similar to the picture of a person with skies and a cup of coffee waiting for a metro is the symbol of Oslo for me, even though both contain much more than this.

<p style="font-size: 6rem">```
Program: output = f(input)
```</p>

 * Side-effects minimized and limited (<=> immutable data)
 * Focus on data transformations
 * (WHAT vs. HOW)
 
=> **SIMPLICITY**

<p style="transform: rotate(180deg); text-align: right;font-size: xx-large">⇧</p>

1. Easier to understand and test
   * and cheaper b/c `costs = f exp(complexity)`
2. Referential transparency enables optimizations
   * memoize, parallelization, ...
3. Laziness, ∞ => only do what's needed, simpler code
4. Concurrency
5. Expressivity: complex transformations in few LoC

#### NOTES

1. Easier thx to pure functions: no hidden behavior
2. 
3. Laziness - don't need to manually manage when to stop evaluating - e.g. can use an inf. sequence without needing to compute and pass around the desired length
4. Concurrency - with pure fn and immutability, no risk of bad interactions due to concurrency

Terms:

* Referential transparency = fn can be replaced with its value without changing the behavior <> must be pure
* Pure fn - no side-effects, depends only on its inputs
* Lazy evaluation - eval sub-expressions not where called but only when the result is actually needed

### What to expect from an FP lib

1. Functions: HOF, composing (comp), deriving (partial), running (iterate) fns
2. Data: Powerful fns for transformation of data structures

# 2. FP library: lodash-fp


    var lodash = require('lodash');
    var _ = require('lodash-fp');




    undefined



## What you get

1. Functions: `chain, partial(Right), curry, spread, memoize, ...`
2. Data: `map, filter, reject, reduce, groupBy, mapValues, zip, ...`
3. Utilities for objects, sets, strings; `method, is*, ...`

## Lodash vs. lodash-fp

Data last, curryied =>


    lodash.filter([1,2,3,4], function(num) { return _.gte(3, num); })




    [ 3, 4 ]




    _.filter( _.gte(3) )
            ( [1,2,3,4] )




    [ 3, 4 ]




    // Find obj with variations ∋ key "key1"
    (lodash.flow(
         _.partialRight(_.result, "variations")  // -> {key1: "val1"}
       , _.keys                                  // -> [key1]
       , _.partialRight(_.includes, "key1")      // -> true 
    ))
    ({variations: {key1: "val1"}})




    true



<p style="transform: rotate(180deg); text-align: right;font-size: xx-large">⇧</p>


    (_.flow(
      _.result("variations"),
      _.keys,
      _.includes("key1")))
    ({variations: {key1: "val1"}})




    true



### SKIP: What we miss in lodash(-fp)

* Our own predicates other than `==`: `filter({age: _.partialRight(_.gt,18)}) // _.flow(_.result('age'),_.gt(18))`
* Immutable data to avoid have ugly nested `forEach` (`_.map` is cumbersome)
* Walk the tree and replace `{fields: obj, ..}` with `obj`
* `partition-with`, nested `map` (`employees.children`)
* Lookup (combine data from 2+ sources)
* Challenge: Split into rows, each of width 12: `[{width:12},{width:6},{width:6},.]`

## The dark side of lodash-fp

* No vararg fns, some split (uniq + uniqBy) => lost functionality
* => You need `lodash` & `lodash-fp` (size?!).

See [github/jakubholynet/lodash-fp_issues/.../lodash-fp_issues.ipynb](https://github.com/jakubholynet/lodash-fp_issues/blob/master/lodash-fp_issues.ipynb).

#### NOTES

Due to auto-currying, lodash-fp cannot have vararg functions like lodash has. Some of the existing vararg fns are split into multiple (uniq => uniq + uniqBy) but not all and you thus lose access some functionality. 

In my experience you need ot keep lodash for the cases where the func isn't available in lodash-fp.
Not sure if including both increases the client-side size / how much.

Examples:

* `_.assign({}, {a:1}, {b:2})` is wrong - max 2
* `_.result` currently doesn't support a default value
* `zipObject([key, val, key2, val2, ..])` doesn't work, only `zipObject(keys, vals)`
* Iteratees such as `map(Values)`, `find` get only value, not the key argument
* Max does not accept the property argument: `_nofp.max([{n: 3},{n: 4}], \"n\") => { n: 4 }`

# 3. Immutable data

## Contestants

Immutable.js - Icepick - seamless-immutable

|                    | Size     | Maturity  |  API  | Plain JS?|
|--------------------|:--------:|:---------:|:-----:|:--------:|
| Immutable.js       |   56 kB  |    ★★★  | ★★★ |          |
| seamless-immutable |   3 kB   |     ★★   |  ★☆ |     ★    |
| icepick            |   3 kB   |     ★    |  ★★  |     ★   |

// `React.addon.update`

* Immutable.js: Facebook, used a lot with React (inspired by ClojureScript's React wrapper Om being faster thant React thanks to immutability). Very mature, lot of functionality, good quality, good performance.
* seamless - maturity: few people from one company, a number of releases. The API gives you all necessary but I find it somewhat lacking.
* icepick: one-man show, only a few releases; better API

### Immutable.js

```javascript
Immutable([1,2,3,4,5])
  .skip(2)
  .map(n => -n)
  .filter(n => n % 2 === 0)
  .take(2).reduce((r, n) => r * n, 1);
```

* New data structures and API (`map.get("prop")`)
* ES3 / ES6
* lazy Seq, Set, Map, Ordered(Set|Map), Range, Record
* push, set, unshift,..; getIn, setIn, updateIn; `withMutations`
* Interop.: `toJS()`

## seamless-immutable

```javascript
Immutable([1,2,3]).concat([4]).map(double).filter(odd);
```

* Plain JS + `Object.freeze` (deep); ES5 (IE9+)
* Modifies existing methods (map, filter, ..) => worse interop
* Mutating methods throw an `ImmutableError` (`push`, `=`, etc.)
  <> map, filter, concat
* Minimalistic API: A: `flatMap, asObject, asMutable`; O: `merge, without, asMutable`
* Prod x dev build (freeze, exceptions)
* **∅ <del>updateIn, setIn</del>**


* The only way to change an array is use map (to change a single value, combine with if), filter to remove, concat to add
* The prod build does neither freeze the data nor does it throw ImmutableError exceptions, the only thing it does is clone the data and add its methods
* I really miss the ability to change/set values deep in a nested structure, something like `updateIn(["employee" "salary"], (salary) => salary*1.05, employee)`. With just the available methods it is cumbersome and requires boilerplate code.

## icepick

```javascript
i.reverse(i.push(i.freeze([1,2,3], 4))
```

* Plain JS + `Object.freeze` (deep)
* A library of functions
* Richer API than s-i: assoc*, dissoc, get*, update*; push, splice, ..
* assign, merge,
* filter, map
* **∅ <del>reduce, map etc. over objects; chaining</del>**
* **one-man show**


* functions for changing particular elements, even in a nested data structure
* like s-immut., no map/reduce/.. over objects; no way as yet to chain the calls so you get a few nested `i.*` calls

### Immutable data conclusions

1. Use Immutable.js if you don't mind the size and don't need lot of interop (`to/fromJS`)
2. Use seamless-immutable + lodash-fp and write own convenience fn `updateIn` etc.
3. Or use icepick + lodash-fp if adventuresome or really need a particular [nested] element change (and write your `chain`)

## Immutable data & performance

Random benchmarks

* Immutable.js: Mutable ~900 ms | Immutable.js ×3 | I+shouldComponentUpdate ×0.7
* Seamless-immutable: Mutable: **TODO** | prod 800 ms | dev ×2-3

*Beware: The only benchmark that counts is yours, in your prod env.*

# 4. Conclusion 1: FP possible but painful; worth it?

1. Libraries quite OK but rough edges
   (other than `lodash-fp`?)
2. Immutable data: sub-par and/or large size & inconveniences

<table><tr><td>
You can make JavaScript fly...
<img src="flying-car.jpg" style="width: 500;height: 300px"/>
</td><td>
But why not to use something *designed* for flying?
<img src="plane-futuristic.jpg" style="width: 500;height: 300px"/>
</td></tr></table>


# Conclusion 2: Don't hesitate to try ClojureScript

Why? Top-notch data functions and immutable data, core-async, macros, great design.

What about the bytes?



| JS: Nettbutikk | Cljs: Startshop|
|--------------------|--------|
| 300 kB      |   300 kB |
| React, router, lodash,.. |   Om, React, .. |
| No immutable, ... | Immutable, core-async, ..|
| (little own code) | (little own code) |

What about debugging? REPL! (And I live w/o debugger in JS anyway)


ClojureScript gives you

* One of the best libraries of functions for working with data. I am still discovering and amazed how practical and composable they are and there are still hidden gems in the standard lib that could replace functions I write manually.
* One of the best, very performant implementations of (persistent) immutable data structures.
* Many goodies: core-async for simple async programming (especially useful in the single-threaded JavaScript that in addition depends on the user and REST calls), pattern matching, macros - the [A-bomb of programming](http://www.paulgraham.com/avg.html)

Size: Not really a problem. These two similar web apps, whose size is dominated by their libraries (including the large React), have the same size, even though one uses ClojureScript (and thus has things the other misses) - thanks to the very aggressive Google Closure compiler that can throw away all you actually don't use and the fact that Cljs is optimized for it.

Debugging: Clojure(Script) developers typically don't need it. Interactive, REPL-based development satisfies/prevents most of what debugging does otherwise.

Last: Developers are surprisingly capable of and quick to learn a new language. At least that is the experience of various teams that moved to Clojure.

# Resources

* [The Power and Practicalities of Immutability](https://vimeo.com/131635253) - Venkat Subramaniam at NDC Oslo 2015
* [Immutable JavaScript](https://vimeo.com/128790457) - Christian Johansen at Web Rebels 2015 (Immutable.js & Minesweeper)
* [React: Rethinking best practices](https://www.youtube.com/watch?v=DgVS-zXgMTk) - Pete Hunt at JSConf 2013

Extra

* [ClojureScript and the Blub Paradox](https://wildermuthn.github.io/2015/08/04/clojurescript-and-the-blub-paradox) - and interesting comparison of Cljs and JS
