# JavaScript Interview Q&A — Phase 3

> Q51–Q100 | Conditionals, Arrays (Sort/Destructuring), Loops, Functions, Strings

---

## Q51. When to use `if-else` vs `switch` vs Ternary in real applications?

The right tool depends on the shape of the condition:

| Construct | Best for |
|-----------|----------|
| `if-else` | Range checks, complex multi-condition logic, multiple statements per branch |
| `switch` | One variable against many exact values |
| Ternary `? :` | Single condition, value assignment, JSX rendering |
| `&&` | Render something or nothing in JSX |
| Object map | Many fixed key→value or key→function mappings |

```js
// if-else — ranges and complex logic
if (score >= 90) grade = "A";
else if (score >= 75) grade = "B";
else grade = "C";

// switch — exact values, same variable
switch (status) {
  case "PENDING":   showSpinner(); break;
  case "SUCCESS":   showData(); break;
  case "ERROR":     showError(); break;
  default:          showUnknown();
}

// Ternary — simple assignment or JSX
const label = isActive ? "Active" : "Inactive";
return isLoading ? <Spinner /> : <Dashboard />;

// Object map — cleaner than large switch
const messages = { 200: "OK", 404: "Not Found", 500: "Server Error" };
const msg = messages[statusCode] ?? "Unknown";
```

**Key points:**
- `switch` without `break` causes **fall-through** — each case executes into the next
- Never nest ternaries — use `if-else` when you have 3+ conditions
- `switch` uses strict equality (`===`) for comparisons

---

## Q52. Difference between `==` and `===`?

> See JSQA.md Q30 for the full table. Summary for this phase:

```js
// The only legitimate use of ==
if (value == null) { }  // catches both null AND undefined in one check
```

### `Object.is()` — the strictest comparison

```js
Object.is(NaN, NaN);   // true  — unlike === which gives false
Object.is(0, -0);      // false — unlike === which gives true
Object.is(1, 1);       // true
```

`Object.is` is used internally by React for state change detection (`useState` comparison).

---

## Q53. Difference between Spread and Rest? *(cross-reference)*

> Covered in detail in phase2.md Q31. Summary:
- **Spread** → expands: `Math.max(...arr)`, `[...arr1, ...arr2]`, `{...obj}`
- **Rest** → collects: `function fn(a, ...rest)`, `const [first, ...tail] = arr`

---

## Q54. What are Arrays? How to Get, Add and Remove Elements?

> Covered in JSQA.md Q14 and phase2.md Q32. Key additions:

### Accessing elements
```js
arr[0];           // first
arr[arr.length - 1]; // last
arr.at(-1);       // last (ES2022, cleaner)
arr.at(-2);       // second to last
```

### Immutable patterns for React state
```js
const arr = [1, 2, 3];

// Add
[...arr, 4]                    // add to end
[0, ...arr]                    // add to start
[...arr.slice(0,2), 99, ...arr.slice(2)] // insert at index 2

// Remove
arr.filter((_, i) => i !== 1)  // remove at index 1
arr.filter(x => x !== 3)       // remove by value

// Update
arr.map((x, i) => i === 1 ? 99 : x) // update at index 1
```

---

## Q55. What is `indexOf()` in Arrays?

> Covered in phase2.md Q33. Key trap:

```js
// indexOf uses strict equality — doesn't work for objects by value
const users = [{ id: 1 }, { id: 2 }];
users.indexOf({ id: 1 }); // -1 — different reference!

// For objects, use findIndex()
users.findIndex(u => u.id === 1); // 0
```

---

## Q56. Difference between `find()` and `filter()`?

> Covered in phase2.md Q34.

Additional:
```js
// findIndex() — like find() but returns index instead of element
const idx = users.findIndex(u => u.id === 101); // -1 if not found
```

---

## Q57. What is `slice()`?

> Covered in phase2.md Q35. Additional use case:

```js
// Clone entire array
const copy = arr.slice();     // same as [...arr]

// Pagination
function getPage(data, page, size) {
  return data.slice((page - 1) * size, page * size);
}
```

---

## Q58. Difference between `push()` and `concat()`?

> Covered in phase2.md Q36.

```js
// Modern preferred pattern in React — spread
const newArr = [...arr, item];        // add one
const merged = [...arr1, ...arr2];    // merge two
```

