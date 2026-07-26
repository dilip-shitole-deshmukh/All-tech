# JavaScript Interview Q&A — Phase 2

> Q31–Q50 | Arrays (Advanced), Operators, Conditionals, Type System

---

## Q31. Difference between Spread (`...`) and Rest (`...`) Operator?

Same syntax, opposite purposes.

| | Spread | Rest |
|---|---|---|
| Purpose | **Expand** an iterable into individual elements | **Collect** multiple elements into an array |
| Used in | Function calls, array/object literals | Function parameters, destructuring |

```js
// Spread — expands
const arr = [1, 2, 3];
Math.max(...arr);          // 3
const copy = [...arr];     // [1, 2, 3]
const merged = [...arr, 4, 5]; // [1, 2, 3, 4, 5]

// Spread with objects
const user = { name: "Alice" };
const updated = { ...user, role: "Admin" }; // { name: "Alice", role: "Admin" }

// Rest — collects
function sum(first, ...rest) {
  return rest.reduce((acc, n) => acc + n, first);
}
sum(1, 2, 3, 4); // 10

// Rest in destructuring
const [head, ...tail] = [10, 20, 30];
// head = 10, tail = [20, 30]
```

**Key points:**
- Spread creates a **shallow copy** — nested objects still share references
- Rest parameter must always be **last** in a function signature
- In React, spread is used constantly for immutable state updates: `setUser({ ...user, name: "New" })`

**Common trap:** `const copy = { ...obj }` is a shallow clone, not deep. Nested objects are still shared.

---

## Q32. What are Arrays in JavaScript? (Advanced)

Arrays are **ordered, zero-indexed, dynamically-sized, mutable** objects. They can hold mixed types, are stored by reference, and have a rich built-in method API.

```js
typeof [];          // "object" — arrays ARE objects
Array.isArray([]);  // true   — correct check
[] === [];          // false  — different references
```

### Mutating vs Non-Mutating methods

| Mutates original | Does NOT mutate |
|-----------------|-----------------|
| `push`, `pop`, `shift`, `unshift` | `map`, `filter`, `slice`, `concat` |
| `splice`, `sort`, `reverse` | `find`, `reduce`, `flat`, `every`, `some` |

**Key points:**
- `const arr = []` — `const` prevents reassignment of the variable, not mutation of the array
- In React, **never mutate state arrays** — always return a new array: `[...arr, newItem]`
- `sort()` converts elements to strings by default — always pass a comparator for numbers: `arr.sort((a, b) => a - b)`

---

## Q33. What is `indexOf()`? How does it differ from `includes()`?

```js
const arr = [10, 20, 30];

arr.indexOf(20);   // 1  — returns index
arr.indexOf(99);   // -1 — not found

arr.includes(20);  // true  — returns boolean
arr.includes(99);  // false
```

| | `indexOf()` | `includes()` |
|---|---|---|
| Returns | Index (number) | Boolean |
| Use when | You need the position | You only need existence check |
| NaN handling | `indexOf(NaN)` → `-1` (broken) | `includes(NaN)` → `true` (correct) |

**Common trap:**
```js
// Using indexOf as a boolean check breaks at index 0
if (arr.indexOf(item)) { }  // ❌ falsy when index is 0!
if (arr.indexOf(item) !== -1) { }  // ✅
if (arr.includes(item)) { }        // ✅ cleaner
```

---

## Q34. Difference between `find()` and `filter()`?

```js
const users = [
  { id: 1, active: true },
  { id: 2, active: false },
  { id: 3, active: true },
];

users.find(u => u.id === 2);         // { id: 2, active: false }  — single object
users.filter(u => u.active);         // [{ id: 1 ... }, { id: 3 ... }] — array
```

| | `find()` | `filter()` |
|---|---|---|
| Returns | First matching **element** (or `undefined`) | New **array** of all matches |
| Stops early | ✅ Yes — stops at first match | ❌ No — iterates all |
| Use when | Looking up by unique ID | Filtering a list |

**Common trap:** Expecting an array from `find()` — it returns the element itself, not a wrapped array.

---

