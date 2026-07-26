# JavaScript Interview Q&A

> Concise, concept-clear answers for frontend interviews.

---

## Q1. What is JavaScript? What is the role of a JavaScript Engine?

**JavaScript** is a high-level, single-threaded, dynamically-typed, JIT-compiled programming language used to make web pages interactive. It runs in browsers and on servers via Node.js.

**JavaScript Engine** is responsible for parsing, compiling, optimizing, and executing JS code. Each browser ships its own engine.

| Browser | Engine |
|---------|--------|
| Chrome / Edge | V8 |
| Firefox | SpiderMonkey |
| Safari | JavaScriptCore |

### How the V8 Engine executes your code (pipeline)

```
Source Code
    ↓
Parser → AST (Abstract Syntax Tree)
    ↓
Ignition (Interpreter) → Bytecode  ← fast startup
    ↓
TurboFan (JIT Compiler) → Optimized Machine Code  ← hot code paths
```

1. **Parser** reads the source and builds an **AST** — a tree structure representing the code's syntax
2. **Ignition** (V8's interpreter) converts AST to **bytecode** and starts executing immediately — this gives fast startup time
3. **TurboFan** (V8's optimizing compiler) watches for "hot" functions (called repeatedly), re-compiles them into highly **optimized machine code**
4. If assumptions break (e.g. a variable's type changes), TurboFan **deoptimizes** and falls back to bytecode

### Why this matters in practice

```js
// V8 optimizes this — consistent number type
function add(a, b) { return a + b; }
add(1, 2);   // V8 assumes: always numbers → optimizes
add("x", 2); // type changed → deoptimization triggered → slower
```

Keeping **consistent types** in hot functions helps V8 keep them optimized — relevant in performance-critical React render loops.

**Key points:**
- JS is **single-threaded** — one call stack, one thing at a time
- JIT = compiled *during* execution, not ahead of time like Java/C++
- V8 powers both Chrome and Node.js — same engine, different environments
- **Deoptimization** is a real performance concern in tight loops with mixed types

**Common trap:** "Is JS interpreted or compiled?" → Neither purely. It's **JIT-compiled**: starts as interpreted bytecode, hot paths get compiled to machine code at runtime.

---

## Q2. What are Client Side and Server Side?

**Client-side** = code that runs in the **user's browser** (HTML, CSS, JavaScript, React, Angular).  
**Server-side** = code that runs on the **backend server** (Node.js, Spring Boot, .NET).

| | Client Side | Server Side |
|---|---|---|
| Runs in | Browser | Server |
| Handles | UI, events, API calls | Business logic, DB, auth |
| Examples | React, Angular | Node.js, Spring Boot |

**Key points:**
- Frontend validation can be bypassed — always re-validate on the backend
- Client and server communicate via **APIs (JSON)**
- React/Angular = client side. Node.js/Spring Boot = server side

**Common trap:** "Can React replace the backend?" → No. React handles UI only.

---

## Q3. What are Variables? Difference between `var`, `let`, and `const`?

Variables are containers for storing data. JS has three declaration keywords:

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Reassign | ✅ | ✅ | ❌ |
| Redeclare | ✅ | ❌ | ❌ |
| Hoisting | `undefined` | TDZ error | TDZ error |

```js
var x = 1;   // function-scoped, avoid in modern code
let y = 2;   // block-scoped, use when value changes
const z = 3; // block-scoped, use by default
```

**Key points:**
- Prefer `const` by default; use `let` only when reassignment is needed
- `const` on objects/arrays only protects the **reference**, not the contents
- `var` leaks out of `if`/`for` blocks — major source of bugs

**Common trap:** `const arr = []; arr.push(1)` → No error. The array contents can change; the reference cannot.

---

## Q4. What are important String operations in JavaScript?

Strings are **immutable** sequences of characters. Any method returns a new string — the original is unchanged.

| Method | Purpose | Example |
|--------|---------|---------|
| `length` | Character count | `"JS".length` → `2` |
| `toUpperCase()` / `toLowerCase()` | Case conversion | `"hi".toUpperCase()` → `"HI"` |
| `includes(str)` | Check substring | `"Hello".includes("ell")` → `true` |
| `slice(start, end)` | Extract part | `"JavaScript".slice(0,4)` → `"Java"` |
| `replace(old, new)` | Replace text | `"Hello Angular".replace("Angular","React")` |
| `split(sep)` | String → Array | `"a,b,c".split(",")` → `["a","b","c"]` |
| `trim()` | Remove whitespace | `"  hi  ".trim()` → `"hi"` |
| `indexOf(str)` | Find position | `"hello".indexOf("e")` → `1` |

```js
// Real-world: case-insensitive search
users.filter(u => u.name.toLowerCase().includes(query.toLowerCase()));
```

**Common traps:**
- `"5" + 5` → `"55"` (concatenation, not addition)
- `slice()` supports negative indexes; `substring()` does not
- `split()` → String to Array; `join()` → Array to String

---

## Q5. What is the DOM? Difference between HTML and DOM?

**HTML** is the static source markup written by developers.  
**DOM (Document Object Model)** is the **live, in-memory tree** the browser builds from HTML — JavaScript interacts with this, not the file.

```
Document → html → body → h1
                        → p
```

```js
// JS modifies DOM, not the HTML file
document.querySelector("h1").textContent = "Updated";
```

**Key points:**
- DOM is a live object tree; HTML is just text
- React uses a **Virtual DOM** — a lightweight copy of the real DOM to batch and minimize expensive updates
- Direct DOM manipulation triggers **reflow/repaint** — avoid doing it frequently

**Common trap:** "Does JS modify the HTML file?" → No. It modifies the in-memory DOM representation.

---

## Q6. What are Selectors in JavaScript?

Selectors are DOM methods to find elements on the page.

| Method | Returns | Notes |
|--------|---------|-------|
| `getElementById(id)` | Single element | Fastest lookup |
| `getElementsByClassName(cls)` | HTMLCollection (live) | No `#` prefix |
| `getElementsByTagName(tag)` | HTMLCollection (live) | e.g. `"div"` |
| `querySelector(css)` | First match | Supports any CSS selector |
| `querySelectorAll(css)` | NodeList (static) | Supports any CSS selector |

```js
document.querySelector(".card");           // first .card
document.querySelectorAll(".card");        // all .cards (NodeList)
document.getElementById("title");         // no # needed here
```

**Key points:**
- `querySelector` / `querySelectorAll` are preferred — they accept full CSS selectors
- `NodeList` supports `forEach`; `HTMLCollection` does not (in older browsers)
- `querySelector` returns **only the first match** — common mistake to expect all elements

**Common trap:** `getElementById("#id")` → wrong. The method does not take `#`.

---

## Q7. Difference between `getElementById`, `getElementsByClassName`, and `getElementsByTagName`?

| Method | Selector Basis | Returns | Live? |
|--------|---------------|---------|-------|
| `getElementById` | ID (unique) | Single Element | N/A |
| `getElementsByClassName` | Class name | HTMLCollection | ✅ Yes |
| `getElementsByTagName` | HTML tag | HTMLCollection | ✅ Yes |

```js
document.getElementById("header");          // one element
document.getElementsByClassName("card");    // all elements with class "card"
document.getElementsByTagName("input");     // all <input> tags
```

**Key points:**
- A **live collection** auto-updates when the DOM changes
- `getElementById` is the fastest because browsers index by ID
- IDs must be unique per HTML spec
- `forEach` on `HTMLCollection` fails in older browsers — convert with `Array.from()` first

**Common trap:** These return HTMLCollection, not an Array. `collection.forEach()` may throw in older environments.

---

## Q8. What are Data Types in JavaScript?

JavaScript has **7 primitive** and **non-primitive (reference)** types.

### Primitive (stored by value)
| Type | Example |
|------|---------|
| Number | `let age = 30` |
| String | `let name = "JS"` |
| Boolean | `let active = true` |
| Undefined | `let x` |
| Null | `let data = null` |
| Symbol | `Symbol("id")` |
| BigInt | `9007199254740991n` |

### Non-Primitive (stored by reference)
| Type | Example |
|------|---------|
| Object | `{ name: "JS" }` |
| Array | `[1, 2, 3]` |
| Function | `function fn() {}` |

**Key points:**
- Primitives are **immutable** and copied by value
- Objects/Arrays are **mutable** and copied by reference
- Functions are first-class objects in JavaScript

**Common traps:**
- `typeof null` → `"object"` — a historic JS bug, not a real type
- `typeof []` → `"object"` — use `Array.isArray()` to check for arrays

---

## Q9. What are Operators? Types of Operators in JavaScript?

Operators perform actions on values. Key categories:

| Category | Operators | Example |
|----------|-----------|---------|
| Arithmetic | `+ - * / % **` | `10 % 3` → `1` |
| Assignment | `= += -= *= /=` | `count += 5` |
| Comparison | `== === != !== > < >= <=` | `5 === "5"` → `false` |
| Logical | `&& \|\| !` | `isLoggedIn && isAdmin` |
| Ternary | `? :` | `age >= 18 ? "Adult" : "Minor"` |
| Type | `typeof instanceof` | `typeof "hi"` → `"string"` |
| String | `+` | `"Hello" + " World"` |

**Key points:**
- `===` (strict) is preferred over `==` (loose) — avoids type coercion bugs
- `&&` and `||` support **short-circuit evaluation**
- `+` is overloaded: addition for numbers, concatenation for strings

**Common traps:**
- `"5" + 5` → `"55"` (string wins with `+`)
- `"5" - 5` → `0` (numeric coercion with `-`)
- `true + true` → `2` (booleans coerce to numbers)

---

## Q10. What are Conditional Statements in JavaScript?

Conditional statements execute different code blocks based on conditions.

### `if / else if / else`
Use for **complex, multi-branch logic**.
```js
if (marks >= 90) {
  grade = "A";
} else if (marks >= 75) {
  grade = "B";
} else {
  grade = "C";
}
```

### `switch`
Use when **one variable is compared against many values**.
```js
switch (role) {
  case "admin":  grantFullAccess(); break;
  case "user":   grantLimitedAccess(); break;
  default:       denyAccess();
}
```

### Ternary Operator
Use for **simple value assignment or JSX rendering**.
```js
const label = isActive ? "Active" : "Inactive";
// React
{isLoading ? <Loader /> : <Dashboard />}
```

**Key points:**
- Ternary keeps JSX clean for single-condition rendering
- `switch` without `break` causes **fall-through** (next case executes too)
- Avoid deeply nested ternaries — use `if-else` for clarity

**Common trap:** `if (a = 5)` — assignment, not comparison. Use `if (a === 5)`.

---

## Q11. What is a Loop? Types of Loops in JavaScript?

Loops execute a block of code repeatedly until a condition becomes false.

| Loop | Use When |
|------|----------|
| `for` | Number of iterations is known |
| `while` | Condition-based, iterations unknown |
| `do...while` | Must execute at least once |
| `for...of` | Iterate **values** of arrays/strings/iterables |
| `for...in` | Iterate **keys** of an object |

```js
// for
for (let i = 0; i < 5; i++) { console.log(i); }

// while
let i = 0;
while (i < 5) { console.log(i); i++; }

// do...while — runs once even if condition is false
let x = 10;
do { console.log(x); } while (x < 5); // prints 10

// for...of — values
for (const skill of ["JS", "React"]) { console.log(skill); }

// for...in — keys
for (const key in { name: "JS", age: 10 }) { console.log(key); }
```

**Key points:**
- `for...of` → use on **arrays**; `for...in` → use on **objects**
- In React, `map()` / `forEach()` / `filter()` are preferred over raw loops for rendering
- `do...while` guarantees at least one execution

**Common traps:**
- Using `for...in` on arrays gives indexes as strings, not values — avoid it
- `while (true)` without a break → infinite loop, crashes the browser

---

## Q12. What are Functions in JavaScript? Types of Functions?

A function is a reusable block of code that performs a specific task.

### Types

```js
// 1. Function Declaration — hoisted, callable before definition
function greet() { return "Hello"; }

// 2. Function Expression — not hoisted
const greet = function() { return "Hello"; };

// 3. Arrow Function — concise, no own `this`
const greet = () => "Hello";

// 4. IIFE — runs immediately, used to create isolated scope
(function() { console.log("Runs once"); })();

// 5. Callback — passed as argument to another function
setTimeout(function() { console.log("Done"); }, 1000);

// 6. Higher-Order Function — takes or returns a function
function execute(fn) { fn(); }
```

**Key points:**
- **Function Declaration** is fully hoisted; **Function Expression** is not
- Functions are **first-class citizens** — they can be assigned to variables, passed as arguments, and returned from other functions
- Arrow functions don't have their own `this`, `arguments`, or `prototype`

**Common traps:**
- Forgetting `return` in a block-body arrow function: `(a, b) => { a + b }` returns `undefined`
- `parameter` = variable in definition; `argument` = value passed during call

---

## Q13. What are Arrow Functions? What is their difference from regular functions?

Arrow functions (ES6) offer a shorter syntax and **lexically bind `this`** — they don't create their own `this`.

```js
// Regular
function add(a, b) { return a + b; }

// Arrow — single expression, implicit return
const add = (a, b) => a + b;
```

### The critical `this` difference

```js
const person = {
  name: "Alice",
  // Regular: this = the object
  greetRegular: function() { console.log(this.name); }, // "Alice"
  // Arrow: this = outer (window/undefined in strict)
  greetArrow: () => { console.log(this.name); }         // undefined
};
```

| Feature | Regular Function | Arrow Function |
|---------|-----------------|----------------|
| `this` | Dynamic (caller) | Lexical (outer scope) |
| `arguments` object | ✅ Yes | ❌ No |
| Can use `new` | ✅ Yes | ❌ No |
| Hoisted | Declaration: ✅ | ❌ No |

**Key points:**
- Arrow functions are ideal for callbacks and array methods (`map`, `filter`, `reduce`)
- React event handlers use arrow functions to avoid `this` binding issues
- Cannot be used as **constructors** or **object methods** that rely on `this`

**Common trap:** Using an arrow function as an object method expecting `this` to point to the object — it won't.

---

## Q14. What are Arrays? How to Get, Add, and Remove Elements?

An array is an **ordered, zero-indexed, mutable** collection that can hold mixed types.

```js
const users = ["Alice", "Bob", "Carol"];
users[0];        // "Alice"  — access by index
users.length;    // 3
```

### Add / Remove

| Method | Action | Mutates? |
|--------|--------|----------|
| `push(val)` | Add to end | ✅ |
| `pop()` | Remove from end | ✅ |
| `unshift(val)` | Add to start | ✅ |
| `shift()` | Remove from start | ✅ |
| `splice(i, n)` | Remove/insert at index | ✅ |
| `slice(start, end)` | Copy a portion | ❌ |

### Transformation / Search

| Method | Returns | Use |
|--------|---------|-----|
| `map(fn)` | New array | Transform each element |
| `filter(fn)` | New array | Keep matching elements |
| `find(fn)` | First match | Find one element |
| `reduce(fn, init)` | Single value | Aggregate |
| `some(fn)` | Boolean | At least one match |
| `every(fn)` | Boolean | All match |
| `includes(val)` | Boolean | Value exists |

```js
const nums = [1, 2, 3, 4, 5];
nums.map(n => n * 2);          // [2, 4, 6, 8, 10]
nums.filter(n => n > 2);       // [3, 4, 5]
nums.find(n => n > 3);         // 4
nums.reduce((sum, n) => sum + n, 0); // 15
```

**Key points:**
- `typeof []` → `"object"` — use `Array.isArray(arr)` to check
- `map` returns a new array; `forEach` returns `undefined`
- In React, **never mutate state arrays directly** — use spread: `[...users, newUser]`

---

## Q15. What are Objects in JavaScript?

An object is a collection of **key-value pairs** used to represent real-world entities.

```js
const user = {
  name: "Alice",
  age: 30,
  greet() { return `Hello, ${this.name}`; }  // method
};

// Access
user.name;           // dot notation
user["name"];        // bracket notation (use for dynamic keys)

// Add / Modify / Delete
user.role = "Admin"; // add
user.age = 31;       // modify
delete user.role;    // delete
```

### Useful Patterns

```js
// Nested object
const emp = { address: { city: "Pune" } };
emp.address.city;  // "Pune"

// Check if key exists
"name" in user;               // true
user.hasOwnProperty("name");  // true

// Object methods
Object.keys(user);    // ["name", "age"]
Object.values(user);  // ["Alice", 30]
Object.entries(user); // [["name","Alice"], ["age",30]]
```

**Key points:**
- Objects are **stored by reference** — two variables can point to the same object
- `{} === {}` → `false` (different references, even with same content)
- Copying with `const obj2 = obj1` creates a reference, not a clone — use `{...obj1}` or `Object.assign()`

**Common trap:** In React, always create a new object for state updates: `setUser({ ...user, name: "New" })` — never mutate the existing one.

---

## Q16. What is Scope in JavaScript?

Scope defines **where a variable is accessible** in your code. JavaScript uses **lexical (static) scoping** — the scope of a variable is determined by where it is **written** in the source code, not where it is called from.

| Scope | Created by | Accessible |
|-------|-----------|------------|
| **Global** | Outside any function/block | Everywhere |
| **Function** | Inside a `function` | Only within that function |
| **Block** | Inside `{}` (`if`, `for`, etc.) | Only within that block (`let`/`const` only) |
| **Module** | ES Module (`import`/`export`) | Only within that file |

```js
const global = "global";     // accessible everywhere

function greet() {
  const local = "local";     // only inside greet()

  if (true) {
    let block = "block";     // only inside this if block
    var leaked = "leaked";   // var ignores block — hoisted to function scope
  }

  console.log(leaked);  // ✅ "leaked" — var escapes the block
  console.log(block);   // ❌ ReferenceError — let is block-scoped
}
```

---

### Scope Chain

When JS looks up a variable, it starts in the **current scope** and walks up through parent scopes until it finds the variable or hits the global scope. This chain of scopes is the **scope chain**.

```js
const x = "global";

function outer() {
  const x = "outer";

  function inner() {
    const x = "inner";
    console.log(x); // "inner" — found in own scope, stops here
  }

  function middle() {
    // no x here — walks up to outer
    console.log(x); // "outer" — found in parent scope
  }

  inner();   // "inner"
  middle();  // "outer"
}

outer();
console.log(x); // "global"
```

The lookup always goes **inward → outward**, never the reverse. An outer function cannot access variables defined inside an inner function.

---

### Lexical Environment

Internally, every time a function or block is entered, JS creates a **Lexical Environment** — a record of:
1. All variable/function bindings in that scope (**Environment Record**)
2. A reference to the **outer (parent) Lexical Environment**

```
Global Lexical Environment
  ├── x: "global"
  └── outer ref: null

outer() Lexical Environment
  ├── x: "outer"
  └── outer ref → Global Lexical Environment

inner() Lexical Environment
  ├── x: "inner"
  └── outer ref → outer() Lexical Environment
```

This chain of Lexical Environments **is** the scope chain. Closures work by holding a reference to their outer Lexical Environment.

---

### Lexical scope vs dynamic scope

JavaScript uses **lexical scope** — scope is fixed at write time. Some other languages (older Lisps, bash) use dynamic scope where scope is determined at call time.

```js
const name = "global";

function printName() {
  console.log(name); // always looks at where printName was DEFINED, not called
}

function callIt() {
  const name = "local";
  printName(); // still prints "global" — lexical, not dynamic
}

callIt(); // "global"
```

---

**Key points:**
- `let`/`const` → block-scoped; `var` → function-scoped (ignores `{}` blocks)
- Scope chain is walked at **runtime** for variable lookup, but determined at **parse time** (lexical)
- Each function carries a reference to its outer Lexical Environment — this is the foundation for closures
- **Module scope** (ESM) is between function and global — variables are file-private unless explicitly exported
- Avoid polluting global scope — it causes naming collisions and hard-to-trace bugs in large apps

**Common traps:**
- `var` in a `for` loop leaks: `for (var i = 0; i < 3; i++) {}; console.log(i)` → `3`. Use `let`.
- Shadowing: declaring a variable with the same name in an inner scope hides the outer one — can be intentional or a silent bug

---

## Q17. What is Hoisting in JavaScript?

Hoisting is a result of how JavaScript's engine processes code in **two phases** before execution. Understanding hoisting properly requires understanding the **Execution Context**.

---

### Execution Context — the foundation

Every time JavaScript runs code, it creates an **Execution Context (EC)**. There are two types:

| Type | Created when |
|------|-------------|
| **Global Execution Context (GEC)** | Script first loads — created once |
| **Function Execution Context (FEC)** | Every time a function is called |

Each EC is created in **two phases**:

```
Phase 1 — Creation Phase (Memory Allocation)
  ├── var declarations → allocated, initialized to undefined
  ├── let/const declarations → allocated, placed in TDZ (NOT initialized)
  ├── function declarations → entire function stored in memory
  └── this binding is set

Phase 2 — Execution Phase (Code runs line by line)
  ├── var assignments happen
  ├── let/const initialized (TDZ ends)
  └── function calls push new ECs onto the Call Stack
```

**Hoisting is just the visible effect of the Creation Phase** — declarations are processed before any code runs.

---

### Call Stack

JS manages execution contexts with a **Call Stack** (LIFO). GEC is always at the bottom.

```
function a() {
  b();
}
function b() {
  console.log("b");
}
a();

Call Stack progression:
[ GEC ]                  ← script loads
[ GEC | a() EC ]         ← a() called
[ GEC | a() EC | b() EC ]← b() called inside a()
[ GEC | a() EC ]         ← b() returns, its EC is popped
[ GEC ]                  ← a() returns, its EC is popped
```

---

### What gets hoisted and how

| Declaration | Phase 1 value | Phase 2 |
|-------------|--------------|---------|
| `var` | `undefined` | assigned at declaration line |
| `let` / `const` | TDZ — exists but inaccessible | initialized at declaration line |
| Function Declaration | Full function body | already available |
| Function Expression / Arrow | `undefined` (var) or TDZ (let/const) | assigned at declaration line |

```js
// var — Creation Phase sets it to undefined
console.log(a);  // undefined  (no error — it exists, just not assigned yet)
var a = 10;
console.log(a);  // 10

// Function declaration — fully available in Creation Phase
greet();                          // "Hello" — works before definition
function greet() { console.log("Hello"); }

// let/const — in TDZ during Creation Phase
console.log(b);  // ❌ ReferenceError: Cannot access 'b' before initialization
let b = 5;

// Function Expression — var is hoisted as undefined, not the function
sayHi();         // ❌ TypeError: sayHi is not a function
var sayHi = function() { console.log("Hi"); };
```

---

### Temporal Dead Zone (TDZ)

The TDZ is the gap between the **start of the scope** (Creation Phase) and the **line where `let`/`const` is initialized** (Execution Phase). The variable exists in memory but is locked.

```js
{
  // TDZ for `x` starts here ↓
  console.log(x); // ❌ ReferenceError — in TDZ
  let x = 10;     // TDZ ends here, x = 10
  console.log(x); // ✅ 10
}
```

---

### Each function call gets its own EC

```js
function add(a, b) {
  var result = a + b; // result exists only in this EC
  return result;
}

add(2, 3); // new FEC pushed → result = 5 → EC popped → result gone
add(4, 5); // brand new FEC, fresh result variable
```

This is why function-scoped `var` doesn't leak between calls — each call has its own EC.

---

**Key points:**
- Hoisting is a **side effect of the Creation Phase**, not a physical movement of code
- The **Call Stack** manages which EC is currently executing
- `var` → hoisted with `undefined`; `let`/`const` → hoisted into TDZ (inaccessible)
- Function declarations are the only things **fully** available before their line
- Each function call creates a **new, independent** Execution Context

**Common traps:**
- `let`/`const` ARE hoisted — they're just in TDZ. This is different from not being hoisted at all
- Confusing `undefined` (var hoisting) with `ReferenceError` (TDZ) is the most common hoisting interview mistake
- `sayHi()` before `var sayHi = function(){}` → `TypeError` (not `ReferenceError`) because `sayHi` exists as `undefined`, and calling `undefined` as a function throws TypeError

---

## Q18. What is Error Handling in JavaScript?

Error handling lets you **catch and recover from runtime errors** without crashing the application.

```js
try {
  // code that might throw
  const data = JSON.parse(invalidJSON);
} catch (error) {
  // handle the error
  console.error(error.message);
} finally {
  // always runs — use for cleanup (close connections, hide loaders)
  setLoading(false);
}
```

### `throw` — create custom errors

```js
function validateAge(age) {
  if (age < 18) throw new Error("Must be 18+");
}

try {
  validateAge(15);
} catch (e) {
  console.log(e.message); // "Must be 18+"
}
```

### Common Error Types

| Error | Cause |
|-------|-------|
| `SyntaxError` | Invalid JS syntax (parse-time) |
| `ReferenceError` | Accessing undefined variable |
| `TypeError` | Wrong operation on a type (`null.foo`) |
| `RangeError` | Value out of valid range (`new Array(-1)`) |

**Key points:**
- `finally` runs whether or not an error occurred — good for cleanup
- `try-catch` does **not** catch syntax errors (they happen before execution)
- Always log or handle errors in `catch` — an empty `catch` silently swallows bugs

**Common trap:** Using `try-catch` around normal logic (like `if` checks) is unnecessary overhead. Reserve it for genuinely risky operations (API calls, JSON parsing).

---

## Q19. What is JSON?

**JSON (JavaScript Object Notation)** is a lightweight, text-based data format used to exchange data between systems. It is language-independent and the standard format for REST APIs.

```json
{
  "id": 1,
  "name": "Alice",
  "isActive": true,
  "skills": ["JS", "React"]
}
```

### Key Methods

```js
// Object → JSON string (for sending over network)
const json = JSON.stringify({ name: "Alice", age: 30 });
// '{"name":"Alice","age":30}'

// JSON string → Object (for using API response)
const obj = JSON.parse('{"name":"Alice","age":30}');
// { name: "Alice", age: 30 }
```

### JSON vs JavaScript Object

| | JSON | JS Object |
|---|---|---|
| Key quotes | Required (`"key"`) | Optional |
| Values | String, Number, Boolean, Array, Object, null | Any JS value |
| Functions | ❌ Not supported | ✅ Supported |
| `undefined` | ❌ Not supported | ✅ Supported |
| Usage | Data transfer (network) | Runtime data |

**Key points:**
- `JSON.stringify()` removes functions and `undefined` values
- All JSON keys **must** be double-quoted strings
- `response.json()` in `fetch` is just `JSON.parse()` under the hood

**Common trap:** Passing a JS object with methods to `JSON.stringify()` — functions are silently dropped.

---

## Q20. What is Asynchronous Programming in JavaScript?

Async programming allows JavaScript to **perform long-running operations without blocking the main thread**, keeping the UI responsive.

JavaScript is **single-threaded** — one Call Stack, one thing executing at a time. Async operations are offloaded to the **browser/Node runtime**, and their results come back through the **Event Loop**.

---

### The full runtime model

```
┌─────────────────────────────────────┐
│           Call Stack                │  ← JS executes here (one at a time)
│  [ main() → fetch() → ... ]        │
└────────────────┬────────────────────┘
                 │ offloads async work
┌────────────────▼────────────────────┐
│           Web / Node APIs           │  ← setTimeout, fetch, DOM events
│  (browser handles these outside JS) │
└──────────┬──────────────────────────┘
           │ on completion, pushes callback to queue
     ┌─────▼──────┐      ┌─────────────────────┐
     │ Macrotask  │      │   Microtask Queue   │
     │   Queue    │      │  (higher priority)  │
     │ setTimeout │      │  Promise .then()    │
     │ setInterval│      │  async/await        │
     │ DOM events │      │  queueMicrotask()   │
     └─────┬──────┘      └──────────┬──────────┘
           │                        │
           └──────────┬─────────────┘
                      │
           ┌──────────▼──────────┐
           │     Event Loop      │  ← checks: is Call Stack empty?
           │  1. drain Microtask │     → yes: run ALL microtasks first
           │  2. one Macrotask   │     → then: run ONE macrotask
           └─────────────────────┘       → repeat
```

---

### Microtask vs Macrotask — the critical distinction

| | Microtask Queue | Macrotask Queue |
|---|---|---|
| Sources | `Promise.then/catch/finally`, `async/await`, `queueMicrotask()`, `MutationObserver` | `setTimeout`, `setInterval`, `setImmediate` (Node), DOM events, `MessageChannel` |
| Priority | **Higher** — fully drained before any macrotask | Lower — one per Event Loop tick |
| When processed | After every task, before the next macrotask | One per loop iteration |

**Rule:** After the Call Stack empties, the Event Loop drains the **entire** Microtask Queue first, then picks **one** Macrotask. Then drains Microtasks again, and so on.

---

### Classic output questions — must know for 4.5 YOE

#### Example 1 — basic async order
```js
console.log("1");
setTimeout(() => console.log("2"), 0);
console.log("3");
// Output: 1  3  2
// setTimeout goes to macrotask queue — runs after sync code clears
```

#### Example 2 — microtask vs macrotask priority
```js
console.log("start");

setTimeout(() => console.log("setTimeout"), 0);   // macrotask

Promise.resolve().then(() => console.log("promise")); // microtask

console.log("end");

// Output:
// start
// end
// promise      ← microtask runs before macrotask
// setTimeout
```

#### Example 3 — senior level: nested microtasks
```js
console.log("1");

setTimeout(() => console.log("2"), 0);

Promise.resolve()
  .then(() => {
    console.log("3");
    Promise.resolve().then(() => console.log("4")); // nested microtask
  });

console.log("5");

// Output: 1  5  3  4  2
// Microtask queue is fully drained (3 then 4) before setTimeout (2) runs
```

---

### The 3 async patterns (evolution)

```js
// 1. Callback — oldest, leads to "callback hell"
getUser(id, function(user) {
  getPosts(user.id, function(posts) {
    getComments(posts[0].id, function(comments) {
      // deeply nested — hard to read, error handling is painful
    });
  });
});

// 2. Promise — flat chaining, better error handling
getUser(id)
  .then(user => getPosts(user.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(err => console.error(err));   // single catch for the whole chain

// 3. Async/Await — reads like synchronous, easiest to reason about
async function loadData(id) {
  try {
    const user     = await getUser(id);
    const posts    = await getPosts(user.id);
    const comments = await getComments(posts[0].id);
    return comments;
  } catch (err) {
    console.error(err);   // one catch block for all failures
  }
}
```

---

### Promise internals — states

A Promise is always in one of three states:

```
pending → fulfilled (resolved with a value)
        → rejected  (rejected with a reason)
```

Once settled (fulfilled or rejected), a Promise **cannot change state**. `.then()` handlers are scheduled as **microtasks**, not run synchronously.

```js
const p = new Promise((resolve, reject) => {
  // executor runs synchronously
  resolve(42);
});

p.then(val => console.log(val)); // schedules as microtask — runs after current sync code
console.log("sync");

// Output: sync  42
```

---

### `Promise.all` vs `Promise.allSettled` vs `Promise.race`

```js
// Promise.all — runs in parallel, fails fast if any rejects
const [users, products] = await Promise.all([fetchUsers(), fetchProducts()]);

// Promise.allSettled — waits for all, never short-circuits
const results = await Promise.allSettled([fetchA(), fetchB()]);
results.forEach(r => {
  if (r.status === "fulfilled") console.log(r.value);
  else console.log(r.reason);
});

// Promise.race — resolves/rejects with whichever settles first
const result = await Promise.race([fetchWithTimeout(), timeoutPromise(5000)]);
```

---

### async/await under the hood

`async/await` is syntactic sugar over Promises. An `async` function always returns a Promise. `await` pauses execution of that function and schedules the rest as a microtask — it does NOT block the Call Stack.

```js
async function example() {
  console.log("A");
  const result = await Promise.resolve("B"); // pauses here, returns to caller
  console.log(result); // resumes as microtask
  console.log("C");
}

example();
console.log("D");

// Output: A  D  B  C
// After await, the rest of the function is a microtask — D runs first
```

---

**Key points:**
- JS is single-threaded — async does not mean parallel; it means non-blocking
- **Microtasks** (Promises) always run before **Macrotasks** (setTimeout) — fully drained each loop tick
- `async/await` is Promise-based — errors from `await` must be caught with `try/catch`
- `Promise.all` is the go-to for parallel requests; `Promise.allSettled` when you need all results regardless of failure
- The Event Loop is the mechanism that makes JS non-blocking despite being single-threaded

**Common traps:**
- `setTimeout(fn, 0)` does NOT mean immediate — it waits for the Call Stack to clear AND for all microtasks to drain first
- `await` inside a `forEach` does **not** work as expected — `forEach` doesn't await Promises. Use `for...of` or `Promise.all` instead:
```js
// ❌ forEach doesn't await
items.forEach(async (item) => { await process(item); });

// ✅ for...of sequential
for (const item of items) { await process(item); }

// ✅ Promise.all parallel
await Promise.all(items.map(item => process(item)));
```
- Unhandled Promise rejections crash Node.js apps in production — always add `.catch()` or `try/catch`

---

---

## Q21. Difference between Primitive and Non-Primitive Data Types?

The core distinction is **how values are stored and copied in memory**.

| | Primitive | Non-Primitive |
|---|---|---|
| Stored by | Value | Reference |
| Mutable | ❌ No | ✅ Yes |
| Copy behavior | Independent copy | Shared reference |
| Types | String, Number, Boolean, Null, Undefined, Symbol, BigInt | Object, Array, Function |

```js
// Primitive — copy by value
let a = 10;
let b = a;
b = 20;
console.log(a); // 10 — unchanged

// Non-Primitive — copy by reference
const user1 = { name: "Alice" };
const user2 = user1;       // same reference, not a clone
user2.name = "Bob";
console.log(user1.name);   // "Bob" — both affected!
```

### How to properly clone an object

```js
const copy = { ...user1 };          // shallow clone
const deep = JSON.parse(JSON.stringify(user1)); // deep clone (no functions)
```

**Key points:**
- `{} === {}` → `false` — different references even with identical content
- In React, state must be treated as immutable — always pass a new object/array to the setter
- Shallow clone copies only the top level; nested objects still share references

**Common trap:** `const arr2 = arr1` does NOT copy the array — both variables point to the same array in memory.

---

## Q22. Difference between `null` and `undefined`?

Both represent "no value" but for different reasons.

| | `undefined` | `null` |
|---|---|---|
| Set by | JavaScript automatically | Developer intentionally |
| Meaning | Variable declared but not assigned | Explicitly empty / no value |
| `typeof` | `"undefined"` | `"object"` (legacy bug) |
| `== null` | `true` | `true` |
| `=== null` | `false` | `true` |

```js
let user;          // undefined — JS sets this automatically
let manager = null; // null — developer says "no manager exists"

console.log(undefined == null);  // true  (loose equality)
console.log(undefined === null); // false (strict equality)
```

**Key points:**
- Use `null` to intentionally signal "no value" (e.g., API returns no manager: `"manager": null`)
- Use `undefined` as the natural uninitialized state — don't assign it manually
- A function with no `return` statement returns `undefined`
- `typeof null === "object"` is a **historic JS bug** kept for backward compatibility — null is actually a primitive

**Common trap:** `null == undefined` is `true` with `==` but `false` with `===`. Always use `===` in production code.

---

## Q23. What is the `typeof` Operator?

`typeof` returns a **string** indicating the type of a value. Used for runtime type checking and defensive programming.

```js
typeof 42            // "number"
typeof "hello"       // "string"
typeof true          // "boolean"
typeof undefined     // "undefined"
typeof function(){}  // "function"
typeof {}            // "object"
typeof []            // "object"  ← not "array"!
typeof null          // "object"  ← historic bug!
typeof Symbol()      // "symbol"
typeof 42n           // "bigint"
```

### Correct checks for tricky types

```js
Array.isArray([]);          // ✅ check for array
value === null;             // ✅ check for null
value instanceof Date;      // ✅ check for Date
```

**Key points:**
- `typeof` is safe to call on undeclared variables — it returns `"undefined"` instead of throwing
- Use `Array.isArray()` for arrays, not `typeof`
- Use `=== null` for null checks, not `typeof`
- Functions get their own special return value `"function"` even though they are objects

**Common trap:** Developers often write `typeof arr === "array"` — this never returns true. The correct string is `"object"`, and you need `Array.isArray()`.

---

## Q24. What is Type Coercion in JavaScript?

Type coercion is JavaScript's **automatic conversion of one data type to another** when values of different types are used together. It happens silently, which is why it causes many unexpected bugs.

### Implicit Coercion (automatic)

```js
"5" + 5      // "55"  — number → string (+ triggers concatenation)
"5" - 5      //  0   — string → number (- forces numeric)
true + 1     //  2   — boolean → number (true = 1)
false + 1    //  1   — boolean → number (false = 0)
"" + false   // "false" — boolean → string
```

### Explicit Coercion (manual / developer-controlled)

```js
Number("5")     // 5
String(10)      // "10"
Boolean(0)      // false
Boolean("hi")   // true
parseInt("42px") // 42
```

### Equality coercion (`==` vs `===`)

```js
1 == "1"        // true  — coercion applied
1 === "1"       // false — no coercion, types differ
null == undefined  // true
null === undefined // false
[] == false        // true  — both coerce to 0
```

**Key points:**
- `+` with a string always results in string concatenation
- `-`, `*`, `/` always coerce to numbers
- `===` prevents coercion entirely — always prefer it
- Falsy values: `0`, `""`, `null`, `undefined`, `NaN`, `false`

**Common traps:**
- `[] + []` → `""` (both coerce to empty strings)
- `[] + {}` → `"[object Object]"`
- `{} + []` → `0` (depends on context — can be a famous JS quirk)

---

## Q25. What are Operators and their Types? *(Detailed)*

> See Q9 for a quick overview. This covers the less obvious operators.

### Nullish Coalescing (`??`)
Returns the right-hand value only when the left side is `null` or `undefined` (not other falsy values).

```js
const name = null ?? "Guest";  // "Guest"
const count = 0 ?? 10;         // 0  ← 0 is NOT null/undefined
const count2 = 0 || 10;        // 10 ← 0 is falsy, so || replaces it
```

### Optional Chaining (`?.`)
Safely access nested properties — returns `undefined` instead of throwing.

```js
const city = user?.address?.city;   // undefined if any part is null/undefined
user?.greet?.();                    // safe method call
```

### Spread (`...`) and Rest (`...`)

```js
// Spread — expand an iterable
const merged = [...arr1, ...arr2];
const copy = { ...obj };

// Rest — collect remaining arguments
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
sum(1, 2, 3, 4); // 10
```

### Destructuring

```js
const { name, age } = user;               // object destructuring
const [first, second] = skills;           // array destructuring
const { address: { city } } = user;       // nested destructuring
const { name = "Default" } = user;        // with default value
```

**Key points:**
- `??` is safer than `||` when `0`, `false`, or `""` are valid values
- `?.` prevents `TypeError: Cannot read property of null` — very common in real apps
- Spread creates a **shallow copy** — nested objects are still shared references

---

## Q26. Difference between Unary, Binary, and Ternary Operators?

Classification by the **number of operands** an operator requires.

| Type | Operands | Examples |
|------|----------|---------|
| **Unary** | 1 | `!`, `typeof`, `++`, `--`, `-x`, `delete` |
| **Binary** | 2 | `+`, `-`, `*`, `/`, `===`, `&&`, `\|\|` |
| **Ternary** | 3 | `condition ? valueA : valueB` |

```js
// Unary
let x = 5;
console.log(-x);        // -5
console.log(typeof x);  // "number"
console.log(!true);     // false
x++;                    // 6

// Binary
console.log(10 + 5);    // 15
console.log(5 > 3);     // true

// Ternary
const label = x > 18 ? "Adult" : "Minor";
```

**Key points:**
- `+` can be both unary (`+"5"` → `5`, converts string to number) and binary (`5 + 3`)
- Ternary is the **only** JavaScript operator that takes 3 operands
- Avoid nesting ternaries — they hurt readability; use `if-else` instead

**Common trap:** `typeof` is unary, not a function — no parentheses required (though they're allowed: `typeof(x)`).

---

## Q27. What is Short-Circuit Evaluation?

Short-circuit evaluation means logical operators **stop evaluating as soon as the result is determined** — they don't execute unnecessary expressions.

### `&&` (AND) — stops at first falsy

```js
false && console.log("runs?");  // console.log never runs
true  && console.log("runs?");  // prints "runs?"

// Practical use — conditional rendering in React
isLoggedIn && <Dashboard />
```

### `||` (OR) — stops at first truthy

```js
true  || console.log("runs?");  // console.log never runs
false || console.log("runs?");  // prints "runs?"

// Practical use — default value
const name = userName || "Guest";
```

### `??` (Nullish Coalescing) — stops only for `null`/`undefined`

```js
null      ?? "default"  // "default"
undefined ?? "default"  // "default"
0         ?? "default"  // 0  ← not replaced (0 is not null/undefined)
""        ?? "default"  // "" ← not replaced
```

### Key difference: `||` vs `??`

```js
const score = 0;
score || 100   // 100 — wrong! 0 is falsy
score ?? 100   // 0   — correct! 0 is a valid value
```

**Key points:**
- Short-circuiting improves **performance** (skips unnecessary calls) and **safety** (prevents calling methods on null)
- In React, `&&` is the standard pattern for conditional rendering
- Prefer `??` over `||` when `0`, `false`, or `""` are valid meaningful values

**Common trap:** Rendering `0` in React with `&&` — `{count && <Badge />}` renders `0` when count is 0. Use `{count > 0 && <Badge />}` instead.

---

## Q28. What is Operator Precedence?

Operator precedence determines the **order in which operators are evaluated** in a complex expression — higher precedence executes first.

### Precedence order (high → low)

| Priority | Operators |
|----------|-----------|
| Highest | `()` Grouping |
| Unary | `!`, `typeof`, `++`, `--` |
| Multiplicative | `*`, `/`, `%` |
| Additive | `+`, `-` |
| Comparison | `>`, `<`, `>=`, `<=` |
| Equality | `==`, `===`, `!=`, `!==` |
| Logical AND | `&&` |
| Logical OR | `\|\|` |
| Nullish | `??` |
| Ternary | `? :` |
| Lowest | `=` Assignment |

```js
2 + 3 * 4         // 14  — * runs first (higher precedence than +)
(2 + 3) * 4       // 20  — () overrides precedence

5 > 3 && 10 < 20  // true — comparisons run before &&
true || false && false  // true — && runs before ||
```

### Interview output questions

```js
console.log(10 + 5 * 2);         // 20
console.log((10 + 5) * 2);       // 30
console.log(true || false && false); // true (false && false = false, then true || false = true)
```

**Key points:**
- `&&` has higher precedence than `||` — this trips up many developers
- When in doubt, use **parentheses** — they clarify intent and prevent bugs
- Assignment (`=`) has the lowest precedence, so `if (a = 5)` accidentally assigns instead of comparing

**Common trap:** `a && b || c` is evaluated as `(a && b) || c`, not `a && (b || c)`. Add parentheses to be explicit.

---

## Q29. When to use `if-else`, `switch`, and Ternary?

Choosing the right conditional construct improves readability and maintainability.

| Construct | Best For |
|-----------|----------|
| `if-else` | Complex logic, multiple conditions, range checks |
| `switch` | One variable compared against many fixed values |
| Ternary `? :` | Simple value assignment, JSX conditional rendering |
| `&&` | Render something or nothing in JSX |

```js
// if-else — complex multi-condition logic
if (user.age < 18) {
  denyAccess();
} else if (user.role === "ADMIN") {
  grantFullAccess();
} else {
  grantLimitedAccess();
}

// switch — one variable, many exact matches
switch (user.role) {
  case "ADMIN":   return <AdminPanel />;
  case "MANAGER": return <ManagerPanel />;
  case "USER":    return <UserPanel />;
  default:        return <NotFound />;
}

// Ternary — simple binary choice (especially in JSX)
const label = isActive ? "Active" : "Inactive";
return isLoading ? <Spinner /> : <Content />;

// && — render or nothing
return isLoggedIn && <Dashboard />;
```

**Key points:**
- Never use nested ternaries — they're hard to read and maintain
- `switch` requires `break` in each case or execution **falls through** to the next case
- Use `if-else` whenever you have range checks (`>`, `<`, `>=`) — switch only works with exact values

**Common trap:** Using a ternary for 3+ conditions forces nesting. That's a signal to use `if-else` or `switch` instead.

---

## Q30. Difference between `==` and `===`?

`==` (loose equality) performs **type coercion** before comparing.  
`===` (strict equality) compares **value AND type** — no conversion.

```js
// == allows coercion
1 == "1"           // true  (string "1" → number 1)
true == 1          // true  (true → 1)
false == 0         // true  (false → 0)
null == undefined  // true  (special rule)
"" == false        // true  (both → 0)

// === no coercion
1 === "1"          // false (number vs string)
true === 1         // false (boolean vs number)
null === undefined // false (different types)
```

### Quick reference table

| Expression | `==` | `===` |
|------------|------|-------|
| `1` vs `"1"` | `true` | `false` |
| `0` vs `false` | `true` | `false` |
| `null` vs `undefined` | `true` | `false` |
| `""` vs `false` | `true` | `false` |
| `NaN` vs `NaN` | `false` | `false` |

> Note: `NaN` is not equal to itself with either operator. Use `Number.isNaN(value)` to check for NaN.

**Key points:**
- Always use `===` in production code — it's predictable and avoids coercion surprises
- The only safe use of `==` is `value == null` which checks for both `null` and `undefined` in one expression
- ESLint rule `eqeqeq` enforces `===` usage across a codebase

**Common trap:** `0 == false` → `true` but `0 === false` → `false`. This catches many developers off guard in conditional checks.

---

## Q31. What is a Closure in JavaScript?

A closure is a function that **remembers the variables from its outer (lexical) scope even after that outer function has finished executing**.

Every function in JavaScript forms a closure — it carries a reference to the scope it was defined in, not the scope it is called from.

```js
function outer() {
  let count = 0;              // outer variable

  return function inner() {   // inner function — a closure
    count++;
    console.log(count);
  };
}

const increment = outer();   // outer() has returned, but count is NOT garbage collected
increment(); // 1
increment(); // 2
increment(); // 3
```

`inner` closes over `count` — it holds a live reference to it. `count` stays alive in memory as long as `increment` exists.

### Why closures exist — the mechanism

When `inner` is created, JS attaches its **[[Environment]]** (lexical environment) to it. That environment holds the variable bindings of the surrounding scope. Even after `outer` returns, the environment is kept alive because `inner` still references it.

```
outer() call
  └── Lexical Environment: { count: 0 }
        └── inner() captures this environment → closure formed
```

---

### Real-world use cases

#### 1. Data privacy / encapsulation
```js
function createCounter() {
  let count = 0;  // private — not accessible from outside

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount()  { return count; }
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);      // undefined — count is private
```

#### 2. Factory functions / partial application
```js
function multiplier(factor) {
  return (number) => number * factor;  // closes over factor
}

const double = multiplier(2);
const triple = multiplier(3);

double(5); // 10
triple(5); // 15
```

#### 3. Event handlers retaining state (React-like pattern)
```js
function makeAdder(x) {
  return (y) => x + y;   // x is closed over
}
const add5 = makeAdder(5);
add5(3); // 8
add5(10); // 15
```

#### 4. setTimeout in loops — classic closure trap
```js
// ❌ Wrong — var is function-scoped, all callbacks share same i
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3  3  3

// ✅ Fix 1 — let creates a new binding per iteration
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0  1  2

// ✅ Fix 2 — IIFE to create a new scope per iteration (pre-ES6 approach)
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// Output: 0  1  2
```

---

### Closures in React

React hooks are built on closures. Every render creates new function scopes — hooks close over the state and props of that render.

```js
function Counter() {
  const [count, setCount] = useState(0);

  // handleClick closes over the current value of count
  const handleClick = () => {
    console.log(count); // always the count from THIS render
    setCount(count + 1);
  };

  return <button onClick={handleClick}>{count}</button>;
}
```

**Stale closure problem in React** — a common senior-level bug:

```js
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // ❌ stale closure: count is always 0
    }, 1000);
    return () => clearInterval(id);
  }, []); // empty deps → closes over initial count=0
}

// ✅ Fix — use functional update form, doesn't rely on closed-over value
setCount(prev => prev + 1);
```

---

### Closure vs Scope — quick distinction

| | Scope | Closure |
|---|---|---|
| What it is | Rule for variable lookup | Function + its captured environment |
| When created | At code structure level | When a function is defined inside another |
| Persists after return? | No | Yes — as long as the inner function exists |

---

**Key points:**
- Closures are not a special syntax — every function automatically forms one
- They enable **data privacy**, **stateful functions**, **currying**, and **partial application**
- Closures keep variables alive in memory — be aware of **memory leaks** if closures are held unnecessarily (e.g. large objects captured inside event listeners never removed)
- React's useState, useEffect, useCallback all rely on closure behavior

**Common traps:**
- `var` in loops — all iterations share the same binding → use `let` or IIFE
- Stale closures in `useEffect` — the closure captures an old value of state/props → use functional update or add correct dependencies
- Unintentionally retaining large objects in memory via long-lived closures (e.g., closures inside detached DOM event listeners)

---