---

## Q59. Difference between `pop()` and `shift()`?

> Covered in phase2.md Q37.

**Performance note:** `shift()` is O(n) because all remaining elements must be re-indexed. For performance-sensitive queue implementations, use a pointer-based approach or consider a `Map`.

---

## Q60. What is `splice()`?

> Covered in phase2.md Q38.

---

## Q61. Difference between `slice()` and `splice()`?

| | `slice()` | `splice()` |
|---|---|---|
| Mutates original | ❌ No | ✅ Yes |
| Returns | Copy of portion | Removed elements |
| Supports add/replace | ❌ | ✅ |
| React-safe | ✅ | ❌ |

Memory trick: **slice** = salami (clean cut, original intact). **splice** = surgery (modifies the body).

---

## Q62. Difference between `map()` and `forEach()`?

```js
const arr = [1, 2, 3];

// map — returns new array (use for transformation)
const doubled = arr.map(n => n * 2);    // [2, 4, 6]

// forEach — returns undefined (use for side effects)
arr.forEach(n => console.log(n));        // undefined
```

| | `map()` | `forEach()` |
|---|---|---|
| Returns | New array | `undefined` |
| Use for | Transformation | Side effects (logging, API calls) |
| Break/continue | ❌ No | ❌ No |
| Chainable | ✅ `.map().filter()` | ❌ |
| React JSX | ✅ Used for rendering | ❌ |

**Critical trap — async with forEach:**
```js
// ❌ forEach does NOT await — all iterations fire simultaneously
arr.forEach(async item => { await save(item); });

// ✅ for...of — awaits each iteration sequentially
for (const item of arr) { await save(item); }

// ✅ Promise.all — parallel but awaited
await Promise.all(arr.map(item => save(item)));
```

---

## Q63. How to Sort and Reverse an Array?

```js
// Default sort — converts to STRING, broken for numbers
[10, 2, 30].sort();             // [10, 2, 30] — lexicographic!

// Correct numeric sort
[10, 2, 30].sort((a, b) => a - b);   // [2, 10, 30] ascending
[10, 2, 30].sort((a, b) => b - a);   // [30, 10, 2] descending

// Sort strings
["banana", "apple", "cherry"].sort((a, b) => a.localeCompare(b));

// Sort objects by property
users.sort((a, b) => a.age - b.age);
users.sort((a, b) => a.name.localeCompare(b.name));

// Reverse
arr.reverse();  // mutates original!

// React-safe sort/reverse — always spread first
const sorted = [...arr].sort((a, b) => a - b);
const reversed = [...arr].reverse();
```

**Common trap:** Both `sort()` and `reverse()` **mutate** the original array. In React, always clone first with `[...arr]`.

---

## Q64. What is Array Destructuring?

Array destructuring extracts values into variables using position.

```js
const [a, b, c] = [10, 20, 30];  // a=10, b=20, c=30

// Skip elements
const [first, , third] = [1, 2, 3];  // first=1, third=3

// Default values
const [x = 0, y = 0] = [5];          // x=5, y=0

// Rest in destructuring
const [head, ...tail] = [1, 2, 3, 4]; // head=1, tail=[2,3,4]

// Swap variables — elegant trick
let a2 = 1, b2 = 2;
[a2, b2] = [b2, a2];  // a2=2, b2=1

// React useState — uses array destructuring
const [count, setCount] = useState(0);
```

**Key point:** Order matters in array destructuring — you assign by position, not by name. Use object destructuring when names matter.

---

## Q65. What are Array-like Objects?

Objects that have indexed elements and a `length` property but are **not actual Arrays** — they don't have array methods like `map`, `filter`, etc.

| Array-like | Where from |
|-----------|-----------|
| `arguments` | Inside non-arrow functions |
| `NodeList` | `querySelectorAll()` |
| `HTMLCollection` | `getElementsByClassName()`, `getElementsByTagName()` |
| Strings | `"hello"[0]` works, but no `push` |

```js
function demo() {
  console.log(arguments);        // array-like, not an array
  arguments.map(x => x);        // ❌ TypeError
  Array.from(arguments).map(x => x); // ✅
}
```

---

## Q66. How to Convert Array-like Objects to Arrays?

