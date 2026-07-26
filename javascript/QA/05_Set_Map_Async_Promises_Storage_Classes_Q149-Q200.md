# JavaScript Interview Q&A — Phase 5

> Q149–Q200 | Set/Map, Closures, Async, Promises, Browser APIs, Web Storage, Classes, ES6, Modules

---

## Q149. What is a Set?

A `Set` is a collection of **unique values** in insertion order. Any duplicate added is silently ignored.

```js
const set = new Set([1, 2, 2, 3, 3, 3]);
set;         // Set {1, 2, 3}
set.size;    // 3

set.add(4);
set.has(2);  // true
set.delete(2);
set.clear(); // empties the set

// Iterate
for (const val of set) console.log(val);
set.forEach(val => console.log(val));
[...set];    // convert to array
```

### Common use cases

```js
// 1. Remove duplicates from array (most common interview use)
const unique = [...new Set([1, 2, 2, 3, 3])]; // [1, 2, 3]

// 2. Set operations
const a = new Set([1, 2, 3, 4]);
const b = new Set([3, 4, 5, 6]);

const union        = new Set([...a, ...b]);                     // {1,2,3,4,5,6}
const intersection = new Set([...a].filter(x => b.has(x)));     // {3,4}
const difference   = new Set([...a].filter(x => !b.has(x)));    // {1,2}

// 3. Fast membership testing (O(1) vs array's O(n))
const banned = new Set(["spam@a.com", "bad@b.com"]);
if (banned.has(userEmail)) blockUser();
```

**Key points:**
- Uses `===` (strict equality) for uniqueness — `NaN` is treated as equal to itself (unlike `===`)
- Objects are compared by reference — `new Set([{}, {}]).size` is `2` (different references)
- Not indexed — no `set[0]`; iterate with `for...of` or spread to array first

---

## Q150. What is a Map?

A `Map` is a collection of **key-value pairs** where **any type** can be a key, and insertion order is preserved.

```js
const map = new Map();
map.set("name", "Alice");
map.set(42, "the answer");
map.set(true, "boolean key");
map.set({ id: 1 }, "object key");

map.get("name");   // "Alice"
map.get(42);       // "the answer"
map.has("name");   // true
map.size;          // 4
map.delete("name");
map.clear();

// Iteration
for (const [key, value] of map) console.log(key, value);
map.forEach((value, key) => console.log(key, value));
[...map.keys()];
[...map.values()];
[...map.entries()];

// Create from array of pairs or Object.entries
const m = new Map([["a", 1], ["b", 2]]);
const fromObj = new Map(Object.entries({ x: 10, y: 20 }));
```

---

## Q151. Difference between Map and Object?

| Feature | Object | Map |
|---------|--------|-----|
| Key types | String / Symbol only | Any type (objects, functions, primitives) |
| Key order | Insertion order (mostly, modern JS) | Guaranteed insertion order |
| Size | Manual: `Object.keys(o).length` | `map.size` |
| Prototype keys | ✅ Inherited keys can interfere | ❌ No prototype pollution |
| JSON support | ✅ `JSON.stringify` works | ❌ Needs custom serialization |
| Performance (many entries) | OK | Better for frequent add/delete |
| Default value | `undefined` for missing keys | `undefined` for missing keys |

```js
// Object prototype pollution risk
const obj = {};
obj["constructor"];  // points to Object constructor — potential conflict

// Map is clean
const map = new Map();
map.get("constructor"); // undefined — no prototype issue

// Use Map when:
// - Keys are not strings (using objects/numbers as keys)
// - You need guaranteed insertion order for all key types
// - Frequent additions/deletions
// - No JSON serialization needed

// Use Object when:
// - Simple string-keyed data
// - JSON serialization required
// - Modeling domain entities (user, product)
```

---

## Q152. What is a Closure? (Cross-reference)

> Covered in depth in JSQA.md Q31. Quick reference:

```js
function makeCounter(initial = 0) {
  let count = initial;  // closed-over variable
  return {
    increment: () => ++count,
    decrement: () => --count,
    reset:     () => { count = initial; },
    value:     () => count,
  };
}
const counter = makeCounter(10);
counter.increment(); // 11
counter.increment(); // 12
counter.value();     // 12
counter.reset();
counter.value();     // 10
```

---

## Q153. How Does a Closure Work?

> Covered in JSQA.md Q31. Key mechanism recap:

Every function has an internal `[[Environment]]` slot that holds a reference to the **Lexical Environment** where it was created. That environment stays alive in memory as long as any closure references it.

```js
// var-in-loop classic bug and fix
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3 3 3 — all share same i
}

for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0 1 2 — let creates new binding per iteration
}
```

---

## Q154. Practical Uses of Closures?

