# JavaScript Interview Q&A — Final Phase

> Q201–Q310 | Execution Context, JS Internals, Advanced Patterns, Performance, Output Questions

---

## Q201. What is an Execution Context?

An Execution Context (EC) is the **environment in which JavaScript code is evaluated and executed**. It contains all the information needed to run a piece of code: variables, functions, scope chain, and the value of `this`.

### Two types

| Type | Created when |
|------|-------------|
| **Global Execution Context (GEC)** | Once, when the script first loads |
| **Function Execution Context (FEC)** | Every time a function is invoked |

### Two phases of every EC

**Phase 1 — Creation Phase** (before any code runs):
- Variable declarations (`var`) are hoisted and initialized to `undefined`
- `let`/`const` declarations are hoisted into the **Temporal Dead Zone** (inaccessible)
- Function declarations are fully stored in memory (name + body)
- `this` binding is determined
- Outer environment reference (scope chain) is set up

**Phase 2 — Execution Phase** (code runs line by line):
- Variable assignments happen
- Function calls push new ECs onto the Call Stack

```js
// What the engine sees during Creation Phase:
var a;              // undefined
function greet() { console.log("Hello"); }

// Execution Phase:
console.log(a);    // undefined — var was hoisted
a = 10;            // assignment happens now
greet();           // executes using stored function
```

**Key insight:** Hoisting is not a physical movement of code — it's the visible effect of the Creation Phase running before the Execution Phase.

---

## Q202. What is the Call Stack?

The Call Stack is a **LIFO (Last In, First Out)** data structure that tracks the currently executing Execution Contexts.

```
Function call  → push new EC onto stack
Function return → pop EC off stack
```

```js
function c() { console.log("c"); }
function b() { c(); }
function a() { b(); }
a();

// Call Stack progression:
// [GEC]
// [GEC, a()]         a() called
// [GEC, a(), b()]    b() called inside a()
// [GEC, a(), b(), c()] c() called inside b()
// [GEC, a(), b()]    c() returns
// [GEC, a()]         b() returns
// [GEC]              a() returns
```

### Stack overflow
```js
function infinite() { infinite(); } // no base case
infinite(); // RangeError: Maximum call stack size exceeded
```

Each function call adds a frame. Without a return, the stack grows until memory exhaustion.

---

## Q203. What is the Scope Chain and Lexical Environment?

Every Execution Context has a **Lexical Environment** consisting of:
1. **Environment Record** — the variable bindings in this scope
2. **Outer reference** — pointer to the parent Lexical Environment

The chain of outer references from inner → outer → global is the **Scope Chain**.

```js
const x = "global";

function outer() {
  const x = "outer";

  function inner() {
    const x = "inner";
    console.log(x); // "inner" — found in own scope
  }

  function middle() {
    // x not in middle → walks up to outer
    console.log(x); // "outer"
  }

  inner();
  middle();
}

outer();
console.log(x); // "global"
```

```
inner() Lexical Env  → outer() Lexical Env  → Global Lexical Env  → null
  { x: "inner" }        { x: "outer" }          { x: "global" }
```

Variable lookup always goes **inward → outward**. An outer scope cannot see inner scope variables.

---

## Q204. What is Lexical Scope?

Lexical scope means the scope of a variable is determined by **where it is written in source code**, not where the function is called from.

```js
const lang = "JavaScript";

function outer() {
  const framework = "React";

  function inner() {
    // Can access both — defined inside outer which is inside global
    console.log(lang);      // "JavaScript"
    console.log(framework); // "React"
  }

  return inner;
}

const fn = outer();
fn(); // called from global — still accesses "React" via lexical scope
// This is a closure — inner remembers its lexical environment
```

**Lexical vs Dynamic scope:**
- JavaScript uses **lexical** — scope fixed at write time
- Some languages (bash, older Lisps) use dynamic — scope determined at call time

---

## Q205. Global vs Function vs Block Scope?

```js
// Global — accessible everywhere
var globalVar = "global";
let globalLet = "also global";

function example() {
  // Function scope — accessible only inside this function
  var funcVar = "function";
  let funcLet = "also function";

  if (true) {
    // Block scope — only inside this {}
    var blockVar = "leaks out!"; // var ignores blocks
    let blockLet = "stays here";
    const blockConst = "stays here too";

    console.log(blockLet);   // ✅ accessible
  }

  console.log(blockVar);  // ✅ "leaks out!" — var escaped the block
  console.log(blockLet);  // ❌ ReferenceError — let is block-scoped
}
```

**Module scope (ESM):** Variables declared at the top level of a module are module-scoped — not global. They can only leave the module via `export`.

---

## Q206. What is Nested Function Scope?

```js
const country = "India";

function outer() {
  const state = "Maharashtra";

  function inner() {
    const city = "Pune";
    // Can access: city (own), state (parent), country (grandparent)
    console.log(`${city}, ${state}, ${country}`); // "Pune, Maharashtra, India"
  }

  // outer cannot access city — it's below in the chain
  console.log(typeof city); // "undefined"
  inner();
}
```

**Scope chain lookup order:** current scope → immediate parent → ... → global → throws `ReferenceError`

---

## Q207. `call()`, `apply()`, `bind()` — Deep Dive?

> Core covered in JSQA.md Q88 and phase3.md Q88.

### Classic output question
```js
const person = { name: "Alice" };

function greet(greeting, punct) {
  return `${greeting}, ${this.name}${punct}`;
}

greet.call(person, "Hello", "!");       // "Hello, Alice!"
greet.apply(person, ["Hello", "!"]);    // "Hello, Alice!"
const greetAlice = greet.bind(person);
greetAlice("Hi", ".");                  // "Hi, Alice."

// Partial application with bind
const sayHello = greet.bind(person, "Hello");
sayHello("!");  // "Hello, Alice!"
sayHello("?");  // "Hello, Alice?"
```

### Why arrow functions ignore bind/call/apply
```js
const obj = { val: 42 };
const arrow = () => this.val;

arrow.call(obj);   // undefined — lexical this, cannot be overridden
arrow.apply(obj);  // undefined
arrow.bind(obj)(); // undefined
```

### Borrowing methods
```js
// Borrow Array methods for array-like objects
const args = { 0: "a", 1: "b", length: 2 };
Array.prototype.join.call(args, "-"); // "a-b"
Array.prototype.map.call(args, x => x.toUpperCase()); // ["A","B"]
```

---

## Q208. Event Bubbling vs Event Capturing?

Events travel through the DOM in two phases:

```
CAPTURING (top → bottom)    BUBBLING (bottom → top)
   window                      window
     ↓                           ↑
   document                   document
     ↓                           ↑
   <html>                     <html>
     ↓                           ↑
   <body>                     <body>
     ↓                           ↑
   <div>      ←── target ───→  <div>
     ↓                           ↑
  <button>  ← event fires here  <button>
```

```js
// Default — bubble phase (useCapture = false)
parent.addEventListener("click", () => console.log("parent bubble"));
child.addEventListener("click",  () => console.log("child bubble"));
// Click child → logs: "child bubble" then "parent bubble"

// Capture phase (useCapture = true or { capture: true })
parent.addEventListener("click", () => console.log("parent capture"), true);
child.addEventListener("click",  () => console.log("child capture"),  true);
// Click child → logs: "parent capture" then "child capture"

// Stop propagation
child.addEventListener("click", e => {
  e.stopPropagation();  // stops bubbling (and capturing)
  // e.stopImmediatePropagation() — also stops other handlers on same element
});
```

---

## Q209. What is Event Delegation?

Attaching **one listener to a parent** instead of many listeners to each child. Works because of bubbling.