Three approaches — modern preference is `Array.from()` or spread:

```js
const nodeList = document.querySelectorAll(".card");

// Method 1 — Array.from() (most readable)
const arr1 = Array.from(nodeList);

// Method 2 — Spread (concise)
const arr2 = [...nodeList];

// Method 3 — slice.call() (legacy, still seen in old codebases)
const arr3 = Array.prototype.slice.call(nodeList);

// After conversion, all array methods work
arr1.forEach(el => el.classList.add("active"));
arr1.map(el => el.textContent);
```

**`Array.from()` bonus — with map function:**
```js
Array.from({ length: 5 }, (_, i) => i + 1); // [1, 2, 3, 4, 5]
Array.from("hello");  // ["h","e","l","l","o"]
```

---

## Q67. What are Loops in JavaScript? Types?

> See JSQA.md Q11 for the full breakdown. This section adds depth.

### `break` and `continue`

```js
// break — exits the loop entirely
for (let i = 0; i < 5; i++) {
  if (i === 3) break;
  console.log(i);  // 0 1 2
}

// continue — skips current iteration
for (let i = 0; i < 5; i++) {
  if (i === 3) continue;
  console.log(i);  // 0 1 2 4
}

// break in switch inside a loop — only exits the switch, not the loop
for (const item of items) {
  switch (item.type) {
    case "A": process(item); break; // breaks switch, loop continues
  }
}
```

**Common trap:** `break` inside `forEach` throws a `SyntaxError` — `forEach` is a function call, not a loop. Use `for...of` when you need early exit.

---

## Q68. Difference between `while` and `for` Loops?

```js
// for — initialization, condition, and increment in one place
// Best when: number of iterations is known
for (let i = 0; i < arr.length; i++) { }

// while — only condition
// Best when: iterations depend on a changing condition
let retries = 0;
while (retries < 3 && !success) {
  success = attemptConnection();
  retries++;
}
```

| | `for` | `while` |
|---|---|---|
| Best for | Known iteration count | Unknown / condition-based |
| Readability | Compact | More flexible |
| Infinite loop risk | Lower (increment explicit) | Higher (easy to forget increment) |

---

## Q69. Difference between `while` and `do...while`?

```js
// while — checks condition BEFORE first execution
let i = 10;
while (i < 5) { console.log(i); } // never runs

// do...while — executes ONCE before checking condition
let j = 10;
do { console.log(j); } while (j < 5); // prints 10 once
```

**Real-world use:** `do...while` is useful when you must run code at least once regardless of condition — e.g., prompting for input until valid, or retry logic where the first attempt should always happen.

---

## Q70. Difference between `break` and `continue`?

```js
// break — terminates the entire loop
for (let i = 0; i < 5; i++) {
  if (i === 3) break;
}
// i never reaches 4 or 5

// continue — skips rest of current iteration, continues loop
for (let i = 0; i < 5; i++) {
  if (i === 3) continue;
  console.log(i); // 0 1 2 4
}
```

| | `break` | `continue` |
|---|---|---|
| Effect | Exits the entire loop | Skips current iteration |
| Works in `forEach`? | ❌ | ❌ |
| Works in `switch`? | ✅ (exits switch) | ❌ |
| Works in `for...of`? | ✅ | ✅ |

---

## Q71. Difference between `for` and `for...of`?

```js
const skills = ["JS", "React", "Angular"];

// for — index-based, full control
for (let i = 0; i < skills.length; i++) {
  console.log(i, skills[i]);  // 0 "JS", 1 "React"...
}

// for...of — value-based, cleaner
for (const skill of skills) {
  console.log(skill);  // "JS", "React", "Angular"
}

// for...of works on any iterable
for (const char of "hello") { console.log(char); }
for (const [key, val] of map) { console.log(key, val); }
for (const item of new Set([1,2,3])) { console.log(item); }
```

**When to prefer `for` over `for...of`:** When you need the index, or when breaking by index condition.

---

## Q72. Difference between `for...of` and `for...in`?

```js
const arr = ["a", "b", "c"];
const obj = { x: 1, y: 2 };

// for...of — iterates VALUES of iterables (arrays, strings, maps, sets)
for (const val of arr) { console.log(val); }  // "a" "b" "c"

// for...in — iterates KEYS of objects (and array indexes as strings)
for (const key in obj) { console.log(key); }  // "x" "y"
for (const key in arr) { console.log(key); }  // "0" "1" "2" — string keys!
```