## Q35. What is `slice()`? How does it differ from `splice()`?

**Quick memory trick:** `slice` = safe copy, `splice` = surgery on the array.

```js
const arr = [10, 20, 30, 40, 50];

// slice(start, end) — NON-mutating, end is exclusive
arr.slice(1, 4);    // [20, 30, 40] — original unchanged
arr.slice(-2);      // [40, 50]     — negative index from end

// splice(start, deleteCount, ...items) — MUTATING
arr.splice(1, 2);         // removes [20, 30], arr is now [10, 40, 50]
arr.splice(1, 0, 99);     // inserts 99 at index 1
arr.splice(1, 1, 88);     // replaces index 1 with 88
```

| | `slice()` | `splice()` |
|---|---|---|
| Mutates | ❌ No | ✅ Yes |
| Returns | New array (copy) | Array of removed elements |
| Add/remove | ❌ | ✅ Both |
| React-safe | ✅ | ❌ Avoid in state |

---

## Q36. Difference between `push()` and `concat()`?

```js
const arr = [1, 2];

// push — mutates, returns new length
arr.push(3);           // arr = [1, 2, 3], returns 3

// concat — non-mutating, returns new array
const newArr = arr.concat(4);  // [1, 2, 3, 4], arr unchanged
const merged = arr.concat([5, 6]); // [1, 2, 3, 5, 6]
```

| | `push()` | `concat()` |
|---|---|---|
| Mutates | ✅ Yes | ❌ No |
| Returns | New length (number) | New array |
| React-safe | ❌ | ✅ |

Modern alternative to `concat` in React: spread operator `[...arr, newItem]`.

---

## Q37. Difference between `pop()` and `shift()`?

Both remove and return one element, both mutate the array.

```js
const arr = [10, 20, 30];

arr.pop();    // returns 30, arr = [10, 20]  — removes from END
arr.shift();  // returns 10, arr = [20]      — removes from START
```

| | `pop()` | `shift()` |
|---|---|---|
| Removes from | End | Start |
| Performance | O(1) fast | O(n) slower — must re-index all elements |
| Stack (LIFO) | ✅ | |
| Queue (FIFO) | | ✅ |

---

## Q38. What is `splice()`? What can it do?

`splice()` is the most versatile array mutator — it can **remove**, **insert**, and **replace** at any position.

```js
const arr = ["A", "B", "C", "D"];

// Remove 2 elements starting at index 1
arr.splice(1, 2);        // returns ["B","C"], arr = ["A","D"]

// Insert without removing (deleteCount = 0)
arr.splice(1, 0, "X", "Y"); // arr = ["A","X","Y","D"]

// Replace 1 element
arr.splice(1, 1, "Z");   // arr = ["A","Z","Y","D"]
```

**Common trap:** `splice(1)` with no deleteCount removes **everything from index 1 onward** — not just one element.

**In React:** Never use splice on state arrays. Use `filter` to remove, spread to add, `map` to replace.

---

## Q39. What is `map()`?

`map()` transforms every element of an array and returns a **new array of the same length**. It never mutates the original.

```js
const nums = [1, 2, 3, 4];
nums.map(n => n * 2);     // [2, 4, 6, 8]

const users = [{ name: "Alice" }, { name: "Bob" }];
users.map(u => u.name);   // ["Alice", "Bob"]

// React list rendering
users.map(u => <UserCard key={u.id} user={u} />);
```

**Key points:**
- Always returns a **new array of the same length** as the input
- If you don't need the returned array, use `forEach` instead
- Forgetting `return` in a block-body arrow function returns `undefined` for every element

**Common trap:**
```js
arr.map(item => { item * 2 }); // returns [undefined, undefined, ...]
arr.map(item => item * 2);     // ✅ implicit return
arr.map(item => { return item * 2; }); // ✅ explicit return
```

---

## Q40. What is `filter()`?

`filter()` returns a new array containing only the elements for which the callback returns `true`. The original array is unchanged.