```js
// ❌ Without delegation — N listeners for N items
document.querySelectorAll(".delete-btn").forEach(btn => {
  btn.addEventListener("click", handleDelete);
});
// Problem: new items added dynamically won't have listeners

// ✅ With delegation — one listener, works for current and future elements
document.getElementById("list").addEventListener("click", e => {
  const btn = e.target.closest(".delete-btn");
  if (btn) {
    const id = btn.dataset.id;
    deleteItem(id);
  }
});
```

### Why `closest()` instead of just `e.target`
If a button contains an `<svg>` icon, clicking the icon makes `e.target` the `<svg>`, not the button. `closest()` walks up until it finds the matching selector.

**Performance benefit:** 1 listener vs 1000 listeners — significant for large lists, virtual scrolling, dynamic tables.

---

## Q210. Primitive vs Reference Types?

> Covered in JSQA.md Q21 and phase4.md Q132. Key output question:

```js
// Pass by value — primitives
function double(n) { n = n * 2; }
let x = 5;
double(x);
console.log(x); // 5 — function got a copy

// Pass by reference — objects (actually "pass by value of the reference")
function addRole(user) {
  user.role = "admin"; // modifies the same object
}
function replaceUser(user) {
  user = { name: "Bob" }; // reassigns local variable, original unchanged
}

const alice = { name: "Alice" };
addRole(alice);
console.log(alice.role); // "admin" — mutated via reference

replaceUser(alice);
console.log(alice.name); // "Alice" — original reference untouched
```

---

## Q211. `undefined` vs `not defined`?

```js
// undefined — variable declared, no value assigned
var a;
console.log(a);           // undefined
console.log(typeof a);    // "undefined"

// not defined — variable never declared in any accessible scope
console.log(b);           // ReferenceError: b is not defined
console.log(typeof b);    // "undefined" — typeof is safe for undeclared variables!

// The typeof safety trick
if (typeof process !== "undefined") {
  // safe to use process — useful for code that runs in both browser and Node
}
```

---

## Q212. What is an IIFE?

An **Immediately Invoked Function Expression** executes as soon as it's defined.

```js
// Classic IIFE syntax
(function() {
  const private = "not accessible outside";
  console.log("Runs immediately");
})();

// Arrow IIFE
(() => {
  console.log("Also runs immediately");
})();

// IIFE with parameters
(function(global) {
  global.myLib = {};
})(window);

// Why IIFEs existed — create private scope before ES modules
// Modern equivalent: just use a module (import/export)

// Still useful: top-level await in non-module contexts
(async () => {
  const data = await fetchData();
  render(data);
})();
```

---

## Q213. `Object.freeze()` vs `Object.seal()`?

```js
const obj = { name: "Alice", age: 30 };

// Object.seal() — no add/delete, but CAN modify existing
Object.seal(obj);
obj.name = "Bob";   // ✅ allowed — modifying existing
obj.email = "...";  // ❌ silently fails (throws in strict mode)
delete obj.age;     // ❌ silently fails

// Object.freeze() — no add/delete/modify
Object.freeze(obj);
obj.name = "Carol"; // ❌ silently fails (throws in strict mode)

// Check status
Object.isSealed(obj);  // true
Object.isFrozen(obj);  // true
```

| | `seal()` | `freeze()` |
|---|---|---|
| Add property | ❌ | ❌ |
| Delete property | ❌ | ❌ |
| Modify existing | ✅ | ❌ |
| More restrictive | No | Yes |

**Important:** Both are **shallow** — nested objects are not frozen/sealed.
```js
const obj = Object.freeze({ nested: { x: 1 } });
obj.nested.x = 99; // ✅ works — nested object is not frozen
```

---

## Q214. `Object.create()` vs `Object.assign()`?

```js
// Object.create(proto) — creates new object with proto as prototype
const animal = { breathe() { return "breathing"; } };
const dog = Object.create(animal);
dog.name = "Rex";
dog.breathe(); // found on prototype chain

// Object.assign(target, ...sources) — copies own enumerable properties
const base = { a: 1, b: 2 };
const extra = { b: 3, c: 4 };
const merged = Object.assign({}, base, extra); // { a:1, b:3, c:4 }
// Note: later sources overwrite earlier ones (b:2 → b:3)

// Object.assign is a SHALLOW copy
const obj = { nested: { x: 1 } };
const copy = Object.assign({}, obj);
copy.nested.x = 99;
console.log(obj.nested.x); // 99 — shared reference!
```

| | `Object.create()` | `Object.assign()` |
|---|---|---|
| Purpose | Prototypal inheritance | Shallow copy / merge |
| Sets prototype | ✅ | ❌ |
| Copies properties | ❌ | ✅ |

---

## Q215. What are Higher-Order Functions?

> Covered in phase3.md Q80. Additional patterns:

```js
// Compose — combine functions right-to-left
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

// Pipe — combine functions left-to-right
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const trim      = s => s.trim();
const lower     = s => s.toLowerCase();
const noSpaces  = s => s.replace(/\s+/g, "-");

const slugify = pipe(trim, lower, noSpaces);
slugify("  Hello World  "); // "hello-world"
```

---

## Q216. Pure vs Impure Functions?

> Covered in phase3.md Q86. Output question addition:

```js
// Is this pure?
let calls = 0;
function count() { return ++calls; }
// ❌ Impure — modifies external state, different output each call

// Is this pure?
function formatDate(date) { return date.toLocaleDateString(); }
// ❌ Impure — output varies by locale/timezone of the machine

// Is this pure?
const add = (a, b) => a + b;
// ✅ Pure — same inputs always give same output, no side effects
```

---

## Q217. What is Functional Programming in JavaScript?

Core principles applied in real JavaScript:

```js
// 1. Avoid mutation — transform, don't modify
const doubled = nums.map(n => n * 2);    // new array
const filtered = users.filter(u => u.active); // new array

// 2. Pure functions — no side effects
const calculateTotal = (items, tax) =>
  items.reduce((sum, item) => sum + item.price, 0) * (1 + tax);

// 3. Function composition
const processUsers = pipe(
  users => users.filter(u => u.active),
  users => users.map(u => ({ ...u, name: u.name.trim() })),
  users => users.sort((a, b) => a.name.localeCompare(b.name))
);

// 4. Immutability
const updateUser = (users, id, updates) =>
  users.map(u => u.id === id ? { ...u, ...updates } : u);
```

**React is functional:** Components are functions, `useState` avoids mutation, `useReducer` uses pure reducer functions, hooks compose behavior.

---

## Q218. What is Currying?

> Covered in phase3.md Q87 and JSQA.md Q87. Interview implementation:

```js
// Generic curry
const curry = fn => {
  const arity = fn.length;
  return function curried(...args) {
    return args.length >= arity
      ? fn(...args)
      : (...more) => curried(...args, ...more);
  };
};

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);
curriedAdd(1)(2)(3);   // 6
curriedAdd(1, 2)(3);   // 6
curriedAdd(1)(2, 3);   // 6

// Real-world: configurable validators
const validate = curry((rule, errorMsg, value) =>
  rule(value) ? null : errorMsg
);
const required = validate(v => v !== "", "Field is required");
const minLen5  = validate(v => v.length >= 5, "Too short");

required("");      // "Field is required"
required("Alice"); // null
minLen5("Hi");     // "Too short"
```

---

## Q219. What is Memoization?

> Covered in JSQA.md Q219. Implementation with WeakMap for objects:

```js
// Basic memoize (works for primitive args)
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const fib = memoize(function(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

fib(40); // fast — each value computed once

// React equivalent
const expensiveValue = useMemo(() => computeHeavy(data), [data]);
const stableCallback = useCallback(() => handleClick(id), [id]);
```

---

## Q220. What is Recursion?