**Common trap:** Using `for...in` on an array gives you string indexes (`"0"`, `"1"`) and can also iterate inherited prototype properties. Always use `for...of` for arrays.

| | `for...of` | `for...in` |
|---|---|---|
| Use with | Arrays, strings, iterables | Objects |
| Returns | Values | Keys (as strings) |
| Arrays | ✅ Preferred | ❌ Avoid |

---

## Q73. `forEach()` vs `for...of` vs `for...in` comparison?

| Feature | `forEach` | `for...of` | `for...in` |
|---------|-----------|-----------|-----------|
| Works on | Arrays | Any iterable | Objects |
| Returns values | ✅ | ✅ | Keys only |
| `break`/`continue` | ❌ | ✅ | ✅ |
| `async/await` | ❌ problematic | ✅ works | ✅ |
| Skip prototype props | ✅ | ✅ | ❌ need `hasOwnProperty` |

**Rule of thumb:**
- Side effects with no early exit → `forEach`
- Need early exit or `async/await` → `for...of`
- Object property iteration → `for...in` (or `Object.keys/entries`)

---

## Q74. When to use `for...of` vs `forEach()`?

```js
// Use for...of when:
// 1. You need break/continue
for (const user of users) {
  if (user.isBlocked) break;
  await processUser(user);   // 2. You use await
}

// Use forEach when:
// 1. Simple side effects, no early exit needed
users.forEach(u => console.log(u.name));
users.forEach(u => analyticsService.track(u));
```

**The most important rule:** Never use `forEach` with `async/await` if you need sequential execution or to wait for all operations to finish. Use `for...of` for sequential, `Promise.all(arr.map(...))` for parallel.

---

## Q75. What are Functions? Types of Functions?

> See JSQA.md Q12 for the full breakdown. Key additions:

### Generator functions
```js
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}
const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2
```

### Async functions
```js
async function fetchUser(id) {
  const res = await fetch(`/users/${id}`);
  return res.json();
}
// async function always returns a Promise
```

### Function composition
```js
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const transform = pipe(trim, toLowerCase, removeSpaces);
```

---

## Q76. Named vs Anonymous Functions — When to Use Which?

```js
// Named function — has a name, appears in stack traces
function calculateTax(amount) { return amount * 0.18; }

// Anonymous function expression — no name
const calculateTax = function(amount) { return amount * 0.18; };

// Named function expression — has a name but assigned to variable
const calculateTax = function calcTax(amount) { return amount * 0.18; };
// calcTax is only accessible inside the function (useful for recursion)
```

**Prefer named functions when:**
- Business logic that needs to be reusable and debuggable
- The function name clarifies its intent
- You need the function in stack traces (easier debugging)

**Use anonymous/arrow when:**
- Callbacks and one-liners: `.map(x => x * 2)`
- Event handlers: `onClick={() => setOpen(true)}`
- Short-lived, context-specific logic

---

## Q77. What is a Function Expression?

```js
// Function Declaration — hoisted, callable before definition
greet(); // ✅ works
function greet() { return "Hello"; }

// Function Expression — NOT hoisted
greet2(); // ❌ TypeError: greet2 is not a function
const greet2 = function() { return "Hello"; };
```

**Why use function expressions?**
- Assign conditionally: `const fn = isAdmin ? adminFn : userFn`
- Pass directly as argument: `arr.sort(function(a,b){ return a-b; })`
- Prevents accidental early calls (forces intentional usage after definition)

---

## Q78. What are Arrow Functions?

> See JSQA.md Q13 for the full breakdown. Key additions:

### Returning object literals (a common gotcha)
```js
// ❌ Wraps in block — returns undefined
const getUser = name => { name: name };

// ✅ Wrap object in parentheses
const getUser = name => ({ name: name });
const getUser = name => ({ name });  // shorthand
```

### Single parameter — parentheses optional
```js
const double = n => n * 2;      // ✅ one param, no parens needed
const add = (a, b) => a + b;    // requires parens for 2+ params
const noop = () => {};           // requires parens for zero params
```

---

