# JavaScript: Interview Study Guide

All 49 JavaScript topics from the study list, explained with short examples and the traps interviewers like to set.

## Part 1: Language Core

### 1. Variables (var, let, const)

`var` is function-scoped, hoisted and initialized to `undefined`, and can be redeclared. `let` and `const` are block-scoped and hoisted but sit in the **temporal dead zone** until their declaration runs. `const` forbids reassignment, not mutation: `const a = []; a.push(1)` is fine. Default to `const`, use `let` when you must reassign, avoid `var`.

### 2. Data Types and Type Coercion

Primitives: `string`, `number`, `bigint`, `boolean`, `undefined`, `null`, `symbol`. Everything else is an object (arrays and functions included). `typeof null === "object"` is a historical bug. Coercion converts types implicitly: `"5" + 1 === "51"` but `"5" - 1 === 4`. `NaN !== NaN`; use `Number.isNaN`.

### 3. Functions and Arrow Functions

Functions can be declarations (hoisted), expressions, or arrows. Arrow functions are shorter, have **no own `this`, `arguments` or `prototype`**, cannot be used with `new`, and take `this` from the enclosing scope. Use them for callbacks, not for object methods that need their own `this`.

### 4. Scope and Hoisting

Scope decides where a variable is visible: global, function, block, and module. The engine resolves names up the **scope chain** (lexical scoping, decided by where code is written). Hoisting moves declarations to the top of their scope: function declarations are fully hoisted; `var` is hoisted as `undefined`; `let`/`const` throw a ReferenceError if used early.

### 5. Closures

A function that remembers variables from the scope where it was created, even after that scope has finished.

```js
function counter() { let n = 0; return () => ++n; }
const c = counter(); c(); c(); // 2
```

Uses: private state, factories, memoization, callbacks. Risk: retaining large objects causes memory leaks.

### 6. this Keyword

`this` depends on **how a function is called**: method call (`obj.fn()`) gives `obj`; plain call gives `undefined` in strict mode (global object otherwise); `new` gives the new object; `call/apply/bind` set it explicitly; arrow functions inherit it lexically. Losing `this` when passing `obj.method` as a callback is the classic bug.

### 7. Prototypes and Inheritance

Every object has an internal link (`[[Prototype]]`) to another object. Property lookup walks this **prototype chain** until found or `null`. `Object.create(proto)` sets it directly; `Object.getPrototypeOf` reads it. Methods placed on the prototype are shared across instances, saving memory.

### 8. Classes

Syntactic sugar over prototypes (ES6): `class A { constructor(){} method(){} static s(){} }`, `extends`, `super`, getters/setters, and private fields with `#x`. Classes are not hoisted like functions and must be called with `new`. Under the hood they still use prototype inheritance.

### 9. Arrays and Array Methods (map, filter, reduce)

`map` transforms each element into a new array; `filter` keeps elements passing a test; `reduce` folds an array into one value: `[1,2,3].reduce((a, b) => a + b, 0)`. Also know `forEach`, `find`, `some`, `every`, `flat`, `flatMap`, `includes`, `sort` (mutates, and sorts as strings by default) and `slice` vs `splice` (non-mutating vs mutating).

### 10. Objects and Destructuring

Objects are key-value collections. **Destructuring** unpacks values: `const { name, age: years = 0 } = user;` and `const [a, b] = arr;`. Supports defaults, renaming, nesting and use in function parameters. Shorthand properties and computed keys (`{[key]: v}`) are related features.

### 11. Spread and Rest Operators

Both use `...`. **Spread** expands: `[...a, ...b]`, `{...obj, x: 1}`, `fn(...args)`. **Rest** collects: `function f(...args)` or `const [first, ...others] = arr`. Spread makes *shallow* copies only.

## Part 2: Asynchronous JavaScript

### 12. Callbacks

Functions passed to other functions to run later. They enable async work but nested callbacks create "callback hell" and tangled error handling (the Node convention is error-first: `(err, data)`). Promises and async/await largely replaced them.