```js
// Every recursive function needs:
// 1. Base case — stops recursion
// 2. Recursive case — makes progress toward base case

function factorial(n) {
  if (n <= 1) return 1;        // base case
  return n * factorial(n - 1); // recursive case
}

// Flatten deeply nested array (real interview question)
function flatten(arr) {
  return arr.reduce((flat, item) =>
    Array.isArray(item)
      ? [...flat, ...flatten(item)]  // recurse
      : [...flat, item],             // base case
  []);
}
flatten([1, [2, [3, [4]]]]); // [1, 2, 3, 4]

// Deep clone (recursion over object tree)
function deepClone(val) {
  if (val === null || typeof val !== "object") return val;
  if (val instanceof Date)  return new Date(val);
  if (Array.isArray(val))   return val.map(deepClone);
  return Object.fromEntries(
    Object.entries(val).map(([k, v]) => [k, deepClone(v)])
  );
}
```

**Stack overflow risk:** Deep recursion (thousands of levels) exhausts the call stack. Solve with iteration + an explicit stack, or trampolining for functional style.

---

## Q221. What is Debouncing?

> Covered in JSQA.md Q221. Production-grade implementation:

```js
function debounce(fn, delay, { leading = false } = {}) {
  let timer;
  return function(...args) {
    const callNow = leading && !timer;
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      if (!leading) fn.apply(this, args);
    }, delay);
    if (callNow) fn.apply(this, args);
  };
}

// Usage
const searchUsers = debounce(async (query) => {
  const results = await api.search(query);
  setResults(results);
}, 300);

input.addEventListener("input", e => searchUsers(e.target.value));

// React hook version
function useDebounce(value, delay) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debounced;
}
```

---

## Q222. What is Throttling?

```js
function throttle(fn, interval) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= interval) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  const scrolled = window.scrollY / document.body.scrollHeight;
  setProgress(Math.round(scrolled * 100));
}, 100); // max 10 calls per second

window.addEventListener("scroll", handleScroll);
```

---

## Q223. Debounce vs Throttle?

| | Debounce | Throttle |
|---|---|---|
| Executes | After inactivity ends | At fixed time intervals |
| Execution frequency | Once per burst | Regularly during continuous event |
| Best for | Search input, form validation, resize | Scroll tracking, mouse move, game loop |
| Example | Search API call | Scroll progress bar |

```
User types: a → ab → abc → abcd (pause)
Debounce:                            ↑ one call after pause
Throttle:   ↑          ↑          ↑  calls at fixed intervals
```

---

## Q224. Deep Copy vs Shallow Copy?

> Covered in JSQA.md Q21 and phase4.md Q132–Q133. Comprehensive methods:

```js
const original = {
  name: "Alice",
  address: { city: "Pune" },
  skills: ["JS", "React"],
  dob: new Date("1990-01-01"),
};

// Shallow — nested objects still shared
const s1 = { ...original };
const s2 = Object.assign({}, original);

// Deep — structuredClone (modern, handles Date/Map/Set/circular)
const d1 = structuredClone(original);
d1.address.city = "Mumbai";
original.address.city; // "Pune" — independent

// JSON method — fast but lossy
const d2 = JSON.parse(JSON.stringify(original));
// ❌ Loses: Date→string, undefined, Symbol, functions, Map, Set
// ❌ Throws on circular references

// What structuredClone handles that JSON doesn't:
structuredClone(new Map([["a", 1]]));  // ✅
structuredClone(new Set([1, 2, 3]));  // ✅
structuredClone(new Date());          // ✅ keeps as Date
structuredClone(undefined);           // ✅
```

---

## Q225. Polyfill for `Array.prototype.map()`?

```js
Array.prototype.myMap = function(callback, thisArg) {
  if (typeof callback !== "function") {
    throw new TypeError(callback + " is not a function");
  }
  const result = new Array(this.length);
  for (let i = 0; i < this.length; i++) {
    if (i in this) { // handles sparse arrays
      result[i] = callback.call(thisArg, this[i], i, this);
    }
  }
  return result;
};

[1, 2, 3].myMap(n => n * 2); // [2, 4, 6]
```

---

## Q226. Polyfill for `Array.prototype.filter()`?

```js
Array.prototype.myFilter = function(callback, thisArg) {
  if (typeof callback !== "function") throw new TypeError(callback + " is not a function");
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (i in this && callback.call(thisArg, this[i], i, this)) {
      result.push(this[i]);
    }
  }
  return result;
};

[1, 2, 3, 4].myFilter(n => n % 2 === 0); // [2, 4]
```

---

## Q227. Polyfill for `Array.prototype.reduce()`?

```js
Array.prototype.myReduce = function(callback, initialValue) {
  if (typeof callback !== "function") throw new TypeError(callback + " is not a function");

  let acc, startIndex;

  if (arguments.length >= 2) {
    acc = initialValue;
    startIndex = 0;
  } else {
    if (this.length === 0) throw new TypeError("Reduce of empty array with no initial value");
    acc = this[0];
    startIndex = 1;
  }

  for (let i = startIndex; i < this.length; i++) {
    if (i in this) acc = callback(acc, this[i], i, this);
  }
  return acc;
};

[1, 2, 3, 4].myReduce((sum, n) => sum + n, 0); // 10
```

---

## Q228. Flatten a Nested Array?

```js
// Built-in (cleanest)
[1, [2, [3, [4]]]].flat(Infinity);   // [1, 2, 3, 4]
[1, [2, 3]].flat();                  // [1, 2, 3] — one level by default

// Recursive implementation (interview version)
function flatten(arr, depth = Infinity) {
  return arr.reduce((flat, item) => {
    if (Array.isArray(item) && depth > 0) {
      return [...flat, ...flatten(item, depth - 1)];
    }
    return [...flat, item];
  }, []);
}

// Iterative (avoids stack overflow for very deep arrays)
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];
  while (stack.length) {
    const item = stack.pop();
    if (Array.isArray(item)) {
      stack.push(...item);
    } else {
      result.unshift(item); // unshift to maintain order (since we use pop)
    }
  }
  return result;
}
```

---

## Q229. Flatten a Nested Object?

```js
// Input:  { a: 1, b: { c: 2, d: { e: 3 } } }
// Output: { "a": 1, "b.c": 2, "b.d.e": 3 }

function flattenObject(obj, prefix = "", result = {}) {
  for (const [key, val] of Object.entries(obj)) {
    const newKey = prefix ? `${prefix}.${key}` : key;
    if (val !== null && typeof val === "object" && !Array.isArray(val)) {
      flattenObject(val, newKey, result); // recurse into nested objects
    } else {
      result[newKey] = val;
    }
  }
  return result;
}

flattenObject({ a: 1, b: { c: 2, d: { e: 3 } } });
// { "a": 1, "b.c": 2, "b.d.e": 3 }

// Reverse — unflatten
function unflattenObject(flat) {
  return Object.entries(flat).reduce((obj, [key, val]) => {
    key.split(".").reduce((curr, part, i, parts) => {
      curr[part] = i === parts.length - 1 ? val : (curr[part] || {});
      return curr[part];
    }, obj);
    return obj;
  }, {});
}
```

---

## Q230. Implement Memoization?

> See Q219. Additional pattern — memoize with cache size limit (LRU):

```js
function memoizeWithLimit(fn, limit = 100) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      // Move to end (most recently used)
      const val = cache.get(key);
      cache.delete(key);
      cache.set(key, val);
      return val;
    }
    const result = fn.apply(this, args);
    if (cache.size >= limit) {
      // Delete least recently used (first entry)
      cache.delete(cache.keys().next().value);
    }
    cache.set(key, result);
    return result;
  };
}
```

---

## Q231. Implement Currying?

> See Q218. Already covered with generic implementation.

---

## Q232. Simple Promise Polyfill?