## Q79. What are Callback Functions?

> See JSQA.md Q12 and phase3 Q75. Key depth:

### Callback Hell — why it happens
```js
// Pyramid of doom — deeply nested, hard to read and handle errors
login(user, function(err, session) {
  if (err) return handleError(err);
  fetchProfile(session, function(err, profile) {
    if (err) return handleError(err);
    fetchOrders(profile.id, function(err, orders) {
      if (err) return handleError(err);
      render(orders);
    });
  });
});
```

### Solutions in order of evolution
```js
// 1. Named functions (reduces nesting, not indentation)
login(user, onLogin);

// 2. Promises
login(user).then(fetchProfile).then(fetchOrders).then(render).catch(handleError);

// 3. Async/Await (best)
try {
  const session = await login(user);
  const profile = await fetchProfile(session);
  const orders  = await fetchOrders(profile.id);
  render(orders);
} catch (err) { handleError(err); }
```

---

## Q80. What are Higher-Order Functions (HOF)?

A function is higher-order if it **accepts a function as an argument** or **returns a function**.

```js
// Accepts function — map, filter, reduce are all HOFs
[1,2,3].map(n => n * 2);

// Returns function — factory / currying pattern
function multiplier(factor) {
  return n => n * factor;  // returns a function
}
const double = multiplier(2);
const triple = multiplier(3);
double(5); // 10
triple(5); // 15

// Both — middleware pattern
function withLogging(fn) {
  return function(...args) {
    console.log("Calling with", args);
    const result = fn(...args);
    console.log("Result:", result);
    return result;
  };
}
const loggedAdd = withLogging((a, b) => a + b);
loggedAdd(2, 3); // logs + returns 5
```

**Common HOFs in JavaScript:** `map`, `filter`, `reduce`, `forEach`, `sort`, `setTimeout`, `addEventListener` — all accept functions as arguments.

---

## Q81. Difference between Parameters and Arguments?

```js
function add(a, b) { return a + b; }  // a, b are PARAMETERS (definition)
add(10, 20);                           // 10, 20 are ARGUMENTS (call)
```

### Default parameters (ES6)
```js
function greet(name = "Guest") { return `Hello ${name}`; }
greet();           // "Hello Guest"
greet("Alice");    // "Hello Alice"
greet(undefined);  // "Hello Guest" — undefined triggers default
greet(null);       // "Hello null"  — null does NOT trigger default
```

### Missing/extra arguments
```js
function show(a, b, c) { console.log(a, b, c); }
show(1, 2);       // 1 2 undefined — missing args become undefined
show(1, 2, 3, 4); // 1 2 3 — extra args are ignored (accessible via `arguments`)
```

---

## Q82. Ways to Pass Arguments to a Function?

```js
// 1. Positional — simple but fragile for many args
createUser("Alice", "alice@example.com", "admin", true);

// 2. Object parameter — readable, order-independent, extensible
function createUser({ name, email, role = "user", active = true }) {
  // ...
}
createUser({ name: "Alice", email: "alice@example.com", role: "admin" });

// 3. Rest parameters — variable number of args
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
sum(1, 2, 3, 4, 5); // 15

// 4. arguments object (legacy, non-arrow functions only)
function legacy() { console.log(arguments[0]); }
```

**Best practice for 4+ parameters:** Use an options object. It's self-documenting, easy to extend, and callers don't need to remember argument order.

---

## Q83. What are Default Parameters?

```js
// ES5 way (problematic — fails for 0, false, "")
function greet(name) { name = name || "Guest"; }

// ES6 default parameters (correct)
function greet(name = "Guest") { return `Hello ${name}`; }

// Defaults can use previous parameters
function box(width = 100, height = width) { return width * height; }
box();        // 10000
box(200);     // 40000
box(200, 50); // 10000

// Default can call functions
function getId(id = generateId()) { return id; }
```

**Key rule:** Defaults only apply when the argument is `undefined`. Passing `null` explicitly does NOT trigger the default.

---

## Q84. What is Event Handling in JavaScript?