```js
const nums = [1, 2, 3, 4, 5, 6];
nums.filter(n => n % 2 === 0);   // [2, 4, 6]

const users = [
  { name: "Alice", active: true },
  { name: "Bob",   active: false },
];
users.filter(u => u.active);     // [{ name: "Alice", active: true }]

// Search pattern in React
const results = users.filter(u =>
  u.name.toLowerCase().includes(query.toLowerCase())
);
```

**Key points:**
- Always returns an **array** (possibly empty)
- Pair with `find()` when you only need one match — `find()` is faster since it stops early

---

## Q41. What is `reduce()`?

`reduce()` processes every element and **accumulates them into a single value** — that value can be a number, string, object, or array.

```js
// Sum
[1, 2, 3, 4].reduce((acc, n) => acc + n, 0);  // 10

// Cart total
const total = cart.reduce((sum, item) => sum + item.price, 0);

// Count occurrences
const count = ["a","b","a","c","a"].reduce((acc, val) => {
  acc[val] = (acc[val] || 0) + 1;
  return acc;
}, {});
// { a: 3, b: 1, c: 1 }

// Flatten (basic)
[[1,2],[3,4]].reduce((acc, arr) => [...acc, ...arr], []); // [1,2,3,4]
```

**Key points:**
- Always provide an **initial value** — without it, `reduce` uses the first element and skips the first iteration, causing subtle bugs on empty arrays
- `reduce` can replace `map` + `filter` in one pass for performance-critical code
- Must `return` the accumulator inside the callback

**Common trap:** Forgetting to return `acc` at the end of the callback — the accumulator becomes `undefined` on the next iteration.

---

## Q42. Difference between `map()`, `filter()`, and `reduce()`?

| Method | Purpose | Returns | Same length? |
|--------|---------|---------|-------------|
| `map()` | Transform each element | New array | ✅ Always |
| `filter()` | Select matching elements | New array | ❌ Smaller or equal |
| `reduce()` | Aggregate into one value | Any type | ❌ Single value |

```js
const products = [
  { name: "Laptop", price: 50000, active: true },
  { name: "Mouse",  price: 500,   active: false },
  { name: "Keyboard", price: 2000, active: true },
];

// Chain them — a very common real-world pattern
const activeTotal = products
  .filter(p => p.active)              // keep active
  .map(p => p.price)                  // extract price
  .reduce((sum, p) => sum + p, 0);    // total = 52000
```

**Senior-level pattern:** Combine all three in one `reduce` for a single-pass implementation when performance matters.

---

## Q43. Difference between `null` and `undefined`?

> Covered in detail in JSQA.md Q22. Key distinction for this phase:

```js
// undefined — JS assigned it (no value yet)
let user;                    // undefined
function greet() {}          // returns undefined
const obj = {}; obj.missing; // undefined

// null — developer assigned it (intentionally empty)
let selectedUser = null;     // "nothing selected"
const response = { manager: null }; // manager doesn't exist
```

**The one safe use of `==`:**
```js
value == null  // true for both null AND undefined — useful for "is it missing?"
value === null // only true for null
```

---

## Q44. What is the `typeof` Operator?

> Covered in detail in JSQA.md Q23. Key additions for this phase:

```js
typeof NaN         // "number"  ← NaN is technically a number type
typeof class C {}  // "function" ← classes report as function
typeof undeclaredVar // "undefined" ← safe, no ReferenceError
```

**Correct type checks for tricky values:**
```js
Number.isNaN(NaN);          // ✅ check for NaN
Array.isArray([]);           // ✅ check for array
value === null;              // ✅ check for null
value instanceof Date;       // ✅ check for Date
Object.prototype.toString.call(value); // ✅ universal type check
```

`Object.prototype.toString.call([])` → `"[object Array]"` — the most reliable way to check any type.

---

## Q45. What is Type Coercion in JavaScript?

> Covered in detail in JSQA.md Q24. Key additions for this phase:

### Falsy values (exactly 8)
```js
false, 0, -0, 0n, "", null, undefined, NaN
```
Everything else is truthy — including `[]`, `{}`, `"0"`, `"false"`.