```js
class MyPromise {
  #state = "pending";
  #value;
  #callbacks = [];

  constructor(executor) {
    const resolve = value => {
      if (this.#state !== "pending") return;
      this.#state = "fulfilled";
      this.#value = value;
      this.#callbacks.forEach(cb => cb.onFulfilled?.(value));
    };
    const reject = reason => {
      if (this.#state !== "pending") return;
      this.#state = "rejected";
      this.#value = reason;
      this.#callbacks.forEach(cb => cb.onRejected?.(reason));
    };
    try { executor(resolve, reject); }
    catch (e) { reject(e); }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      const handle = (fn, settle) => value => {
        try {
          if (typeof fn === "function") resolve(fn(value));
          else settle(value);
        } catch(e) { reject(e); }
      };
      if (this.#state === "fulfilled") {
        queueMicrotask(() => handle(onFulfilled, resolve)(this.#value));
      } else if (this.#state === "rejected") {
        queueMicrotask(() => handle(onRejected, reject)(this.#value));
      } else {
        this.#callbacks.push({
          onFulfilled: handle(onFulfilled, resolve),
          onRejected:  handle(onRejected, reject),
        });
      }
    });
  }

  catch(onRejected) { return this.then(null, onRejected); }
}
```

---

## Q233. Common Promise Coding Questions?

```js
// 1. Sequential execution
async function sequential(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}

// 2. Parallel execution with concurrency limit
async function parallelLimit(tasks, limit) {
  const results = [];
  for (let i = 0; i < tasks.length; i += limit) {
    const batch = tasks.slice(i, i + limit);
    const batchResults = await Promise.all(batch.map(t => t()));
    results.push(...batchResults);
  }
  return results;
}

// 3. Retry with exponential backoff
async function retryWithBackoff(fn, maxRetries = 3, baseDelay = 1000) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (err) {
      if (attempt === maxRetries) throw err;
      const delay = baseDelay * Math.pow(2, attempt);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// 4. Delay / sleep utility
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
await sleep(1000); // pause 1 second

// 5. Promise timeout
function withTimeout(promise, ms) {
  return Promise.race([
    promise,
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error(`Timeout: ${ms}ms`)), ms)
    )
  ]);
}
```

---

## Q234. Fetch API Deep Dive?

```js
// Fetch doesn't throw on 4xx/5xx — check res.ok!
async function safeFetch(url, options = {}) {
  const res = await fetch(url, {
    headers: { "Content-Type": "application/json" },
    ...options,
  });
  if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);
  return res.json();
}

// POST
await safeFetch("/users", {
  method: "POST",
  body: JSON.stringify({ name: "Alice" }),
});

// With auth
await safeFetch("/protected", {
  headers: {
    "Authorization": `Bearer ${token}`,
    "Content-Type": "application/json",
  },
});

// Cancellable with AbortController (React cleanup)
useEffect(() => {
  const controller = new AbortController();
  safeFetch(`/users/${id}`, { signal: controller.signal })
    .then(setUser)
    .catch(err => {
      if (err.name !== "AbortError") setError(err.message);
    });
  return () => controller.abort();
}, [id]);

// Fetch vs Axios
// fetch: native, verbose error handling, manual JSON parsing, no interceptors
// axios: library, auto JSON, interceptors, request cancellation, better defaults
```

---

## Q235. What is REST API?

```
REST = Representational State Transfer

Core constraints:
1. Stateless    — each request contains all info needed
2. Resource-based URLs — /users/123, not /getUser?id=123
3. HTTP methods carry semantic meaning
4. Uniform interface
```

```http
GET    /users          → list all users
GET    /users/123      → get user 123
POST   /users          → create a user
PUT    /users/123      → replace user 123 (full body required)
PATCH  /users/123      → update user 123 (partial update)
DELETE /users/123      → delete user 123
```

```js
// HTTP status codes to know
200 OK          // success
201 Created     // POST success
204 No Content  // DELETE success
400 Bad Request // invalid input from client
401 Unauthorized // not authenticated
403 Forbidden   // authenticated but not authorized
404 Not Found   // resource doesn't exist
409 Conflict    // duplicate / state conflict
422 Unprocessable // validation failed
500 Server Error // backend crashed
```

---

## Q236. Secure API Calls?

```js
// 1. Always HTTPS
// 2. Send token in Authorization header (not URL)
fetch("/api/data", {
  headers: { Authorization: `Bearer ${accessToken}` }
});

// 3. Never hardcode secrets in frontend code
// Use environment variables: process.env.REACT_APP_API_KEY

// 4. Validate responses before using
const data = await res.json();
if (!data || !Array.isArray(data.users)) throw new Error("Unexpected response shape");

// 5. Handle token expiry — refresh pattern
async function apiCall(url) {
  let res = await fetch(url, { headers: authHeader() });
  if (res.status === 401) {
    await refreshToken();
    res = await fetch(url, { headers: authHeader() });
  }
  return res.json();
}

// 6. CSRF protection (server-side concern — use SameSite cookies, CSRF tokens)
// 7. Input sanitization — never build queries by concatenating user input
```

---

## Q237. Input Validation and Security?

```js
// XSS — never set innerHTML with user data
el.innerHTML = userInput;  // ❌
el.textContent = userInput; // ✅

// Sanitize if HTML rendering is needed
import DOMPurify from "dompurify";
el.innerHTML = DOMPurify.sanitize(userInput);

// Client-side validation — improve UX, never trust for security
function validateForm(data) {
  const errors = {};
  if (!data.email?.includes("@")) errors.email = "Invalid email";
  if (data.password?.length < 8)  errors.password = "Too short";
  return errors;
}
// Always validate again on the backend — frontend can be bypassed

// Content Security Policy (CSP) — server header
// "Content-Security-Policy: default-src 'self'"
// Prevents loading scripts from unknown origins — best XSS defence
```

---

## Q238. What are Web Workers?

```js
// worker.js — runs in a separate thread, no DOM access
self.onmessage = function(e) {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};

// main.js — communicate via postMessage
const worker = new Worker("worker.js");
worker.postMessage({ data: largeDataset });
worker.onmessage = e => console.log("Result:", e.data);
worker.onerror   = e => console.error("Worker error:", e.message);

// Terminate when done
worker.terminate();

// Workers CANNOT access:
// document, window, DOM APIs
// But CAN access:
// fetch, setTimeout, WebSockets, IndexedDB, crypto

// Use cases: heavy calculations, image processing, CSV parsing, crypto mining
```

---

## Q239. Memory Management and Garbage Collection?

```js
// GC uses Mark-and-Sweep:
// 1. Start from "roots" (global, call stack)
// 2. Mark all reachable objects
// 3. Sweep (free) unreachable objects

// Memory leak patterns:

// 1. Forgotten event listeners
const handler = () => updateUI();
el.addEventListener("click", handler);
// Later: el is removed from DOM but handler keeps el in memory
el.removeEventListener("click", handler); // fix

// 2. setInterval holding reference
const id = setInterval(() => bigObject.process(), 1000);
clearInterval(id); // fix when done

// 3. Closures holding large data
function leaky() {
  const huge = new Array(1_000_000).fill(0);
  return () => huge[0]; // huge cannot be GC'd
}

// 4. Detached DOM nodes
let detached = document.createElement("div");
document.body.appendChild(detached);
document.body.removeChild(detached);
// detached variable still holds reference — GC can't collect it
detached = null; // fix

// WeakRef and WeakMap allow GC to collect referenced objects
const cache = new WeakMap(); // keys GC'd when object has no other refs
```

---

## Q240. V8 Engine Deep Dive?