```js
const btn = document.getElementById("save");

// addEventListener — preferred (multiple listeners, removable)
btn.addEventListener("click", handleSave);
btn.addEventListener("click", logClick);  // second listener works too

// removeEventListener — cleanup (must pass same function reference)
btn.removeEventListener("click", handleSave);

// Inline HTML — avoid in modern code
// <button onclick="save()"> — mixes HTML and JS, hard to maintain

// Event object
btn.addEventListener("click", function(event) {
  event.preventDefault();   // stop default browser behavior
  event.stopPropagation();  // stop event bubbling
  console.log(event.target, event.type);
});
```

**React cleanup pattern:**
```js
useEffect(() => {
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize);
}, []);
```

---

## Q85. What are First-Class Functions?

JavaScript functions are **first-class citizens** — they can be:
1. Assigned to variables
2. Passed as arguments
3. Returned from functions
4. Stored in arrays/objects

```js
// 1. Assigned
const greet = function() { return "Hello"; };

// 2. Passed
[1,2,3].map(n => n * 2);
setTimeout(() => console.log("done"), 1000);

// 3. Returned
function makeMultiplier(x) { return y => x * y; }

// 4. Stored
const handlers = {
  save: () => saveToDB(),
  cancel: () => resetForm(),
};
handlers.save();
```

This is the foundation that makes callbacks, HOFs, closures, currying, and functional programming patterns possible in JavaScript.

---

## Q86. Pure vs Impure Functions?

> See JSQA.md Q12. Key additions:

### Identifying side effects
```js
// Side effects include:
// - Modifying external variables
// - DOM manipulation
// - API calls / network requests
// - Writing to localStorage / file system
// - console.log (technically a side effect)
// - Math.random(), Date.now() (non-deterministic)

// Pure
const add = (a, b) => a + b;

// Impure — reads external state
let rate = 0.18;
const tax = amount => amount * rate;  // depends on external `rate`

// Impure — modifies argument
function addItem(cart, item) {
  cart.push(item);  // mutates the cart parameter!
  return cart;
}

// Pure equivalent
function addItem(cart, item) {
  return [...cart, item];  // returns new array
}
```

**React implication:** Component render functions should be pure. Side effects belong in `useEffect`.

---

## Q87. What is Function Currying?

> See JSQA.md Q12. Deeper dive:

```js
// Basic currying
const multiply = a => b => a * b;
const double = multiply(2);  // partially applied
double(5);  // 10
double(10); // 20

// Real-world: event handler factory
const handleInput = field => event => {
  setForm(prev => ({ ...prev, [field]: event.target.value }));
};
<input onChange={handleInput("email")} />
<input onChange={handleInput("password")} />

// Generic curry utility
const curry = fn => {
  const arity = fn.length;
  return function curried(...args) {
    return args.length >= arity
      ? fn(...args)
      : (...more) => curried(...args, ...more);
  };
};
```

**Currying vs Partial Application:**
- **Currying** always produces single-argument functions chained: `f(a)(b)(c)`
- **Partial application** fixes some arguments but doesn't require one at a time: `f(a, b)(c)`

---

## Q88. What are `call()`, `apply()`, and `bind()`?

> See JSQA.md Q12 and Q88. Key additions:

### Real interview output question
```js
const obj = { value: 42 };

function getValue() {
  return this.value;
}

getValue();               // undefined (window.value)
getValue.call(obj);       // 42
getValue.apply(obj);      // 42
const bound = getValue.bind(obj);
bound();                  // 42

// Arrow function — call/apply/bind have NO EFFECT on this
const arrow = () => this.value;
arrow.call(obj);          // undefined — lexical this ignores bind
```

### Method borrowing pattern
```js
const arrayLike = { 0: "a", 1: "b", length: 2 };
Array.prototype.join.call(arrayLike, "-");  // "a-b"
```

---

## Q89. What is a String?

> See JSQA.md Q4 for string methods. Key additions for this phase:

```js
// String is a primitive
typeof "hello"     // "string"
typeof new String("hello") // "object" — avoid wrapper objects

// Strings auto-box to String object for method access
"hello".toUpperCase(); // works — JS temporarily wraps it

// String comparison
"apple" < "banana"   // true — lexicographic (character code comparison)
"Z" < "a"            // true — uppercase letters have lower char codes
"10" < "9"           // true — lexicographic, not numeric!
Number("10") < 9     // false — numeric comparison
```

---

## Q90. What are Template Literals and String Interpolation?