```js
// 1. Module pattern — private state
const cart = (() => {
  const items = [];
  return {
    add:   item  => items.push(item),
    remove: id   => items.splice(items.findIndex(i => i.id === id), 1),
    total: ()    => items.reduce((s, i) => s + i.price, 0),
    count: ()    => items.length,
  };
})();

// 2. Memoization
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

// 3. Partial application
function partial(fn, ...preset) {
  return (...rest) => fn(...preset, ...rest);
}
const add5 = partial((a, b) => a + b, 5);
add5(3); // 8

// 4. Once — run a function only once
function once(fn) {
  let called = false, result;
  return (...args) => {
    if (!called) { called = true; result = fn(...args); }
    return result;
  };
}
const initApp = once(setup);
initApp(); // runs setup
initApp(); // returns cached result, setup not called again
```

---

## Q155. How Do Closures Provide Data Privacy?

```js
class BankAccount {
  // ES2022 private fields (class-based privacy)
  #balance;
  constructor(initial) { this.#balance = initial; }
  deposit(n)  { this.#balance += n; }
  withdraw(n) {
    if (n > this.#balance) throw new Error("Insufficient funds");
    this.#balance -= n;
  }
  get balance() { return this.#balance; }
}

// Closure-based equivalent (pre-class or functional style)
function createAccount(initial) {
  let balance = initial; // private
  return {
    deposit:  n => { balance += n; },
    withdraw: n => {
      if (n > balance) throw new Error("Insufficient funds");
      balance -= n;
    },
    getBalance: () => balance,
  };
}

const acc = createAccount(1000);
acc.deposit(500);
acc.getBalance();  // 1500
acc.balance;       // undefined — not directly accessible
```

---

## Q156. How Do Closures Provide Persistent State?

```js
// Closures keep variables alive between function calls
function makeIdGenerator(prefix = "ID") {
  let n = 0;
  return () => `${prefix}-${++n}`;
}
const nextUserId    = makeIdGenerator("USR");
const nextProductId = makeIdGenerator("PRD");

nextUserId();    // "USR-1"
nextUserId();    // "USR-2"
nextProductId(); // "PRD-1" — independent counter
nextUserId();    // "USR-3"
```

React's `useState` is essentially a closure — the state value persists between renders because it lives in React's internal fiber structure, and the setter function closes over it.

---

## Q157. What is Encapsulation via Closures?

Encapsulation = bundling data and the methods that operate on it, while hiding internal details. Closures achieve this without classes.

```js
function createQueue() {
  const items = [];      // private
  let processingCount = 0; // private

  return {
    enqueue(item)  { items.push(item); },
    dequeue()      { return items.shift(); },
    peek()         { return items[0]; },
    get length()   { return items.length; },
    get isProcessing() { return processingCount > 0; },
    async process(fn) {
      processingCount++;
      try { await fn(this.dequeue()); }
      finally { processingCount--; }
    },
  };
}
```

---

## Q158. Limitations / Disadvantages of Closures?

```js
// 1. Memory retention — closed-over variables stay in memory
function createHeavy() {
  const bigData = new Array(1_000_000).fill("data");
  return () => bigData[0]; // bigData cannot be GC'd while closure exists
}
let fn = createHeavy();
fn(); // "data"
fn = null; // now bigData can be garbage collected

// 2. Stale closure in React — the most common real bug
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // ❌ stale — count is always 0 from first render
    }, 1000);
    return () => clearInterval(id);
  }, []); // empty deps → closes over initial count=0
}

// Fix — functional update form
setCount(prev => prev + 1); // ✅ doesn't rely on closed-over value
```

---

## Q159. How to Release Closures from Memory?

```js
// Dereference the closure to allow GC
let counter = makeCounter();
counter.increment(); // 1

counter = null; // all references removed → GC can collect the closed-over scope

// React — remove event listeners on cleanup
useEffect(() => {
  const handler = () => console.log(scrollY);
  window.addEventListener("scroll", handler);
  return () => window.removeEventListener("scroll", handler); // cleanup
}, []);

// Clear timers that hold closures
const id = setInterval(() => expensiveOp(), 1000);
clearInterval(id); // releases the closure
```

---

## Q160. Difference between Regular Function and Closure?

Every function in JavaScript technically forms a closure over its surrounding scope. The distinction that matters in interviews:

| | Regular Function | Closure |
|---|---|---|
| State between calls | ❌ Fresh execution each time | ✅ Retains outer variables |
| Private data | ❌ | ✅ |
| Memory | Released after call | Retained while closure alive |
| Common use | Stateless utilities | Stateful behavior, modules |

```js
// Regular — no state persistence
function add(a, b) { return a + b; }

// Closure — state persists
function makeAdder(a) {
  return b => a + b; // a is retained
}
const add10 = makeAdder(10);
add10(5);  // 15
add10(20); // 30
```

---

## Q161. What is Asynchronous Programming?

> Covered in depth in JSQA.md Q20. Key recap:

JavaScript is **single-threaded** — it processes one task at a time on the call stack. Asynchronous operations are handled by the **browser/Node runtime** (Web APIs), and their callbacks are scheduled via the **Event Loop**.

```js
// Mental model of async
console.log("1");          // sync — call stack
fetch("/api")              // handed to Web API (browser handles network)
  .then(r => r.json())     // scheduled as microtask when fetch resolves
  .then(console.log);      // scheduled after above microtask
console.log("2");          // sync — call stack

// Output: "1", "2", then API result
```

---

## Q162. Synchronous vs Asynchronous?

```js
// Synchronous — blocks until complete
const data = readFileSync("file.txt"); // Node.js — blocks entire process
console.log(data); // only runs after file is read

// Asynchronous — non-blocking
readFile("file.txt", (err, data) => console.log(data)); // callback when done
console.log("continues immediately"); // runs before file is read
```

**In the browser:** Synchronous JS blocks the UI thread — animations freeze, clicks don't respond. All I/O (network, timers, user events) must be asynchronous.

---

## Q163. Techniques for Async Operations?

| Technique | Era | Use when |
|-----------|-----|----------|
| Callbacks | ES5 | Simple one-time events (DOM listeners, setTimeout) |
| Promises | ES6 | Chained async operations, better error handling |
| Async/Await | ES2017 | Most code — reads like sync, cleanest |
| Generators | ES6 | Custom iterators, Redux Saga |
| RxJS Observables | Library | Streams, complex event sequences (Angular) |

---

## Q164. What is `setTimeout()`?

```js
// Syntax: setTimeout(callback, delayMs, ...args)
const id = setTimeout(() => console.log("done"), 1000);

// Cancel before it fires
clearTimeout(id);

// setTimeout with 0 — still asynchronous
console.log("A");
setTimeout(() => console.log("B"), 0); // goes to macrotask queue
console.log("C");
// Output: A C B

// Pass args to callback
setTimeout((name) => console.log(`Hello ${name}`), 500, "Alice");

// Common use cases
setTimeout(hideNotification, 3000);      // auto-dismiss
setTimeout(() => setVisible(false), 300); // delay after animation
```

**Key point:** `setTimeout(fn, 0)` does NOT mean immediate execution. It means "execute after the call stack is empty and all microtasks have run."

---

## Q165. What is `setInterval()`?

```js
// Execute repeatedly every N milliseconds
const id = setInterval(() => updateClock(), 1000);
clearInterval(id); // stop it

// Problem with setInterval — can overlap if callback takes longer than interval
// ✅ Better pattern — recursive setTimeout
function poll() {
  fetchStatus().then(status => {
    updateUI(status);
    setTimeout(poll, 2000); // schedules next only after current completes
  });
}
poll();

// React cleanup — always clear on unmount
useEffect(() => {
  const id = setInterval(() => setTime(new Date()), 1000);
  return () => clearInterval(id); // cleanup prevents memory leak
}, []);
```

---

## Q166. Role of Callbacks in Async?

Callbacks are the foundation of async JavaScript — a function you hand to another function to call **when the async work is done**.

```js
// Event-driven (still uses callbacks — that's fine)
button.addEventListener("click", handleClick);

// Node.js callback convention — error-first
fs.readFile("file.txt", (err, data) => {
  if (err) return handleError(err);
  processData(data);
});

// Problem — not composable or chainable
```

The limitations of callbacks (no return value, no `.catch`, hard to chain) led to Promises.

---

## Q167. What is Callback Hell? How to Avoid It?

```js
// Callback hell — pyramid of doom
getUser(id, (err, user) => {
  if (err) return cb(err);
  getOrders(user.id, (err, orders) => {
    if (err) return cb(err);
    getInvoice(orders[0].id, (err, invoice) => {
      if (err) return cb(err);
      sendEmail(user.email, invoice, (err) => {
        if (err) return cb(err);
        cb(null, "done");
      });
    });
  });
});

// Solution 1 — Promises (flat chain)
getUser(id)
  .then(user => getOrders(user.id))
  .then(orders => getInvoice(orders[0].id))
  .then(invoice => sendEmail(user.email, invoice))
  .catch(handleError);

// Solution 2 — Async/Await (reads like sync code)
async function processOrder(id) {
  const user    = await getUser(id);
  const orders  = await getOrders(user.id);
  const invoice = await getInvoice(orders[0].id);
  await sendEmail(user.email, invoice);
}
```

---

## Q168. What are Promises?

> Covered in depth in JSQA.md Q20. Key additions:

```js
// Creating a Promise
const p = new Promise((resolve, reject) => {
  // executor runs SYNCHRONOUSLY
  setTimeout(() => resolve("data"), 1000);  // fulfills after 1s
  // or: reject(new Error("failed"));
});

// The executor runs immediately — this is synchronous:
console.log("A");
new Promise(resolve => {
  console.log("B"); // runs synchronously inside executor
  resolve("C");
});
console.log("D");
// A B D — then "C" as microtask
```