```
Source Code
    ↓
Parser  →  AST (Abstract Syntax Tree)
    ↓
Ignition (Interpreter)  →  Bytecode
    ↓ (hot paths)
TurboFan (JIT Compiler)  →  Optimized Machine Code
    ↓ (if type assumptions break)
Deoptimization  →  back to bytecode
```

```js
// V8 optimization tip — keep consistent types in hot functions
function add(a, b) { return a + b; }
add(1, 2);   // V8: "always numbers" → optimizes
add("x", 2); // "string + number" → type changed → deoptimization

// Hidden classes — V8 creates internal classes for objects
// Adding properties in the same order reuses hidden classes (faster)
function createUser(name, age) {
  const u = {};
  u.name = name; // same order always
  u.age  = age;  // → V8 reuses hidden class
  return u;
}

// Inline caches — V8 caches property lookup results
// Accessing the same property type repeatedly is optimized
```

---

## Q241. Microtasks vs Macrotasks — Full Reference?

> Covered in phase5.md Q182. Additional nuance:

```js
// Microtask queue is drained COMPLETELY before next macrotask
// New microtasks added during microtask processing are also processed

Promise.resolve()
  .then(() => {
    console.log("MT1");
    Promise.resolve().then(() => console.log("MT2")); // added during microtask
  });
setTimeout(() => console.log("MACRO"), 0);

// Output: MT1  MT2  MACRO
// MT2 is processed before MACRO even though MACRO was scheduled first
```

---

## Q242. Event Loop — Interview Output Questions?

```js
// Question 1 — Classic
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => console.log("C"));
console.log("D");
// A D C B

// Question 2 — Async/Await
async function fn() {
  console.log("1");
  await null;           // schedules rest as microtask
  console.log("2");
}
console.log("0");
fn();
console.log("3");
// 0 1 3 2

// Question 3 — Nested Promises (senior level)
Promise.resolve()
  .then(() => {
    console.log("P1");
    return Promise.resolve("P2"); // wrapping in Promise adds extra microtask tick
  })
  .then(v => console.log(v));

Promise.resolve().then(() => console.log("P3"));
// P1  P3  P2  ← P2 is delayed by the extra Promise wrapping
```

---

## Q243. Message Queue?

The **Macrotask Queue** (also called Message Queue or Task Queue) holds callbacks ready to execute after the call stack empties and all microtasks are drained.

Sources: `setTimeout`, `setInterval`, `setImmediate` (Node), DOM events, I/O.

The Event Loop picks **one macrotask per iteration**, then drains the full microtask queue before the next macrotask.

---

## Q244. Generators?

```js
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i;
  }
}

const gen = range(0, 10, 2);
gen.next(); // { value: 0, done: false }
gen.next(); // { value: 2, done: false }
[...range(0, 5)]; // [0, 1, 2, 3, 4]

// Two-way communication — yield also receives values
function* calculator() {
  let result = 0;
  while (true) {
    const input = yield result;
    result += input;
  }
}
const calc = calculator();
calc.next();    // { value: 0, done: false } — start
calc.next(5);   // { value: 5, done: false } — adds 5
calc.next(3);   // { value: 8, done: false } — adds 3

// Infinite sequence — memory efficient (lazy)
function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}
const fib = fibonacci();
Array.from({ length: 8 }, () => fib.next().value); // [0,1,1,2,3,5,8,13]
```

---

## Q245. Iterators?

```js
// Iterator protocol: object with next() returning { value, done }
function makeCounter(from, to) {
  let current = from;
  return {
    next() {
      return current <= to
        ? { value: current++, done: false }
        : { value: undefined, done: true };
    }
  };
}

const counter = makeCounter(1, 3);
counter.next(); // { value: 1, done: false }
counter.next(); // { value: 2, done: false }
counter.next(); // { value: 3, done: false }
counter.next(); // { value: undefined, done: true }

// Iterable protocol: object with [Symbol.iterator]() returning an iterator
const range = {
  from: 1, to: 5,
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    return {
      next() {
        return current <= last
          ? { value: current++, done: false }
          : { done: true };
      }
    };
  }
};

for (const n of range) console.log(n); // 1 2 3 4 5
[...range]; // [1, 2, 3, 4, 5]
```

---

## Q246. Symbols?

```js
// Every Symbol is unique
const s1 = Symbol("id");
const s2 = Symbol("id");
s1 === s2; // false

// Use as unique object keys — no naming collisions
const ID    = Symbol("id");
const user = { name: "Alice", [ID]: 42 };
user[ID];            // 42
Object.keys(user);   // ["name"] — Symbol not in normal enumeration

// Well-known Symbols — hooks into JS engine behavior
const range = {
  [Symbol.iterator]() { /* makes object iterable */ }
};

// Symbol.toPrimitive — customize type conversion
const money = {
  amount: 100,
  currency: "USD",
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.amount;
    if (hint === "string") return `${this.amount} ${this.currency}`;
    return this.amount;
  }
};
+money;         // 100
`${money}`;     // "100 USD"
money + 50;     // 150
```

---

## Q247. Optional Chaining (`?.`)?

> Covered in JSQA.md Q25. Output questions:

```js
const user = { address: null };

user?.address?.city;    // undefined (no error)
user?.getName?.();      // undefined (safe method call)
user?.skills?.[0];      // undefined (safe array access)

// Short-circuits — stops at first null/undefined
const x = null;
x?.y?.z?.w;             // undefined — doesn't throw

// Does NOT guard against undefined properties (only null/undefined access)
const obj = {};
obj.nonExistent.deep;   // ❌ TypeError — nonExistent is undefined
obj?.nonExistent?.deep; // ✅ undefined
```

---

## Q248. Nullish Coalescing (`??`)?

> Covered in JSQA.md Q25 and Q27. Output questions:

```js
// ?? vs ||
0     ?? "default"  // 0       — 0 is not null/undefined
""    ?? "default"  // ""      — empty string is not null/undefined
false ?? "default"  // false   — false is not null/undefined
null  ?? "default"  // "default"
undefined ?? "default" // "default"

0     || "default"  // "default" — 0 is falsy
""    || "default"  // "default" — "" is falsy
false || "default"  // "default" — false is falsy

// Chaining with ?.
const city = user?.address?.city ?? "Unknown";
const count = response?.data?.length ?? 0;
```

---

## Q249–Q253. Promise Methods (see phase5.md Q171–Q175)?

These are fully covered in phase5.md. Quick decision guide recap:

```
Need ALL results, fail if any fails       → Promise.all
Need ALL results, never fail              → Promise.allSettled
Need FIRST result (success or failure)    → Promise.race (timeouts)
Need FIRST SUCCESS, ignore failures       → Promise.any (CDN fallback)
```

---

## Q254. Execute Promises Sequentially?

```js
// With for...of (sequential, each awaited before next)
async function sequential(promiseFns) {
  const results = [];
  for (const fn of promiseFns) {
    results.push(await fn()); // waits for each
  }
  return results;
}

// With reduce (functional style)
const sequential2 = promiseFns =>
  promiseFns.reduce(
    (chain, fn) => chain.then(async results => [...results, await fn()]),
    Promise.resolve([])
  );
```

---

## Q255. Cancelable Promises (AbortController)?

```js
// Fetch with cancel
function fetchWithCancel(url) {
  const controller = new AbortController();
  const promise = fetch(url, { signal: controller.signal })
    .then(r => r.json());
  return { promise, cancel: () => controller.abort() };
}

const { promise, cancel } = fetchWithCancel("/large-data");
setTimeout(cancel, 5000); // cancel after 5s if not done

// React pattern — cancel on unmount or dependency change
useEffect(() => {
  const controller = new AbortController();
  fetch(`/users/${id}`, { signal: controller.signal })
    .then(r => r.json())
    .then(setUser)
    .catch(e => { if (e.name !== "AbortError") setError(e.message); });
  return () => controller.abort();
}, [id]);
```