> See JSQA.md Q4 and phase4 for full coverage. Key additions:

### Tagged template literals
```js
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) =>
    `${result}${str}${values[i] ? `<b>${values[i]}</b>` : ""}`, "");
}
const name = "Alice";
highlight`Hello ${name}, you have ${5} messages`;
// "Hello <b>Alice</b>, you have <b>5</b> messages"
```

Used by: Styled Components, GraphQL (`gql`), SQL template libraries.

---

## Q91. Difference between `slice()`, `substring()`, and `substr()`?

```js
const str = "JavaScript";

// slice(start, end) — supports negative indexes, end is exclusive
str.slice(0, 4);    // "Java"
str.slice(-6);      // "Script" — 6 from end
str.slice(4, 0);    // "" — if start > end, returns empty

// substring(start, end) — no negative indexes, swaps if start > end
str.substring(0, 4);  // "Java"
str.substring(-3);    // "JavaScript" — negative treated as 0
str.substring(4, 0);  // "Java" — swaps arguments, same as (0,4)

// substr(start, length) — DEPRECATED, second arg is LENGTH not end index
str.substr(4, 6);   // "Script" — start at 4, take 6 characters
```

**Use `slice()` exclusively** — it's the most consistent and modern. `substr()` is deprecated. `substring()` has counterintuitive negative-index behavior.

---

## Q92. Important String Methods?

> See JSQA.md Q4 for the full table. Additional methods for 4.5 YOE:

```js
const str = "  Hello, World!  ";

// Padding
"5".padStart(3, "0");   // "005" — useful for IDs, timestamps
"5".padEnd(3, "0");     // "500"

// Repeat
"ab".repeat(3);          // "ababab"

// At (ES2022)
"hello".at(-1);          // "o" — cleaner than str[str.length-1]

// matchAll — find all regex matches with details
const matches = [...str.matchAll(/\w+/g)];

// replaceAll
"a-b-c".replaceAll("-", "_");  // "a_b_c"

// startsWith / endsWith
"hello.js".endsWith(".js");    // true — useful for file type checks
"https://".startsWith("https"); // true
```

---

## Q93. Difference between `indexOf()` and `includes()` in Strings?

```js
const str = "Frontend Developer";

// indexOf — returns position
str.indexOf("Developer");  // 9
str.indexOf("React");      // -1

// includes — returns boolean (cleaner for existence checks)
str.includes("Developer"); // true
str.includes("React");     // false

// Both support a start position
str.indexOf("e", 5);       // search from index 5
str.includes("e", 5);      // search from index 5
```

**Prefer `includes()`** for readability when you only need to know if the substring exists.

---

## Q94. What is the `split()` Method?

```js
// Basic split
"HTML,CSS,JS".split(",");          // ["HTML","CSS","JS"]
"hello".split("");                  // ["h","e","l","l","o"]
"hello world".split(" ");           // ["hello","world"]

// Limit the result
"a,b,c,d".split(",", 2);           // ["a","b"] — max 2 elements

// Split with regex
"one1two2three".split(/\d/);       // ["one","two","three"]

// Reverse a string using split
"hello".split("").reverse().join(""); // "olleh"

// URL parsing
"/users/101/profile".split("/").filter(Boolean); // ["users","101","profile"]
```

`split()` → String to Array. `join()` → Array to String. They are inverses.

---

## Q95. Difference between `replace()` and `replaceAll()`?

```js
const str = "cat and cat and cat";

str.replace("cat", "dog");     // "dog and cat and cat" — first only
str.replaceAll("cat", "dog");  // "dog and dog and dog" — all

// replace() with regex + global flag — replaces all
str.replace(/cat/g, "dog");   // "dog and dog and dog"

// With function (powerful)
"hello world".replace(/\w+/g, word => word.toUpperCase());
// "HELLO WORLD"
```

**Before `replaceAll` (ES2021):** the pattern was `replace(/text/g, replacement)`. Both are valid; `replaceAll` is more readable for simple string replacements.

---

## Q96. What is RegEx in JavaScript?