### Promise chaining — value flows through `.then()`
```js
fetch("/users")
  .then(res => {
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return res.json();            // returns a new Promise
  })
  .then(users => users.filter(u => u.active))  // receives parsed data
  .then(active => render(active))
  .catch(err => showError(err))   // catches ANY error in the chain
  .finally(() => setLoading(false)); // always runs
```

---

## Q169. States of a Promise?

```
pending  →  fulfilled (resolved with a value)
         →  rejected  (rejected with a reason)
```

Once settled (fulfilled or rejected), a Promise is **immutable** — its state never changes again. This is called being "settled."

```js
const p = Promise.resolve(42);
p.then(v => console.log(v));  // 42
p.then(v => console.log(v));  // 42 again — same settled value, reusable

// Promise.resolve / Promise.reject — create already-settled promises
Promise.resolve("immediate");
Promise.reject(new Error("immediate failure"));
```

---

## Q170. What is Promise Chaining?

```js
// Each .then() receives the RETURN VALUE of the previous .then()
Promise.resolve(1)
  .then(n => n + 1)    // returns 2
  .then(n => n * 3)    // receives 2, returns 6
  .then(console.log);  // logs 6

// Returning a Promise from .then() flattens it (no nested promises)
fetch("/user/1")
  .then(res => res.json())         // returns Promise<data>
  .then(user => fetch(`/orders/${user.id}`)) // returns Promise<Response>
  .then(res => res.json())         // returns Promise<orders>
  .then(orders => render(orders));

// Common mistake — forgetting to return
.then(user => {
  fetch(`/orders/${user.id}`); // ❌ no return — next .then gets undefined
})
.then(res => res.json()); // ❌ TypeError: Cannot read property of undefined
```

---

## Q171. What is `Promise.all()`?

> Covered in JSQA.md Q20. Complete reference:

```js
// All must succeed — fail-fast on first rejection
const [users, products, orders] = await Promise.all([
  fetch("/users").then(r => r.json()),
  fetch("/products").then(r => r.json()),
  fetch("/orders").then(r => r.json()),
]);
// All three run in PARALLEL — much faster than sequential await

// Results are in the SAME ORDER as input, regardless of which resolved first
Promise.all([slowPromise, fastPromise])
  .then(([slow, fast]) => { }); // slow first, fast second

// One failure → entire Promise.all rejects
Promise.all([
  Promise.resolve("a"),
  Promise.reject("error"),
  Promise.resolve("c"),
]).catch(err => console.log(err)); // "error" — other results discarded
```

---

## Q172. What is `Promise.allSettled()`?

```js
// Waits for ALL — never rejects, reports each outcome
const results = await Promise.allSettled([
  fetch("/users").then(r => r.json()),
  fetch("/bad-endpoint"),           // this will fail
  fetch("/products").then(r => r.json()),
]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    render(result.value);
  } else {
    console.error("Failed:", result.reason);
  }
});

// Use when partial success is acceptable
// Dashboard: show available widgets even if some APIs fail
```

---

## Q173. What is `Promise.race()`?

```js
// First to SETTLE (resolve OR reject) wins
function withTimeout(promise, ms) {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(`Timeout after ${ms}ms`)), ms)
  );
  return Promise.race([promise, timeout]);
}

const data = await withTimeout(fetch("/api/slow"), 5000);
// If fetch takes >5s, rejects with timeout error

// Note: losing promises still execute — race doesn't cancel them
```

---

## Q174. What is `Promise.any()`?

```js
// First to SUCCEED wins — ignores rejections until all fail
const data = await Promise.any([
  fetch("https://cdn1.example.com/data"),
  fetch("https://cdn2.example.com/data"),
  fetch("https://cdn3.example.com/data"),
]);
// Returns fastest successful response — automatic CDN fallback

// All fail → rejects with AggregateError
Promise.any([
  Promise.reject("e1"),
  Promise.reject("e2"),
]).catch(err => {
  console.log(err instanceof AggregateError); // true
  console.log(err.errors); // ["e1", "e2"]
});
```

---

## Q175. `Promise.all` vs `allSettled` vs `race` vs `any`?

| Method | Resolves when | Rejects when | Use case |
|--------|--------------|-------------|----------|
| `all` | ALL succeed | ANY fails | Need all results, fail-fast |
| `allSettled` | ALL settle (either way) | Never | Need all outcomes, partial OK |
| `race` | FIRST settles | FIRST rejects | Timeouts, fastest response |
| `any` | FIRST succeeds | ALL fail | Fallbacks, first available |

```js
// Decision guide
// "I need all data before rendering" → Promise.all
// "Show what's available, skip failures" → Promise.allSettled
// "Cancel if takes too long" → Promise.race
// "Use first available CDN/server" → Promise.any
```

---