---

## Q256. `setTimeout` vs `setInterval`?

> Covered in phase5.md Q164–Q165. Prefer recursive `setTimeout` for polling:

```js
// setInterval — can overlap if callback takes longer than interval
// setTimeout recursive — next starts only AFTER current finishes
async function pollStatus() {
  const status = await checkStatus();
  updateUI(status);
  if (status !== "DONE") {
    setTimeout(pollStatus, 2000); // 2s after last call completes
  }
}
```

---

## Q257–Q260. Browser Caching, Storage Comparison, IndexedDB, JSON.stringify?

> Covered in phase5.md Q186–Q192 and Q200. Additional:

```js
// JSON.stringify gotchas
JSON.stringify(undefined);          // undefined (not a string)
JSON.stringify({ a: undefined });   // "{}" — key dropped
JSON.stringify([undefined]);        // "[null]" — undefined → null in arrays
JSON.stringify(NaN);                // "null"
JSON.stringify(Infinity);           // "null"
JSON.stringify(new Date());         // '"2024-01-01T00:00:00.000Z"' — string
JSON.stringify(new Map([["a",1]])); // "{}" — Map not serialized

// Replacer function
JSON.stringify(obj, (key, val) => {
  if (typeof val === "function") return undefined; // skip functions
  return val;
});

// Space parameter for pretty print
JSON.stringify(obj, null, 2);
```

---

## Q261. JSON Deep Dive?

```js
// JSON rules
// Keys MUST be double-quoted strings
// No trailing commas
// No comments
// No undefined, functions, Symbol, Infinity, NaN (→ null)
// Dates → string

// Reviver — transform values when parsing
const user = JSON.parse(jsonStr, (key, val) => {
  if (key === "dob") return new Date(val); // restore Date
  return val;
});

// Circular reference detection
const a = { b: {} };
a.b.a = a; // circular!
JSON.stringify(a); // ❌ TypeError: Converting circular structure to JSON

// Fix with replacer that tracks seen objects
function safeStringify(obj) {
  const seen = new WeakSet();
  return JSON.stringify(obj, (key, val) => {
    if (typeof val === "object" && val !== null) {
      if (seen.has(val)) return "[Circular]";
      seen.add(val);
    }
    return val;
  });
}
```

---

## Q262. Module Pattern?

> Covered in phase5.md Q199 and JSQA.md Q31.

```js
// Modern equivalent using ES modules — no IIFE needed
// utils.js
let _privateCounter = 0;

export function increment() { _privateCounter++; }
export function getCount() { return _privateCounter; }
// _privateCounter is module-private — not exported
```

---

## Q263. Singleton Pattern?

```js
// ES Module singleton — import caching makes this automatic
// config.js — every importer gets the SAME instance
let _instance = null;

export function getInstance() {
  if (!_instance) {
    _instance = {
      theme: "light",
      language: "en",
      update(key, val) { this[key] = val; }
    };
  }
  return _instance;
}

// Class singleton
class AppStore {
  static #instance = null;
  #state = {};

  static getInstance() {
    AppStore.#instance ??= new AppStore();
    return AppStore.#instance;
  }

  get(key)      { return this.#state[key]; }
  set(key, val) { this.#state[key] = val; }
}
```

---

## Q264. Observer Pattern?

```js
class EventEmitter {
  #listeners = new Map();

  on(event, listener) {
    if (!this.#listeners.has(event)) this.#listeners.set(event, []);
    this.#listeners.get(event).push(listener);
    return () => this.off(event, listener); // returns unsubscribe fn
  }

  off(event, listener) {
    const list = this.#listeners.get(event);
    if (list) this.#listeners.set(event, list.filter(l => l !== listener));
  }

  emit(event, ...args) {
    this.#listeners.get(event)?.forEach(l => l(...args));
  }

  once(event, listener) {
    const wrapper = (...args) => { listener(...args); this.off(event, wrapper); };
    return this.on(event, wrapper);
  }
}

const bus = new EventEmitter();
const unsub = bus.on("user:login", user => console.log("Logged in:", user.name));
bus.emit("user:login", { name: "Alice" }); // "Logged in: Alice"
unsub(); // clean up
```

---

## Q265. Code Splitting and Q266. Lazy Loading?

> Covered in phase5.md Q199 and Q200.

```js
// React lazy + Suspense — route-level code splitting
const Dashboard = React.lazy(() => import("./pages/Dashboard"));
const Admin     = React.lazy(() => import("./pages/Admin"));

function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/admin"     element={<Admin />} />
      </Routes>
    </Suspense>
  );
}
// Webpack/Vite splits each lazy import into a separate bundle
// Downloaded only when the route is first visited
```

---

## Q267. CSR vs SSR?

| | CSR (Client-Side Rendering) | SSR (Server-Side Rendering) |
|---|---|---|
| Initial HTML | Empty shell + JS bundle | Full HTML from server |
| First paint | Slower (wait for JS) | Faster |
| SEO | Poor (crawlers see empty page) | Excellent |
| Interactivity | Excellent after hydration | Excellent after hydration |
| Server cost | Low | Higher |
| Examples | React SPA, Vite | Next.js, Angular Universal |

```
CSR flow: Browser → Download JS → Execute → Render DOM
SSR flow: Browser → Request → Server renders HTML → Browser shows HTML → Hydration (JS makes it interactive)

Hydration = attaching React's event system to the server-rendered HTML
```

**Hybrid approaches:** Next.js supports SSR, SSG (Static Site Generation), and ISR (Incremental Static Regeneration) per route.

---

## Q268. Redux Toolkit?

```js
// createSlice — combines actions + reducer, uses Immer under the hood
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

const fetchUsers = createAsyncThunk("users/fetch", async () => {
  const res = await fetch("/api/users");
  return res.json();
});

const usersSlice = createSlice({
  name: "users",
  initialState: { data: [], status: "idle", error: null },
  reducers: {
    addUser(state, action) {
      state.data.push(action.payload); // looks like mutation — Immer makes it safe
    },
    removeUser(state, action) {
      state.data = state.data.filter(u => u.id !== action.payload);
    },
  },
  extraReducers(builder) {
    builder
      .addCase(fetchUsers.pending,   state => { state.status = "loading"; })
      .addCase(fetchUsers.fulfilled, (state, { payload }) => {
        state.status = "succeeded";
        state.data = payload;
      })
      .addCase(fetchUsers.rejected,  (state, action) => {
        state.status = "failed";
        state.error = action.error.message;
      });
  },
});

export const { addUser, removeUser } = usersSlice.actions;
export default usersSlice.reducer;
```

---

## Q269. What is Babel?

Babel is a **JavaScript transpiler** (source-to-source compiler) that converts modern JS syntax into older syntax for browser compatibility.

```
ES2022+ source  →  Babel  →  ES5 compatible code

const add = (a, b) => a + b;
↓
var add = function(a, b) { return a + b; };
```

**Babel vs bundler distinction:**
- Babel **transpiles** syntax (arrow functions, classes, optional chaining)
- Webpack/Vite **bundles** modules (resolve imports, combine files)
- They work together: Vite uses esbuild for fast transpilation in dev, Rollup + Babel/SWC for production

---

## Q270. Polyfills vs Transpilers?

```
Transpiler (Babel):     converts SYNTAX
  arrow functions, classes, optional chaining, destructuring

Polyfill (core-js):    adds missing RUNTIME APIs
  Promise, Array.from, Object.assign, Array.prototype.flat, globalThis

Both needed:
  Syntax:   const fn = () => {}  →  Babel handles
  Runtime:  Promise.allSettled   →  Polyfill needed (not syntax, it's an API)
```

---

## Q284. Weird JavaScript Output Questions?