### 13. Promises

An object representing a future value in one of three states: pending, fulfilled or rejected, settled once. Use `.then()`, `.catch()`, `.finally()`; chains return new promises. `new Promise((resolve, reject) => {...})` creates one. Always handle rejections.

### 14. Async/Await

Syntax over promises: an `async` function always returns a promise; `await` pauses *that function* until the promise settles without blocking the thread. Use `try/catch` for errors. Pitfall: sequential `await` in a loop is slow; use `Promise.all` for independent tasks.

### 15. Event Loop

JavaScript is single-threaded. The **call stack** runs code; async work (timers, I/O) is handled by the environment and its callbacks are queued. When the stack is empty, the loop first drains the **microtask queue** (promises), then takes one task from the **macrotask queue** (timers, events). This explains why `setTimeout(fn, 0)` runs after promise callbacks.

### 16. DOM Manipulation

The DOM is the browser's tree of the page. Select with `querySelector(All)`, change with `textContent`, `classList`, `setAttribute`, `style`, create with `createElement` and `append`. Prefer `textContent` over `innerHTML` for untrusted input (XSS). Batch updates to avoid layout thrashing.

### 17. Event Handling and Bubbling

`el.addEventListener("click", handler)`. Events travel in three phases: **capture** (down from window), **target**, and **bubble** (back up). `stopPropagation()` halts propagation; `preventDefault()` cancels default behavior (like form submit). `{ once: true }` and `{ passive: true }` are useful options.

### 18. ES Modules

`export` and `import` give file-level scope and explicit dependencies: `export default`, named exports, `import { x } from "./a.js"`. Imports are static (analyzable, enabling tree shaking), live bindings, strict mode by default. Dynamic `import()` returns a promise for lazy loading.

### 19. Error Handling

Use `try/catch/finally`, throw `Error` objects (not strings), and create custom error classes. `catch` handles synchronous errors and awaited rejections but **not** errors inside un-awaited callbacks. Global safety nets: `window.onerror`, `unhandledrejection`.

### 20. Fetch API and JSON

`fetch(url, options)` returns a promise of a `Response`. It **does not reject on HTTP errors** (404/500): check `res.ok`. Parse with `await res.json()`. `JSON.stringify` and `JSON.parse` convert objects to text and back (functions, `undefined` and circular references are not serializable).

### 21. == vs === and Truthy/Falsy values

`===` compares without coercion; `==` coerces types (`0 == ""` is true). Use `===`. Falsy values: `false, 0, -0, 0n, "", null, undefined, NaN`. Everything else is truthy, including `"0"`, `[]` and `{}`.

### 22. Template Literals