## Q176. What is Async/Await?

> Covered in JSQA.md Q20. Key additions:

```js
// async function always returns a Promise — even if you return a plain value
async function getNum() { return 42; }
getNum().then(console.log); // 42

// await unwraps a Promise — pauses the async function (not the whole thread)
async function loadDashboard(userId) {
  setLoading(true);
  try {
    // Parallel — faster than sequential
    const [user, settings] = await Promise.all([
      getUser(userId),
      getSettings(userId),
    ]);

    // Sequential — when second depends on first
    const orders = await getOrders(user.id);

    return { user, settings, orders };
  } catch (err) {
    setError(err.message);
    throw err;    // re-throw so caller knows it failed
  } finally {
    setLoading(false);
  }
}
```

---

## Q177. How Does Async/Await Work Internally?

```js
// This async/await code:
async function example() {
  const a = await stepOne();
  const b = await stepTwo(a);
  return b;
}

// Is equivalent to this Promise chain:
function example() {
  return stepOne()
    .then(a => stepTwo(a))
    .then(b => b);
}

// Key execution flow
async function demo() {
  console.log("1");          // sync
  const val = await getVal(); // pauses here, returns to caller
  console.log("3");          // resumes as microtask after getVal resolves
}
console.log("0");
demo();
console.log("2");
// Output: 0 1 2 3
```

`await` suspends the async function and schedules the rest of it in the **microtask queue**. The main thread is NOT blocked.

---

## Q178. Error Handling with Async/Await?

```js
// try/catch — handles both sync throws and rejected Promises
async function loadUser(id) {
  try {
    const res = await fetch(`/users/${id}`);
    if (!res.ok) throw new ApiError(res.status, "Fetch failed");
    return await res.json();
  } catch (err) {
    if (err instanceof ApiError && err.status === 404) return null;
    throw err; // unexpected errors bubble up
  }
}

// .catch() on the returned Promise — equivalent
loadUser(1).catch(err => console.error(err));

// Avoid await inside catch — use finally for cleanup
async function withCleanup() {
  let connection;
  try {
    connection = await openConnection();
    return await connection.query("SELECT ...");
  } finally {
    await connection?.close(); // always close, even on error
  }
}
```

---

## Q179. Promises vs Async/Await?

```js
// Same operation, two styles
// Promises
function loadUser(id) {
  return fetch(`/users/${id}`)
    .then(res => { if (!res.ok) throw new Error(res.status); return res.json(); })
    .catch(err => { logError(err); throw err; });
}

// Async/Await
async function loadUser(id) {
  const res = await fetch(`/users/${id}`);
  if (!res.ok) throw new Error(res.status);
  return res.json(); // errors from above propagate automatically
}
```

| | Promises | Async/Await |
|---|---|---|
| Readability | OK for chains | Reads like sync code |
| Error handling | `.catch()` | `try/catch` |
| Debugging | Stack traces can be confusing | Cleaner stack traces |
| Parallel | `Promise.all(...)` | `await Promise.all(...)` |
| Built on | Native | Syntactic sugar over Promises |

**Choose async/await** for most code. Use raw Promises when you need `Promise.all`, `Promise.race`, or when building utilities that return Promises.

---

## Q180. What is the Event Loop?

> Covered in depth in JSQA.md Q20. Complete model:

```
┌──────────────────────────────┐
│         Call Stack           │  ← executes one frame at a time
│  [main] → [fetch] → [then]  │
└─────────────┬────────────────┘
              │ offloads async work
┌─────────────▼────────────────┐
│       Web / Node APIs        │  ← browser handles: timers, fetch, events
└──────────┬───────────────────┘
           │ completion → push to queue
    ┌──────▼──────┐    ┌───────────────────┐
    │  Macrotask  │    │  Microtask Queue  │
    │   Queue     │    │  (higher priority)│
    │ setTimeout  │    │  Promise.then     │
    │ setInterval │    │  async/await      │
    │ DOM events  │    │  queueMicrotask() │
    └──────┬──────┘    └────────┬──────────┘
           └──────────┬─────────┘
                      ▼
           ┌─────────────────────┐
           │     Event Loop      │  Checks: is call stack empty?
           │  1. Drain microtask │   → yes: run ALL microtasks
           │  2. Run ONE macro   │   → then ONE macrotask
           └─────────────────────┘   → repeat
```

---

## Q181. Call Stack, Web APIs, and Callback Queue?

```js
// Execution trace
function greet(name) { return `Hello ${name}`; }
function main() {
  const result = greet("Alice");  // greet pushed, then popped
  console.log(result);            // console.log pushed, then popped
}
main(); // main pushed to stack

// Async trace
console.log("start");            // call stack: console.log → pop
setTimeout(cb, 0);               // call stack: setTimeout → Web API timer → pop
console.log("end");              // call stack: console.log → pop
// timer fires → cb goes to macrotask queue
// Event Loop: stack empty → pushes cb → executes
```