```js
// Type coercion puzzles
typeof null         // "object"   ← historic bug
typeof []           // "object"
typeof NaN          // "number"   ← NaN is a number type
typeof class C {}   // "function"

[] + []             // ""         ← both → ""
[] + {}             // "[object Object]"
{} + []             // 0          ← {} parsed as empty block, +[] = 0
({}) + []           // "[object Object]" ← now it's an expression

NaN === NaN         // false      ← use Number.isNaN()
null == undefined   // true       ← special rule
null === undefined  // false

"5" + 3             // "53"       ← string concatenation
"5" - 3             // 2          ← numeric coercion
"5" * "3"           // 15
true + true         // 2          ← booleans → numbers
[] == false         // true        ← [] → "" → 0, false → 0
[] == ![]           // true        ← ![] = false, [] == false = true
0.1 + 0.2 === 0.3  // false       ← floating point imprecision
(0.1 + 0.2).toFixed(1) === "0.3" // true
```

---

## Q285. `this` Output Questions?

```js
// 1 — method call
const obj = { x: 1, fn() { return this.x; } };
obj.fn(); // 1

// 2 — detached method
const fn = obj.fn;
fn(); // undefined (strict) or window.x (non-strict)

// 3 — arrow in object literal
const obj2 = { x: 2, fn: () => this?.x };
obj2.fn(); // undefined — lexical this (module/window)

// 4 — arrow inside regular method
const obj3 = {
  x: 3,
  outer() {
    const inner = () => this.x; // inherits outer's this = obj3
    return inner();
  }
};
obj3.outer(); // 3

// 5 — class method lost
class Counter {
  count = 0;
  increment() { this.count++; }
}
const c = new Counter();
const inc = c.increment;
inc(); // ❌ TypeError — this is undefined in strict mode
// Fix: bind in constructor or use arrow method
class Counter2 {
  count = 0;
  increment = () => { this.count++; }; // arrow — always bound
}
```

---

## Q286. Hoisting Output Questions?

```js
// 1 — var
console.log(a); // undefined (hoisted, not initialized)
var a = 10;
console.log(a); // 10

// 2 — let TDZ
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 5;

// 3 — function declaration
greet(); // "Hello" — fully hoisted
function greet() { console.log("Hello"); }

// 4 — function expression
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi"); };
// var sayHi = undefined at the time of the call

// 5 — shadowing in function
var x = "global";
function test() {
  console.log(x); // undefined — local var x is hoisted
  var x = "local";
  console.log(x); // "local"
}
test();

// 6 — class hoisting
const c = new MyClass(); // ❌ ReferenceError — classes are in TDZ
class MyClass {}
```

---

## Q287. Event Loop Output Questions?

> Covered in Q242 above. Additional senior-level questions:

```js
// Q1 — Promise inside setTimeout
setTimeout(() => {
  Promise.resolve().then(() => console.log("P"));
  console.log("T");
}, 0);
// T  P  (T is sync inside macrotask, P is microtask after T)

// Q2 — Multiple setTimeout
setTimeout(() => console.log("1"), 0);
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
// 3  1  2  (microtask first, then macrotasks in order)

// Q3 — async/await order
async function foo() {
  console.log("A");
  await Promise.resolve();
  console.log("B");
  await Promise.resolve();
  console.log("C");
}
foo();
console.log("D");
// A  D  B  C
// Each await suspends and schedules rest as microtask
```

---

## Q288. Closure Output Questions?

```js
// 1 — counter
function make() {
  let n = 0;
  return () => ++n;
}
const inc = make();
console.log(inc()); // 1
console.log(inc()); // 2
console.log(inc()); // 3

// 2 — var loop (classic)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 3 3 3
}

// 3 — let loop (fix)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i)); // 0 1 2
}

// 4 — closure capture
function outer() {
  let x = 10;
  function inner() { x++; return x; }
  return inner;
}
const fn = outer();
fn(); // 11
fn(); // 12
fn(); // 13

// 5 — multiple closures sharing scope
function makeAdder() {
  let sum = 0;
  return {
    add: n => (sum += n),
    get: () => sum,
  };
}
const adder = makeAdder();
adder.add(5); adder.add(3);
adder.get(); // 8 — both methods share the same sum
```

---

## Q289. Promise Output Questions?

```js
// 1 — executor is synchronous
console.log("A");
new Promise(resolve => {
  console.log("B"); // sync
  resolve();
}).then(() => console.log("C")); // microtask
console.log("D");
// A B D C

// 2 — chaining order
Promise.resolve()
  .then(() => { console.log(1); return 2; })
  .then(v => { console.log(v); return 3; }) // v = 2
  .then(v => console.log(v));              // v = 3
// 1  2  3

// 3 — async/await with setTimeout
async function run() {
  const result = await new Promise(resolve =>
    setTimeout(() => resolve("done"), 100)
  );
  console.log(result); // "done" after 100ms
}
run();
console.log("sync"); // "sync" first (before the 100ms)
// sync  done
```

---

## Q290–Q296. Advanced JavaScript Internals?

### WeakMap and WeakSet
```js
// WeakMap — keys must be objects, GC-friendly
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveCalc(obj);
  cache.set(obj, result);
  return result;
}
// When obj is no longer referenced elsewhere, cache entry is GC'd automatically

// Use case: private data for DOM elements
const metadata = new WeakMap();
metadata.set(element, { clicks: 0, lastSeen: Date.now() });
// When element is removed from DOM, metadata is automatically freed
```

### Property Descriptors
```js
Object.defineProperty(obj, "id", {
  value: 42,
  writable: false,    // cannot change value
  enumerable: false,  // won't appear in for...in or Object.keys
  configurable: false // cannot be redefined or deleted
});

// Getters and setters via defineProperty
Object.defineProperty(obj, "fullName", {
  get() { return `${this.first} ${this.last}`; },
  set(val) { [this.first, this.last] = val.split(" "); },
  enumerable: true,
  configurable: true,
});
```

### Proxy and Reflect
```js
const handler = {
  get(target, key) {
    return key in target ? target[key] : `Property "${key}" not found`;
  },
  set(target, key, value) {
    if (typeof value !== "number") throw new TypeError("Only numbers allowed");
    return Reflect.set(target, key, value); // delegate to default behavior
  },
};
const proxy = new Proxy({}, handler);
proxy.x = 5;   // OK
proxy.y = "s"; // TypeError
proxy.z;       // 'Property "z" not found'
```

---

## Q297. Machine Coding Round — What Interviewers Evaluate?

For 4.5 YOE, interviewers care more about **code quality and decision-making** than feature completeness.

**What they look for:**
1. Clarify requirements before coding
2. Component/module design — separation of concerns
3. State management — local vs lifted vs global
4. Reusable, composable code
5. Edge cases handled (empty state, loading, error)
6. Performance considerations (debounce on search, virtualization for large lists)
7. Clean, readable code with meaningful names

**Common tasks:**
- Infinite scroll / pagination
- Search with debounce
- Drag and drop Kanban
- Custom modal / toast system
- Form with validation
- Data table with sort + filter

```
// Approach template:
1. Understand → ask about edge cases, API format
2. Design → sketch component tree, state shape
3. Build → start with data layer, then UI
4. Polish → loading/error states, accessibility
5. Discuss → tradeoffs, what you'd do with more time
```

---

## Q298. Frontend System Design?