Backtick strings with interpolation and multi-line support: `` `Hi ${name}, ${a + b}` ``. **Tagged templates** (`tag`\`...\`\`) pass the pieces to a function, used by libraries like styled-components and for safe SQL/HTML building.

## Part 3: Functions in Depth

### 23. Higher-Order Functions

Functions that take functions as arguments or return them (`map`, `filter`, `setTimeout`, decorators). They work because functions are first-class values. They enable abstraction and composition.

### 24. call, apply and bind

All set `this` explicitly. `fn.call(obj, a, b)` invokes immediately with arguments listed; `fn.apply(obj, [a, b])` with an array; `fn.bind(obj, a)` returns a **new function** with `this` (and optionally arguments) fixed. A bound function's `this` cannot be changed again.

### 25. Currying and Function Composition

**Currying** turns `f(a, b, c)` into `f(a)(b)(c)`: `const add = a => b => a + b`. It enables partial application. **Composition** chains functions: `const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x)`.

### 26. Debouncing and Throttling

**Debounce** waits until calls stop for N ms, then runs once (search-as-you-type). **Throttle** runs at most once per N ms (scroll, resize).

```js
function debounce(fn, ms) {
  let t;
  return (...args) => { clearTimeout(t); t = setTimeout(() => fn(...args), ms); };
}
```

### 27. Memoization

Caching a function's results by its arguments to skip repeat computation. Only valid for pure functions.

```js
const memo = fn => { const c = new Map(); return n => c.has(n) ? c.get(n) : (c.set(n, fn(n)), c.get(n)); };
```

### 28. Event Delegation and Capturing

Attach one listener to a parent and use `event.target` (or `closest`) to handle children, relying on bubbling. Saves memory, and works for elements added later. Capturing (`{ capture: true }`) intercepts the event on the way down, before the target.

### 29. Microtasks vs Macrotasks

**Microtasks** (promise callbacks, `queueMicrotask`, `MutationObserver`) run right after the current script, before rendering and before the next macrotask. **Macrotasks** (`setTimeout`, `setInterval`, I/O, UI events) run one per loop turn. Output order: sync code, then microtasks, then macrotasks.

### 30. Promise.all, race, allSettled, any

`all`: resolves with all results, rejects on the first rejection. `race`: settles with the first settled promise. `allSettled`: waits for all and reports each outcome, never rejects. `any`: resolves with the first *fulfilled*; rejects (AggregateError) only if all reject.

## Part 4: Modern Features and Data Structures

### 31. Generators and Iterators

An **iterator** has `next()` returning `{ value, done }`; iterables implement `[Symbol.iterator]`. A **generator** (`function*` with `yield`) produces values lazily and can pause and resume, useful for infinite sequences and custom iteration. `for...of` consumes iterables.

### 32. Map, Set, WeakMap, WeakSet

`Map` stores key-value pairs with **any type of key** and preserves insertion order. `Set` stores unique values. `WeakMap`/`WeakSet` hold objects *weakly* (keys must be objects, are garbage-collectable, are not iterable); ideal for private metadata without leaks.

### 33. Symbols

Unique, immutable primitive values, used as non-colliding property keys: `const id = Symbol("id")`. Well-known symbols (`Symbol.iterator`, `Symbol.toPrimitive`) customize language behavior. `Symbol.for(key)` shares via a global registry.

### 34. Optional Chaining and Nullish Coalescing

`a?.b?.[0]?.()` returns `undefined` instead of throwing if something in the chain is `null`/`undefined`. `x ?? default` uses the default only for `null` or `undefined`, whereas `||` also replaces `0`, `""` and `false`.

### 35. Shallow vs Deep Copy (structuredClone)

A **shallow** copy (`{...o}`, `Object.assign`, `slice`) duplicates only the top level; nested objects stay shared. A **deep** copy duplicates everything: `structuredClone(obj)` (handles dates, maps, sets, cycles; not functions) is now preferred over `JSON.parse(JSON.stringify(x))`, which drops many types.

### 36. Immutability (Object.freeze)

Immutable data is never modified in place; you create updated copies. `Object.freeze` blocks add/change/delete at **one level only** (shallow) and silently fails outside strict mode. Immutability makes change detection and reasoning easier (core to React state).

### 37. new keyword and Constructor Functions

`new F()` creates an empty object, links its prototype to `F.prototype`, runs `F` with `this` as the new object, and returns it (unless the function returns another object). Constructors are conventionally capitalized. Forgetting `new` is a classic bug that classes prevent.

### 38. Closures in loops (classic interview trap)

```js
for (var i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 3 3 3
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i)); // 0 1 2
```

`var` is one shared binding, so all callbacks see the final value; `let` creates a fresh binding per iteration. The older fix is an IIFE capturing `i`.

### 39. CommonJS vs ES Modules

CommonJS (`require`/`module.exports`) is synchronous, dynamic and Node's original system. ES Modules (`import`/`export`) are static, asynchronous, standard in browsers and enable tree shaking. In Node, enable ESM via `.mjs` or `"type": "module"`. ESM has no `__dirname` (use `import.meta.url`).

### 40. Garbage Collection and Memory Leaks

JS uses automatic **mark-and-sweep**: objects unreachable from roots are freed. Leaks come from accidental globals, forgotten timers and listeners, detached DOM nodes, growing caches and closures retaining big data. Diagnose with browser heap snapshots; fix with cleanup, `WeakMap` and `AbortController`.

## Part 5: Browser and Platform

### 41. Web Storage (localStorage, sessionStorage), Cookies

`localStorage` persists until cleared; `sessionStorage` lasts for the tab. Both are synchronous string key-value stores of about 5 MB, same origin, readable by scripts (avoid secrets). **Cookies** are sent with every request, have expiry and flags: `HttpOnly` (no JS access), `Secure`, `SameSite`. Use `HttpOnly` cookies for session tokens.

### 42. CORS

Browsers block cross-origin reads unless the server opts in via headers like `Access-Control-Allow-Origin`. "Non-simple" requests trigger a **preflight** `OPTIONS` request. CORS is enforced by the browser, not the server, and does not protect your API from other clients. Never use `*` with credentials.

### 43. Regular Expressions

Pattern matching: `/^\d{3}-\d{4}$/.test(s)`, `str.match`, `matchAll`, `replace`. Know classes (`\d \w \s`), quantifiers (`* + ? {n,m}`), groups and named groups, lookahead `(?=...)`, and flags `g i m s u y`. Beware catastrophic backtracking.

### 44. Date and Intl APIs

`Date` stores milliseconds since the Unix epoch, has zero-indexed months and awkward time-zone handling (the Temporal API is the modern replacement). `Intl.DateTimeFormat`, `Intl.NumberFormat` and `Intl.RelativeTimeFormat` handle locale-aware formatting. Store and transmit dates in UTC ISO 8601.

### 45. Polyfills (write your own map, bind, Promise)

A polyfill implements a missing feature in older environments, and writing one is a common interview exercise.

```js
Array.prototype.myMap = function (cb, thisArg) {
  const out = [];
  for (let i = 0; i < this.length; i++)
    if (i in this) out[i] = cb.call(thisArg, this[i], i, this);
  return out;
};
```

For `bind`, return a function that calls the original with fixed `this` and merged args; for `Promise`, track state, a value and a queue of handlers.

### 46. Strict Mode

`"use strict"` (automatic in modules and classes) turns silent mistakes into errors: undeclared variables throw, `this` is `undefined` in plain calls, duplicate parameters and writing to read-only properties are errors, and `with` is banned.

### 47. Service Workers and Web Workers

**Web Workers** run scripts on a background thread for heavy computation, communicating via `postMessage` (no DOM access). **Service Workers** are a network proxy between page and server: they enable offline support, caching strategies, push notifications and background sync; they require HTTPS and have their own lifecycle (install, activate, fetch).

### 48. Node.js basics (modules, fs, npm, event emitter)

Node runs JS outside the browser on the V8 engine with a non-blocking, event-driven I/O model. Core modules: `fs` (use `fs/promises`), `path`, `http`, `os`, `events`. **npm** manages dependencies via `package.json` and `package-lock.json`. `EventEmitter` implements the observer pattern: `on`, `emit`, `once`. Know streams and that CPU-heavy work blocks the event loop.

### 49. TypeScript basics

A typed superset of JavaScript that compiles to plain JS and catches errors at build time. Know type annotations, `interface` vs `type`, unions and intersections, generics, `unknown` vs `any`, optional and readonly properties, enums, type narrowing, utility types (`Partial`, `Pick`, `Omit`, `Record`) and `strict` mode. Types are erased at runtime.

---

## Practice Prompts

1. Predict the output of a snippet mixing `setTimeout`, a promise and sync logs.
2. Implement `debounce`, `throttle`, `memoize` and `Promise.all` from scratch.
3. Explain why `[10, 9, 1].sort()` gives `[1, 10, 9]`.
4. Fix a closure-in-loop bug three different ways.
5. Explain `this` in five different call styles.