---

## Q182. Microtask vs Macrotask Queue?

> Covered in JSQA.md Q20. Classic interview output question:

```js
console.log("1");                          // sync
setTimeout(() => console.log("2"), 0);    // macrotask
Promise.resolve().then(() => console.log("3")); // microtask
queueMicrotask(() => console.log("4"));   // microtask
console.log("5");                          // sync

// Output: 1  5  3  4  2
// Sync first → then ALL microtasks (3 then 4) → then ONE macrotask (2)

// Microtask sources: Promise.then/catch/finally, async/await, queueMicrotask, MutationObserver
// Macrotask sources: setTimeout, setInterval, setImmediate, DOM events, I/O
```

**Critical rule:** The Event Loop drains the **entire** microtask queue between each macrotask. New microtasks added during microtask processing are also processed before the next macrotask.

---

## Q183. Event Loop Execution Order (Interview Questions)?

```js
// Classic 3-part question
async function run() {
  console.log("A");
  await Promise.resolve();  // suspends, schedules rest as microtask
  console.log("B");
}

setTimeout(() => console.log("C"), 0);
run();
console.log("D");

// Output: A  D  B  C
// Explanation: A (sync in run), D (sync after run()), B (microtask from await), C (macrotask)
```

---

## Q184. What are Browser APIs?

Browser APIs are capabilities provided by the **browser environment** (not the JS language itself). JavaScript calls them, the browser handles the work.

| Category | APIs |
|----------|------|
| DOM | `document`, `element`, `event` |
| Network | `fetch`, `XMLHttpRequest`, `WebSocket` |
| Storage | `localStorage`, `sessionStorage`, `IndexedDB`, `cookies` |
| History/Navigation | `history.pushState`, `location` |
| Timers | `setTimeout`, `setInterval`, `requestAnimationFrame` |
| Geolocation | `navigator.geolocation` |
| Media | `getUserMedia`, `HTMLMediaElement` |
| Workers | `Worker`, `SharedWorker`, `ServiceWorker` |
| Notifications | `Notification` |
| Clipboard | `navigator.clipboard` |

**None of these are in the ECMAScript spec** — they're browser extensions. Node.js has its own set (fs, http, path, etc.).

---

## Q185. What is Web Storage? Types?

Web Storage lets web apps store **key-value string data** in the browser. Two types:

| | `localStorage` | `sessionStorage` |
|---|---|---|
| Lifetime | Permanent (until cleared) | Tab session only |
| Scope | All tabs, same origin | Current tab only |
| Capacity | ~5–10 MB | ~5 MB |
| Server sent? | ❌ No | ❌ No |
| Use for | User preferences, theme, cache | Wizard state, temp checkout |

Both share the same API:
```js
storage.setItem("key", "value");
storage.getItem("key");
storage.removeItem("key");
storage.clear();
storage.length;
storage.key(0); // key at index
```

---

## Q186. What is localStorage?

```js
// Store — values must be strings
localStorage.setItem("theme", "dark");
localStorage.setItem("user", JSON.stringify({ name: "Alice", id: 1 }));

// Retrieve
const theme = localStorage.getItem("theme");              // "dark"
const user  = JSON.parse(localStorage.getItem("user"));   // { name, id }

// Remove
localStorage.removeItem("theme");
localStorage.clear(); // removes ALL items for this origin

// Check existence
localStorage.getItem("missing"); // null — not undefined

// Practical pattern
const getStoredUser = () => {
  try {
    const raw = localStorage.getItem("user");
    return raw ? JSON.parse(raw) : null;
  } catch {
    return null; // handle malformed JSON
  }
};
```

**Security note:** localStorage is accessible to any JavaScript on the page. Never store sensitive tokens without understanding the XSS risk. Consider `HttpOnly` cookies for auth tokens.

---

## Q187. What is sessionStorage?

```js
// Same API as localStorage
sessionStorage.setItem("step", "2");
sessionStorage.getItem("step"); // "2"

// Cleared when TAB closes — survives page refresh
sessionStorage.setItem("formData", JSON.stringify(formState));
window.location.reload();
sessionStorage.getItem("formData"); // still there

// Each tab has its own sessionStorage — not shared
// Opening page in new tab → new empty sessionStorage
```

---

## Q188. Difference between localStorage and sessionStorage?

> See Q185 table. Key interviewer follow-ups:

```js
// Tab isolation — sessionStorage is tab-specific
// Open same URL in 3 tabs → 3 independent sessionStorages

// Persistence test
localStorage.setItem("a", "1");
sessionStorage.setItem("b", "2");
// Close and reopen browser:
localStorage.getItem("a");   // "1" — survives
sessionStorage.getItem("b"); // null — cleared on close
```

---

## Q189. Storage Capacity Limits?