```js
// Two ways to create
const pattern1 = /hello/i;          // literal (prefer — compiled at load time)
const pattern2 = new RegExp("hello", "i");  // constructor (use for dynamic patterns)

// Common methods
/\d+/.test("abc123");              // true — does it match?
"hello world".match(/\w+/g);       // ["hello","world"]
"hello world".replace(/world/, "JS"); // "hello JS"
"a1b2c3".split(/\d/);              // ["a","b","c",""]

// Common patterns
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const phoneRegex = /^\+?[\d\s\-()]{10,}$/;
const urlRegex   = /^https?:\/\/.+/;
const digitsOnly = /^\d+$/;
const alphaOnly  = /^[a-zA-Z]+$/;
```

### Flags
| Flag | Meaning |
|------|---------|
| `g` | Global — find all matches |
| `i` | Case insensitive |
| `m` | Multiline — `^` and `$` match line boundaries |
| `s` | Dotall — `.` matches newlines too |

---

## Q97. Common RegEx Patterns for Frontend Interviews?

```js
// Email
/^[^\s@]+@[^\s@]+\.[^\s@]+$/

// Password: min 8 chars, at least one uppercase, one digit
/^(?=.*[A-Z])(?=.*\d).{8,}$/

// Only digits
/^\d+$/

// Only alphabets
/^[A-Za-z]+$/

// Remove all whitespace
str.replace(/\s+/g, "")

// Extract numbers from string
"Order 123, Item 456".match(/\d+/g)  // ["123","456"]

// Validate URL
/^https?:\/\/([\w.-]+)(\/[\w./?%&=-]*)?$/
```

**Interview tip:** For validation questions, explain the pattern structure — interviewers care more about your regex literacy than memorization. Know what `^`, `$`, `+`, `*`, `?`, `\d`, `\w`, `\s`, `(?=...)` mean.

---

## Q98. What are Objects in JavaScript?

> See JSQA.md Q15 for the core. Key additions for this phase:

### Object shorthand and computed properties (ES6)
```js
const name = "Alice";
const age = 30;

// Shorthand — when key equals variable name
const user = { name, age };  // same as { name: name, age: age }

// Computed property names
const field = "email";
const obj = { [field]: "alice@example.com" };  // { email: "alice@example.com" }

// Method shorthand
const api = {
  getData() { },          // instead of getData: function() {}
  async fetchUsers() { }, // async method
  get count() { },        // getter
  set count(v) { },       // setter
};
```

### Object.fromEntries — reverse of Object.entries
```js
const entries = [["name", "Alice"], ["age", 30]];
Object.fromEntries(entries); // { name: "Alice", age: 30 }

// Transform object values
const prices = { apple: 1, banana: 2 };
const doubled = Object.fromEntries(
  Object.entries(prices).map(([k, v]) => [k, v * 2])
);
// { apple: 2, banana: 4 }
```

---

## Q99. How to Create Objects?

> See JSQA.md Q15. Key addition — when to use each pattern:

```js
// 1. Object literal — use for plain data, config, one-off objects
const config = { theme: "dark", lang: "en" };

// 2. Factory function — use when you need many similar objects without class overhead
function createUser(name, role) {
  return {
    name,
    role,
    greet() { return `Hello, I'm ${name}`; }
  };
}

// 3. Class — use for complex objects needing inheritance, private state, or TypeScript
class User {
  #id;  // private field (ES2022)
  constructor(name) { this.name = name; this.#id = Math.random(); }
  getId() { return this.#id; }
}

// 4. Object.create() — use for prototype-based inheritance
const animal = { breathe() { console.log("breathing"); } };
const dog = Object.create(animal);
dog.bark = () => console.log("woof");
dog.breathe(); // inherited
```

---

## Q100. Different Ways to Access Object Properties?

> See JSQA.md Q15. Key additions:

```js
const user = { name: "Alice", "home city": "Pune" };

// Dot notation — use when key is a valid identifier and known at code time
user.name;

// Bracket notation — required for:
user["home city"];     // keys with spaces or special characters
const key = "name";
user[key];             // dynamic key stored in variable
user["name"];          // also valid, same as dot

// Optional chaining — avoid TypeError on null/undefined
user?.address?.city;   // undefined instead of TypeError

// Destructuring — extract multiple keys at once
const { name, address: { city = "Unknown" } = {} } = user;

// Dynamic access pattern common in forms
function handleChange(field, value) {
  setForm(prev => ({ ...prev, [field]: value }));
}
```