```
Given: Design a Twitter-like feed

1. Component Architecture
   FeedPage → FeedList → TweetCard → LikeButton
                      → LoadMoreButton
   Sidebar → TrendingTopics
             WhoToFollow

2. State Management
   Local:   form input, modal open/close
   Lifted:  feed data, user profile
   Global:  auth state, notification count

3. Data Fetching
   Initial: SSR or SSG for SEO
   Scroll:  cursor-based pagination (not offset — handles inserts)
   Real-time: WebSocket or Server-Sent Events for new tweets

4. Performance
   Virtual scrolling for feed (react-window)
   Image lazy loading
   Tweet components memoized (React.memo)
   Debounced search
   Code-split routes

5. Caching
   SWR / React Query — stale-while-revalidate
   localStorage for draft tweets

6. Error Handling
   Error boundaries per section
   Retry on failure
   Optimistic updates for likes

7. Security
   XSS: sanitize tweet content
   Auth: HttpOnly cookies for tokens
   Rate limiting: UI debounce + backend enforcement
```

---

## Q299. React vs Angular?

> Covered in phase5.md. Key for 4.5 YOE level:

```
React:
- Library, not framework — you assemble the stack
- Functional components + hooks (2019+)
- JSX — HTML in JS
- Virtual DOM
- Flexible: choose your router (React Router), state (Redux/Zustand/Jotai), forms

Angular:
- Full framework — router, HTTP, forms, DI all built-in
- TypeScript first
- Components + Directives + Pipes + Services
- Change detection (Zone.js or Signals in v17+)
- Opinionated: follows a strict structure (great for large teams)
- RxJS heavily used for async

Performance:
- Both are fast in production with proper optimization
- Angular Signals (v17) brings fine-grained reactivity similar to React
- React Compiler (2024) adds automatic memoization

For interviews: know which you've used deeply and be honest about the other
```

---

## Q300. Top Rapid-Fire Answers (4.5 YOE Level)?

```js
// What is the TDZ?
// The period between a let/const being hoisted and its initialization line.
// Accessing it throws ReferenceError.

// What does 'use strict' do?
// Enables strict mode: disables silent errors, forbids undeclared variables,
// this is undefined in regular functions (not window), disables with, etc.

// What is a Symbol?
// A unique, immutable primitive. Use as unique object keys to avoid collisions.
// Well-known Symbols (Symbol.iterator, Symbol.toPrimitive) hook into JS internals.

// What is a Proxy?
// Wraps an object and intercepts operations (get, set, delete, has, etc.)
// Used for validation, reactivity (Vue 3), logging, default values.

// What is the difference between == and Object.is()?
// Object.is(NaN, NaN) → true (unlike ===)
// Object.is(0, -0)   → false (unlike ===)
// Used by React for state comparison.

// What is tail call optimization?
// If the last operation in a function is a recursive call, some engines
// can reuse the current stack frame instead of creating a new one.
// ES6 specifies it but most engines (V8) don't fully implement it.

// What is a memory leak?
// Memory that is allocated but never freed because references to it still exist.
// Common causes: forgotten event listeners, uncleaned intervals, growing closures.

// What is the temporal dead zone?
// The time between the start of a block scope and the declaration of a let/const.
// Variable exists in memory but is inaccessible — ReferenceError if accessed.

// What makes a function 'async'?
// It always returns a Promise. await suspends the function (not the thread)
// and schedules the rest as a microtask when the awaited Promise resolves.

// What is tree shaking?
// Bundlers (Rollup, Webpack, Vite) analyze static imports and remove
// exported functions/variables that are never imported anywhere.
// Only works with static ES module imports (not CommonJS require).
```

---

## Q301. WeakMap vs Map?

> Covered in phase4.md Q136. Quick addition:

```js
// WeakMap use case: associate private data with objects
const _private = new WeakMap();

class Widget {
  constructor(element) {
    _private.set(this, { clicks: 0, element });
  }
  click() {
    _private.get(this).clicks++;
  }
  get clicks() {
    return _private.get(this).clicks;
  }
}
// _private data is GC'd when widget is destroyed — no memory leak
```

---

## Q302. WeakSet vs Set?

```js
// WeakSet — track object references without preventing GC
const seen = new WeakSet();

function processOnce(obj) {
  if (seen.has(obj)) return; // already processed
  seen.add(obj);
  doExpensiveWork(obj);
}
// When obj is no longer used elsewhere, seen entry is GC'd
```

---

## Q303. Property Descriptors?

> Covered in Q290 above.

---

## Q304. `Object.defineProperty()`?

> Covered in Q290 above. How Vue 2 built reactivity:

```js
// Vue 2 pattern (simplified)
function makeReactive(obj, key) {
  let value = obj[key];
  Object.defineProperty(obj, key, {
    get() {
      trackDependency(); // record what's watching this
      return value;
    },
    set(newVal) {
      value = newVal;
      notifyWatchers(); // trigger re-render
    },
  });
}
// Vue 3 replaced this with Proxy — more powerful (catches new properties)
```

---

## Q305. Dynamic Imports?

```js
// import() returns a Promise
button.addEventListener("click", async () => {
  const { Chart } = await import("./Chart.js"); // loaded only when clicked
  new Chart(canvas, config);
});

// React.lazy uses dynamic import internally
const Chart = React.lazy(() => import("./Chart"));

// Conditional loading
async function loadPolyfill() {
  if (!window.IntersectionObserver) {
    await import("intersection-observer"); // load only if needed
  }
}
```

---

## Q306. Tagged Template Literals?

> Covered in phase4.md Q102.

---

## Q307. BigInt?

```js
// Number.MAX_SAFE_INTEGER = 9007199254740991
// Beyond this, regular numbers lose precision
9007199254740991 + 1  // 9007199254740992 ✅
9007199254740991 + 2  // 9007199254740992 ❌ — same result!

// BigInt — arbitrary precision integers
9007199254740991n + 2n // 9007199254740993n ✅

// Cannot mix BigInt and Number
1n + 1  // TypeError
1n + BigInt(1) // 2n ✅
Number(2n)     // 2  ✅

// Use cases: financial calculations, cryptography, IDs from databases
```

---

## Q308. Structured Clone Algorithm?

> Covered in phase4.md Q132.

```js
// structuredClone handles what JSON.parse/stringify doesn't:
structuredClone(new Date());        // ✅ Date → Date (not string)
structuredClone(new Map([["k",1]])); // ✅ Map → Map
structuredClone(new Set([1,2,3]));  // ✅ Set → Set
structuredClone(/regex/g);          // ✅ RegExp → RegExp

// Circular references
const a = {}; a.self = a;
structuredClone(a); // ✅ handles circular refs
JSON.stringify(a);  // ❌ TypeError

// Does NOT handle:
structuredClone(() => {}); // ❌ DataCloneError — functions can't be cloned
structuredClone(undefined); // ✅ actually works (returns undefined)
```

---

## Q309. JavaScript Memory Leak Patterns?

> Covered in Q239 above.

---

## Q310. Parsing vs Compilation vs Execution?

```
1. Source code received by JS engine

2. PARSING
   Lexer  → tokenizes source into tokens (keywords, operators, identifiers)
   Parser → builds AST (Abstract Syntax Tree) from tokens
   SyntaxError thrown here for invalid syntax

3. COMPILATION (V8 — Ignition)
   AST → bytecode (Ignition interpreter)
   Execution begins immediately (fast startup)

4. OPTIMIZATION (V8 — TurboFan)
   Profiler identifies "hot" functions (called frequently)
   TurboFan JIT-compiles them to optimized machine code
   Assumptions recorded (e.g., "this function always receives numbers")

5. DEOPTIMIZATION
   If assumptions break (e.g., string passed where number expected)
   TurboFan discards optimized code → falls back to bytecode
   Performance temporarily drops

6. GARBAGE COLLECTION
   Mark-and-Sweep runs between execution ticks
   V8 uses generational GC: short-lived objects in "young" generation,
   long-lived objects promoted to "old" generation
```

This understanding explains why consistent types in hot functions matter for performance, and why V8 is so fast despite JavaScript being dynamic.