### Implicit coercion rules
```js
// + with string → string concatenation
"5" + 5          // "55"
"5" + true       // "5true"

// -, *, / → always numeric coercion
"5" - 5          // 0
"5" * "2"        // 10
null + 1         // 1    (null → 0)
undefined + 1    // NaN  (undefined → NaN)

// Comparison with ==
[] == false      // true  (both → 0)
"" == false      // true  (both → 0)
null == 0        // false (null only == undefined)
```

**Common trap:** `[] == false` is `true` due to coercion, but `!![]` is also `true` because `[]` is truthy. This is why `===` matters.

---

## Q46. What are the Types of Operators in JavaScript?

> Covered in detail in JSQA.md Q9 and Q25. Key additions:

### Logical Assignment Operators (ES2021)

```js
// ||= — assign only if left is falsy
a ||= "default";   // equivalent to: a = a || "default"

// &&= — assign only if left is truthy
a &&= newValue;    // equivalent to: a = a && newValue

// ??= — assign only if left is null/undefined
a ??= "fallback";  // equivalent to: a = a ?? "fallback"
```

### `in` and `instanceof`
```js
"name" in { name: "Alice" };   // true — property exists
[] instanceof Array;            // true
[] instanceof Object;           // true — all arrays are objects
```

### Comma operator (rare but asked)
```js
const x = (1, 2, 3);  // x = 3 — evaluates all, returns last
```

---

## Q47. Difference between Unary, Binary, and Ternary Operators?

> Covered in JSQA.md Q26. Key additions:

### Pre vs Post increment/decrement (unary)
```js
let a = 5;
console.log(a++);  // 5  — returns THEN increments
console.log(a);    // 6

let b = 5;
console.log(++b);  // 6  — increments THEN returns
console.log(b);    // 6
```

This distinction matters in expressions and is a common output question.

---

## Q48. What is Short-Circuit Evaluation?

> Covered in detail in JSQA.md Q27. Key additions:

### Guard pattern
```js
// Call a function only if the object exists
user && user.save();
user?.save();        // modern equivalent with optional chaining
```

### Default value patterns
```js
const port = process.env.PORT || 3000;  // fallback
const name = user?.name ?? "Anonymous"; // nullish fallback (0 and "" preserved)
```

### React rendering pitfall
```js
// ❌ Renders "0" when count is 0 — 0 is falsy but renders in JSX
{count && <Badge count={count} />}

// ✅ Fix — explicit boolean
{count > 0 && <Badge count={count} />}
{!!count && <Badge count={count} />}
```

---

## Q49. What is Operator Precedence?

> Covered in JSQA.md Q28. Key additions for output questions:

### Must-know precedence traps

```js
// + vs * 
2 + 3 * 4        // 14 (not 20)

// && before ||
true || false && false  // true: evaluated as true || (false && false)
false || true && false  // false: evaluated as false || (true && false)

// Comparison before equality
1 < 2 === true   // true: (1 < 2) === true → true === true

// Unary minus
-2 ** 2  // SyntaxError in some engines — use (-2) ** 2 = 4

// Assignment is right-associative
let a, b;
a = b = 5;  // a = 5, b = 5
```

**Best practice:** Use parentheses to make complex expressions explicit — readability always wins over operator precedence knowledge.

---

## Q50. When to use `if-else`, `switch`, and Ternary?

> Covered in JSQA.md Q29. Key additions:

### Modern alternative — object lookup table
When you have many `switch` cases mapping a value to a result, an object map is often cleaner:

```js
// Instead of switch with 5+ cases
const roleLabels = {
  ADMIN:   "Administrator",
  MANAGER: "Manager",
  USER:    "Standard User",
};
const label = roleLabels[role] ?? "Unknown";

// Or with functions
const actions = {
  add:    (a, b) => a + b,
  sub:    (a, b) => a - b,
  mul:    (a, b) => a * b,
};
actions[operation]?.(a, b);
```

**When to use what:**
- `if-else` → complex conditions, range checks, multiple statements per branch
- `switch` → one variable against many exact values, fall-through is intentional
- Ternary → single value assignment or JSX rendering (one condition only)
- Object map → many fixed key→value mappings