| Storage | Typical Limit |
|---------|--------------|
| localStorage | 5–10 MB (Chrome: 10 MB, Safari: 5 MB) |
| sessionStorage | 5 MB |
| Cookies | 4 KB per cookie |
| IndexedDB | Hundreds of MB (device-dependent) |

```js
// Quota exceeded error
try {
  localStorage.setItem("large", "x".repeat(10_000_000));
} catch (e) {
  if (e instanceof DOMException && e.name === "QuotaExceededError") {
    console.error("Storage quota exceeded");
  }
}
```

---

## Q190. What are Cookies?

```js
// Set a cookie
document.cookie = "username=Alice; max-age=3600; path=/";
document.cookie = "theme=dark; expires=Fri, 31 Dec 2030 23:59:59 GMT";

// Read all cookies as a string (all at once, no filtering)
document.cookie; // "username=Alice; theme=dark"

// Parse a specific cookie
function getCookie(name) {
  return document.cookie.split("; ")
    .find(row => row.startsWith(`${name}=`))
    ?.split("=")[1];
}

// Delete — set max-age=0 or past expiry
document.cookie = "username=; max-age=0";
```

### Cookie attributes
| Attribute | Purpose |
|-----------|---------|
| `max-age` / `expires` | Lifetime |
| `path` | Which paths can access it |
| `domain` | Which domain can access it |
| `Secure` | HTTPS only |
| `HttpOnly` | Not accessible via JS — XSS protection |
| `SameSite` | CSRF protection: `Strict`, `Lax`, `None` |

---

## Q191. Difference between Cookies and Web Storage?

| | Cookies | localStorage | sessionStorage |
|---|---|---|---|
| Size limit | ~4 KB | ~5–10 MB | ~5 MB |
| Sent to server | ✅ Every HTTP request | ❌ Never | ❌ Never |
| JS accessible | ✅ (unless HttpOnly) | ✅ | ✅ |
| Expiry | Configurable | Manual | Tab close |
| CSRF risk | ✅ (use SameSite) | ❌ | ❌ |
| XSS risk | Lower (HttpOnly) | Higher | Higher |

**Auth token storage debate:**
- `HttpOnly cookie` — server sets it, JS can't read it → best XSS protection
- `localStorage` — accessible to JS → XSS risk
- **Recommendation:** Use `HttpOnly` `Secure` `SameSite=Strict` cookies for auth tokens

---

## Q192. When to Use Cookies vs Web Storage?

```
Need server to receive it on every request?  → Cookie
Auth session token?                           → HttpOnly Cookie
User preference (theme, language)?            → localStorage
Wizard/multi-step form state?                 → sessionStorage
Data that must survive browser restart?       → localStorage
Data only needed for current tab session?     → sessionStorage
Large dataset (>5 MB)?                        → IndexedDB
```

---

## Q193. What are Classes in JavaScript?

Classes are **syntactic sugar over prototype-based inheritance** — under the hood, they still use the same prototype chain.

```js
class Animal {
  #name;           // private field (ES2022)
  #sound;

  constructor(name, sound) {
    this.#name  = name;
    this.#sound = sound;
  }

  speak() {
    return `${this.#name} says ${this.#sound}`;
  }

  get name()    { return this.#name; }  // getter
  set name(val) { this.#name = val; }   // setter

  static create(name, sound) {          // static method
    return new Animal(name, sound);
  }
}

class Dog extends Animal {
  constructor(name) {
    super(name, "woof");  // must call super() before using this
  }

  fetch(item) { return `${this.name} fetches ${item}`; }
}

const dog = new Dog("Rex");
dog.speak();   // "Rex says woof"
dog.fetch("ball"); // "Rex fetches ball"
Dog.create("Buddy", "bark"); // static method on parent
```

---

## Q194. What is a Constructor?

```js
class User {
  constructor(name, email) {  // called automatically with new
    this.name  = name;
    this.email = email;
    this.createdAt = new Date();
  }
}

// Only one constructor per class (SyntaxError for multiple)
// If omitted, a default empty constructor is used

// new keyword:
// 1. Creates a new empty object
// 2. Sets its prototype to User.prototype
// 3. Calls constructor with this = new object
// 4. Returns the new object (unless constructor explicitly returns an object)
const u = new User("Alice", "alice@example.com");
```

---

## Q195. What are Constructor Functions?

```js
// Pre-ES6 way to create objects with shared prototype methods
function Person(name, age) {
  this.name = name;
  this.age  = age;
}

// Add to prototype — shared by all instances (memory-efficient)
Person.prototype.greet = function() {
  return `Hi, I'm ${this.name}`;
};

const alice = new Person("Alice", 30);
alice.greet(); // "Hi, I'm Alice"

// Under the hood, class syntax does the same thing:
class Person {
  constructor(name) { this.name = name; }
  greet() { return `Hi, I'm ${this.name}`; }  // goes to Person.prototype
}
// typeof Person === "function" — classes are functions internally
```

---

## Q196. What is the `this` Keyword?

> Core covered in JSQA.md Q13 and finalphase.md Q285. Reference summary:

```js
// this is determined at CALL TIME, not definition time (except arrow functions)

// 1. Method call — this = the object
user.greet();  // this = user

// 2. Regular function call — this = undefined (strict) or window (non-strict)
greet();       // this = undefined in modules / strict mode

// 3. Constructor — this = new object
new User();    // this = new User instance

// 4. Arrow function — lexical this (inherits from enclosing scope)
const obj = {
  value: 42,
  getValue: () => this.value,  // this = module/window, NOT obj
  getValue2() { return this.value; }, // this = obj ✅
};

// 5. Explicit binding
fn.call(obj, args);
fn.apply(obj, [args]);
const bound = fn.bind(obj);

// 6. Event handler — this = element that triggered the event
btn.addEventListener("click", function() { this === btn; }); // true
btn.addEventListener("click", () => { this !== btn; });       // arrow: lexical this
```

---

## Q197. What is Prototypal Inheritance?

```js
// Every object has a hidden [[Prototype]] link to another object
// Property lookup walks the chain until found or null

const animal = {
  breathe() { return `${this.name} is breathing`; }
};

const dog = Object.create(animal);
dog.name = "Rex";
dog.bark = function() { return "Woof!"; };

dog.bark();    // found on dog itself
dog.breathe(); // not on dog → found on animal (prototype)
dog.toString();// not on dog or animal → found on Object.prototype

// Prototype chain: dog → animal → Object.prototype → null

Object.getPrototypeOf(dog) === animal;  // true
dog.__proto__ === animal;               // true (avoid __proto__, use getPrototypeOf)

// Classes use prototypes under the hood
class Dog extends Animal {}
// Dog.prototype.__proto__ === Animal.prototype
```

---

## Q198. Important ES6+ Features?

```js
// 1. let / const (block scope)
// 2. Arrow functions (lexical this)
// 3. Template literals (`${expr}`)
// 4. Destructuring (arrays and objects)
// 5. Default parameters
// 6. Rest/Spread (...) 
// 7. Classes (syntax sugar over prototypes)
// 8. Modules (import/export)
// 9. Promises
// 10. Symbol
// 11. Map / Set / WeakMap / WeakSet
// 12. for...of
// 13. Generators (function*)
// 14. Proxy / Reflect

// ES2017+
// async/await
// Object.entries() / Object.values()

// ES2020
// Optional chaining (?.)
// Nullish coalescing (??)
// Promise.allSettled()
// BigInt

// ES2021
// Logical assignment (&&=, ||=, ??=)
// Promise.any()
// String.replaceAll()

// ES2022
// Array.at(-1)
// Object.hasOwn()
// Private class fields (#name)
// structuredClone() [actually globalThis — available in modern environments]
```

---

## Q199. What are JavaScript Modules?

```js
// Named exports — export multiple values from a file
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export class Calculator { /* ... */ }

// Import named
import { PI, add } from "./math.js";
import { add as sum } from "./math.js";  // rename
import * as MathUtils from "./math.js"; // namespace import

// Default export — one per file, any name on import
// user.js
export default class User { /* ... */ }

// Import default
import User from "./user.js";         // name can be anything
import MyUser from "./user.js";       // same thing, different local name

// Mixed
import User, { validateUser } from "./user.js";

// Dynamic import — lazy loading
const { add } = await import("./math.js");
const Chart = (await import("./Chart.js")).default;
```

**Modules are always strict mode.** Top-level `await` works in modules (ES2022). Each module has its own scope — no global pollution.

---

## Q200. What is `export`/`import`? What are Bundlers?

```js
// Re-export — barrel files (index.js pattern)
// components/index.js
export { Button } from "./Button.js";
export { Modal }  from "./Modal.js";
export { Table }  from "./Table.js";
// Then: import { Button, Modal } from "./components"

// Circular imports — can cause issues
// A imports B, B imports A → one will get undefined at import time
// Fix: extract shared code to a third module C

// Tree shaking — bundlers remove unused exports
// Only works with named static exports (not dynamic import())
```

### Bundlers

| Bundler | Used in | Key feature |
|---------|---------|-------------|
| **Webpack** | Older React, Angular | Highly configurable, rich ecosystem |
| **Vite** | Modern React, Vue | Uses native ES modules in dev, Rollup for prod — very fast |
| **Rollup** | Libraries | Excellent tree-shaking, small output |
| **Parcel** | Quick prototypes | Zero config |
| **esbuild** | Go-based, used inside Vite | Extremely fast compilation |

**What bundlers do:**
- Resolve module dependencies
- Bundle many files into fewer (code splitting)
- Tree-shake unused code
- Transpile (via Babel/SWC)
- Minify/compress
- Handle assets (CSS, images)
