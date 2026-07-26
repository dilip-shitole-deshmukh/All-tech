# JavaScript Interview Q&A — Phase 4

> Q101–Q148 | Strings, DOM, Objects (Deep Dive), Error Handling

---

## Q101. What is a String?

> Core covered in JSQA.md Q4 and phase3.md Q89. Key internals:

Strings in JavaScript are **primitive values** but they auto-box to a `String` object when you access methods. The engine temporarily wraps the primitive, calls the method, then discards the wrapper.

```js
// Primitives — stored by value, immutable
let a = "hello";
let b = a;
b = "world";
console.log(a); // "hello" — unchanged

// String object wrapper (avoid this)
const s = new String("hello");
typeof s;        // "object" — not "string"
s === "hello";   // false — different types

// Auto-boxing happens transparently
"hello".toUpperCase(); // JS briefly creates String object, calls method, discards it
```

**Immutability in practice:**
```js
let str = "hello";
str[0] = "H";     // silently fails (non-strict) or throws (strict mode)
console.log(str); // still "hello"

str = str[0].toUpperCase() + str.slice(1); // correct way: create new string
```

---

## Q102. What are Template Literals and String Interpolation?

> Core covered in JSQA.md Q4 and phase3.md Q90. Additional patterns:

```js
// Multi-line strings — no need for \n
const html = `
  <div class="card">
    <h1>${user.name}</h1>
    <p>${user.email}</p>
  </div>
`;

// Expressions inside ${}
const total = `Total: ${(price * qty * 1.18).toFixed(2)}`;
const label = `Status: ${isActive ? "Active" : "Inactive"}`;

// Nested template literals
const list = items.map(item => `<li>${item}</li>`).join("");
const html2 = `<ul>${list}</ul>`;
```

### Tagged templates — how libraries use them
```js
// styled-components uses this exact pattern
const Button = styled.button`
  background: ${props => props.primary ? "blue" : "white"};
  padding: 10px 20px;
`;

// GraphQL queries
const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) { name email }
  }
`;
```

---

## Q103. Single Quotes vs Double Quotes vs Backticks?

All three create strings. The choice is mostly stylistic, but there are practical differences:

```js
'hello'   // single — common in JS
"hello"   // double — common in HTML attributes, JSON
`hello`   // backtick — template literal (ES6)

// Avoid escaping by mixing
const msg1 = "It's a test";       // no need to escape '
const msg2 = 'He said "hello"';   // no need to escape "
const msg3 = `It's "fine"`;       // backtick, no escaping needed

// Only backticks support:
// 1. Interpolation: `Hello ${name}`
// 2. Multi-line: `line1\nline2` vs actual newlines
// 3. Tagged templates
```

**Team standard:** Pick one style (`"` or `'`) for static strings and always use backticks for interpolated strings. ESLint `quotes` rule enforces this.

---

## Q104. Important String Operations?

> See JSQA.md Q4 and phase3.md Q92–Q95. Additional for 4.5 YOE level:

### Chaining methods — real interview pattern
```js
const input = "  hello WORLD  ";
input.trim().toLowerCase().replace("world", "JS"); // "hello JS"

// Search highlight
function highlight(text, query) {
  return text.replace(
    new RegExp(query, "gi"),
    match => `<mark>${match}</mark>`
  );
}

// Truncate with ellipsis
function truncate(str, maxLen) {
  return str.length > maxLen ? str.slice(0, maxLen) + "..." : str;
}

// Capitalize each word
const titleCase = str =>
  str.toLowerCase().replace(/\b\w/g, c => c.toUpperCase());
titleCase("hello world"); // "Hello World"
```

---

## Q105. What is String Immutability?

Strings cannot be changed in place. Every string operation creates a **new string**. The original is untouched.

```js
let str = "hello";
str.toUpperCase(); // creates "HELLO" — str is still "hello"
str = str.toUpperCase(); // reassign to keep the result

// Why this matters for performance
// ❌ Building a string in a loop — creates O(n²) strings
let result = "";
for (let i = 0; i < 1000; i++) { result += i; } // slow for large n

// ✅ Collect in array, join once
const parts = [];
for (let i = 0; i < 1000; i++) { parts.push(i); }
const result2 = parts.join(""); // one string creation
```

**Immutability benefits:** Strings can be safely shared (no defensive copying needed), cached as keys in maps, and compared by value.

---

## Q106. Ways to Concatenate Strings?

```js
// 1. + operator (simple but can cause coercion bugs)
"Hello" + " " + name;

// 2. Template literals (preferred — readable, supports expressions)
`Hello ${name}`;

// 3. Array.join() (good for building lists)
["Hello", name, "!"].join(" ");

// 4. String.concat() (avoid — verbose, no advantage)
"Hello".concat(" ", name);

// Performance at scale
// For many concatenations: Array.join() > template literal > +
const lines = data.map(d => `${d.id}: ${d.name}`).join("\n");
```

---

## Q107. What is the DOM? Difference between HTML and DOM?

> Core covered in JSQA.md Q5. Key additions for 4.5 YOE:

### How the browser builds the DOM
```
1. Byte stream → Characters → Tokens → Nodes → DOM Tree
2. CSS bytes → CSSOM Tree
3. DOM + CSSOM → Render Tree
4. Layout (Reflow) → Paint → Composite
```

```js
// HTML is static text — the DOM is a live object tree
// Changing the DOM does NOT change the HTML file on disk

// Live vs Static NodeLists
const live   = document.getElementsByClassName("card"); // live — auto-updates
const static = document.querySelectorAll(".card");       // static snapshot

document.body.appendChild(newEl);
live.length;   // updated automatically
static.length; // stays the same as when querySelectorAll ran
```

### Why DOM manipulation is expensive
```
element.style.width = "100px";
   → triggers Reflow (layout recalculation for the element and possibly siblings)
   → triggers Repaint (visual update)
   → browser may need to Composite layers

// Batch DOM writes to minimize reflow
// ❌ Multiple reflows
el1.style.width = "100px";
el2.style.height = "200px";

// ✅ DocumentFragment — batch off-DOM, then insert once
const frag = document.createDocumentFragment();
items.forEach(item => {
  const li = document.createElement("li");
  li.textContent = item;
  frag.appendChild(li);
});
list.appendChild(frag); // single reflow
```

---

## Q108. Select, Modify, Create, and Remove DOM Elements?

```js
// --- SELECT ---
document.getElementById("title");
document.querySelector(".card");          // first match
document.querySelectorAll(".card");       // NodeList of all

// --- MODIFY ---
el.textContent = "Safe text";            // sets text, no XSS risk
el.innerHTML   = "<strong>Bold</strong>"; // parses HTML — XSS risk with user input
el.setAttribute("disabled", true);
el.removeAttribute("disabled");
el.classList.add("active");
el.classList.remove("active");
el.classList.toggle("dark");
el.classList.contains("active");         // boolean check
el.style.color = "red";                  // inline style

// --- CREATE ---
const div = document.createElement("div");
const text = document.createTextNode("Hello");
div.appendChild(text);
document.body.appendChild(div);

// Modern append / prepend (accepts strings and nodes)
parent.append(div, "text");   // end
parent.prepend(div);          // start
el.before(sibling);           // before el
el.after(sibling);            // after el
el.replaceWith(newEl);        // replace

// --- REMOVE ---
el.remove();                  // modern
parent.removeChild(el);       // old way
```

---

## Q109. What are Selectors in JavaScript?

> Core covered in JSQA.md Q6. Key additions:

```js
// Scoped queries — search within an element, not the whole document
const card = document.querySelector(".card");
card.querySelector(".title");  // only within .card, not global

// closest() — walk UP the DOM tree to find ancestor
button.addEventListener("click", e => {
  const row = e.target.closest("tr");  // find containing <tr>
  const id = row?.dataset.id;
});

// matches() — check if element satisfies a selector
el.matches(".card.active");  // true/false
el.matches("[data-role='admin']");

// Event delegation with matches()
document.addEventListener("click", e => {
  if (e.target.matches(".delete-btn")) {
    deleteItem(e.target.dataset.id);
  }
});
```

---

## Q110. Difference between `getElementById`, `getElementsByClassName`, `getElementsByTagName`?

> Core covered in JSQA.md Q7. Key additions:

```js
// HTMLCollection is LIVE — reflects DOM changes
const cards = document.getElementsByClassName("card");
console.log(cards.length); // 3
document.body.appendChild(newCard);
console.log(cards.length); // 4 — auto-updated!

// NodeList from querySelectorAll is STATIC
const nodes = document.querySelectorAll(".card");
document.body.appendChild(newCard);
console.log(nodes.length); // still 3

// Convert HTMLCollection to Array for array methods
Array.from(cards).forEach(c => c.classList.add("loaded"));
[...cards].map(c => c.textContent);
```

---

## Q111. Difference between `querySelector()` and `querySelectorAll()`?

> Core covered in JSQA.md Q6. Key additions:

```js
// NodeList is array-like but NOT an Array
const nodes = document.querySelectorAll(".card");
nodes.forEach(n => n.classList.add("active")); // forEach works (modern browsers)
nodes.map(n => n.textContent);  // ❌ TypeError — no .map on NodeList

// Convert first if you need full array methods
[...nodes].map(n => n.textContent);
Array.from(nodes, n => n.textContent); // with map function
```

---

## Q112. Methods to Modify Element Properties and Attributes?

```js
// Properties vs Attributes — important distinction
// Attribute = HTML source value (string, always initial value)
// Property  = current DOM runtime value (any type, changes with user interaction)

const input = document.querySelector("input");
input.setAttribute("value", "hello");  // sets HTML attribute
input.value = "world";                 // sets DOM property (what user sees)

input.getAttribute("value"); // "hello" — still original attribute
input.value;                  // "world" — current property value

// Practical: use properties for form values, attributes for initial setup
input.checked;      // current check state (boolean property)
input.disabled;     // current disabled state (boolean property)
input.src;          // resolved absolute URL (property)
input.getAttribute("src"); // original relative attribute value

// data-* attributes — the bridge between HTML and JS
el.dataset.userId;          // reads data-user-id attribute
el.dataset.userId = "123";  // sets data-user-id attribute
el.setAttribute("data-user-id", "123"); // same as above
```

---

## Q113. Difference between `innerHTML` and `textContent`?

```js
const div = document.createElement("div");

// textContent — sets/gets plain text, escapes HTML entities
div.textContent = "<script>alert('xss')</script>";
div.innerHTML;    // "&lt;script&gt;..." — safely escaped

// innerHTML — sets/gets HTML, parses tags
div.innerHTML = "<strong>Bold</strong>";  // renders bold text
div.textContent; // "Bold" — strips tags

// innerText — like textContent but CSS-aware (respects display:none, visibility)
// textContent is faster (doesn't trigger layout); prefer it for plain text
```

**Security rule:** Never use `innerHTML` with unsanitized user input — it directly creates XSS vulnerabilities.

```js
// ❌ XSS vulnerability
el.innerHTML = userComment;

// ✅ Safe alternatives
el.textContent = userComment;                     // plain text only
el.innerHTML = DOMPurify.sanitize(userComment);   // sanitized HTML
```

---

## Q114. Add and Remove Properties of HTML Elements in DOM?

```js
// Boolean properties — assign true/false directly
button.disabled = true;
input.checked   = true;
input.readOnly  = true;

// Custom attributes via setAttribute
el.setAttribute("aria-label", "Close dialog");
el.getAttribute("aria-label");   // "Close dialog"
el.hasAttribute("disabled");     // true/false
el.removeAttribute("disabled");

// data-* attributes — preferred for JS-HTML communication
// HTML: <div data-user-id="42" data-role="admin">
el.dataset.userId;  // "42" (always a string)
el.dataset.role;    // "admin"
el.dataset.userId = "99"; // sets data-user-id="99"

// classList API (preferred over className string manipulation)
el.classList.add("active", "visible");  // add multiple
el.classList.remove("active");
el.classList.toggle("open");            // adds if missing, removes if present
el.classList.replace("old", "new");
el.classList.contains("active");        // boolean
[...el.classList]                       // convert to array
```

---

## Q115. Add and Remove Styles from HTML Elements?

```js
// Inline styles (use sparingly — hard to override with CSS)
el.style.color = "red";
el.style.backgroundColor = "blue";  // camelCase for CSS properties
el.style.setProperty("--theme-color", "#ff0000");  // CSS custom properties

// Remove inline style
el.style.removeProperty("color");
el.style.color = "";  // also removes it

// Computed style (what's actually applied, including from stylesheets)
window.getComputedStyle(el).color;
window.getComputedStyle(el).getPropertyValue("--theme-color");

// PREFERRED — toggle CSS classes (keeps styling in CSS where it belongs)
el.classList.toggle("dark-mode");
el.classList.add("error");
el.classList.remove("loading");

// Practical dark mode toggle
document.body.classList.toggle("dark");
localStorage.setItem("theme", document.body.classList.contains("dark") ? "dark" : "light");
```

---

## Q116. How to Create New Elements in DOM?

```js
// Basic creation and insertion
const card = document.createElement("div");
card.className = "card";
card.textContent = "Hello";
document.body.appendChild(card);  // insert at end of body

// Better: set up before inserting (minimize reflows)
const ul = document.createElement("ul");
const items = ["A", "B", "C"];

// Use DocumentFragment to batch insertions
const frag = document.createDocumentFragment();
items.forEach(text => {
  const li = document.createElement("li");
  li.textContent = text;
  frag.appendChild(li);
});
ul.appendChild(frag);  // single DOM update

// insertAdjacentHTML — insert HTML string at a specific position
el.insertAdjacentHTML("beforebegin", "<div>before el</div>");
el.insertAdjacentHTML("afterbegin",  "<div>first child</div>");
el.insertAdjacentHTML("beforeend",   "<div>last child</div>");
el.insertAdjacentHTML("afterend",    "<div>after el</div>");
```

---

## Q117. Difference between `createElement()` and `cloneNode()`?

```js
// createElement() — brand new empty element
const el = document.createElement("div");

// cloneNode(deep) — copies an existing element
const original = document.querySelector(".template-card");
const shallow = original.cloneNode(false);  // copies element, NOT children
const deep    = original.cloneNode(true);   // copies element AND all descendants

// Important: cloneNode does NOT copy event listeners attached via addEventListener
// It DOES copy inline event handlers (onclick="...") — but you shouldn't use those

// Use case — template pattern
const template = document.querySelector("#item-template");
function createItem(data) {
  const clone = template.cloneNode(true);
  clone.querySelector(".name").textContent = data.name;
  clone.querySelector(".price").textContent = data.price;
  return clone;
}
```

---

## Q118. Difference between `createElement()` and `createTextNode()`?

```js
// createElement — creates an HTML element node
const p = document.createElement("p");

// createTextNode — creates a pure text node (no HTML parsing)
const text = document.createTextNode("Hello <world>");  // angle brackets are literal
p.appendChild(text);
// result: <p>Hello &lt;world&gt;</p> — safely escaped

// Comparison with textContent and innerHTML
p.textContent = "Hello <world>";  // same safe result as createTextNode
p.innerHTML = "Hello <world>";    // ❌ parses the < and > as HTML
```

`createTextNode` is the safest way to inject user-generated text into the DOM — it's equivalent to `el.textContent =` but useful when building nodes programmatically.

---

## Q119. What is Error Handling in JavaScript?

> Core covered in JSQA.md Q18. Key additions for 4.5 YOE:

### Error handling layers in a real app
```
User Input Validation  →  throw custom ValidationError
API Request            →  try/catch around await fetch()
Global uncaught errors →  window.onerror / window.addEventListener("unhandledrejection")
React component errors →  Error Boundaries
Angular service errors →  HttpInterceptor
```

```js
// Async error handling — the most common modern pattern
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);

    // fetch doesn't throw for 4xx/5xx — must check manually
    if (!res.ok) throw new Error(`HTTP ${res.status}: ${res.statusText}`);

    return await res.json();
  } catch (err) {
    if (err instanceof NetworkError) {
      // handle network failure
    } else {
      // handle other errors
    }
    throw err;  // re-throw so caller can handle too
  } finally {
    setLoading(false);  // always hides the loader
  }
}
```

---

## Q120. Role of the `finally` Block?

```js
// finally runs ALWAYS — even after return or throw
function riskyOp() {
  try {
    return "success";
  } catch (e) {
    return "error";
  } finally {
    console.log("cleanup");  // always runs, before the return
  }
}
riskyOp(); // logs "cleanup", returns "success"

// finally overrides return (tricky interview question)
function test() {
  try {
    return 1;
  } finally {
    return 2;  // ⚠️ overrides the try's return!
  }
}
test(); // 2
```

**Use cases for `finally`:**
- Hide loading spinners: `setLoading(false)`
- Close database connections
- Release file handles
- Re-enable form buttons after submission attempt

---

## Q121. What is the `throw` Statement?

```js
// Throw any value — but always throw Error objects in production
throw "string error";        // ❌ loses stack trace
throw 404;                   // ❌ same problem
throw new Error("message");  // ✅ includes stack trace, message, name

// Custom error classes — clean pattern for domain errors
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

class ApiError extends Error {
  constructor(statusCode, message) {
    super(message);
    this.name = "ApiError";
    this.statusCode = statusCode;
  }
}

// Usage
function validateEmail(email) {
  if (!email.includes("@")) {
    throw new ValidationError("email", "Invalid email format");
  }
}

try {
  validateEmail("notanemail");
} catch (e) {
  if (e instanceof ValidationError) {
    showFieldError(e.field, e.message);
  } else {
    throw e;  // re-throw unexpected errors
  }
}
```

---

## Q122. What is Error Propagation?

Errors propagate **up the call stack** until caught. If no `catch` intercepts them, they become unhandled exceptions.

```js
function c() { throw new Error("deep error"); }
function b() { c(); }               // no catch — propagates up
function a() { b(); }               // no catch — propagates up

try { a(); }                         // caught here
catch (e) { console.log(e.message); } // "deep error"

// Async propagation
async function fetchData() {
  const res = await fetch("/bad-url");  // throws on network error
  return res.json();                    // throws on invalid JSON
}

async function loadPage() {
  const data = await fetchData();       // propagates up
  render(data);
}

// Caught at the top level
loadPage().catch(err => showErrorPage(err));
```

**Re-throwing pattern:** Catch only what you can handle. Re-throw everything else.
```js
catch (e) {
  if (e instanceof ValidationError) {
    handleValidation(e);
  } else {
    throw e;  // unexpected — let it propagate to global handler
  }
}
```

---

## Q123. Best Practices for Error Handling?

```js
// 1. Use specific error types
throw new ValidationError("email", "Invalid format");

// 2. Never swallow errors silently
catch (e) { }  // ❌ black hole — hides bugs

// 3. Log errors with context
catch (e) {
  console.error("[loadUsers]", e.message, { userId, timestamp: Date.now() });
  Sentry.captureException(e);  // monitoring service
}

// 4. Show user-friendly messages, not technical details
catch (e) {
  setError("Unable to load data. Please try again.");  // ✅
  // NOT: setError(e.message)  ← exposes internals
}

// 5. Handle async errors at appropriate layers
// Low-level utilities: throw errors
// Mid-level services: catch, enrich, re-throw
// UI components: catch, display user message

// 6. Set up global handlers
window.onerror = (msg, src, line, col, err) => {
  Sentry.captureException(err);
};
window.addEventListener("unhandledrejection", event => {
  Sentry.captureException(event.reason);
});
```

---

## Q124. Types of JavaScript Errors?

| Error Type | When it occurs | Example |
|-----------|---------------|---------|
| `SyntaxError` | Parse time — invalid JS syntax | `if (` with no closing `)` |
| `ReferenceError` | Runtime — accessing undeclared variable | `console.log(x)` where `x` not declared |
| `TypeError` | Runtime — wrong operation for the type | `null.toString()`, calling non-function |
| `RangeError` | Runtime — value outside valid range | `new Array(-1)`, recursive stack overflow |
| `URIError` | Malformed URI | `decodeURIComponent("%")` |
| `EvalError` | Legacy — `eval()` misuse | Rare in modern code |

```js
// TypeError is the most common in frontend
null.property;         // TypeError: Cannot read property of null
undefined();           // TypeError: undefined is not a function
[].map.call(null, fn); // TypeError

// ReferenceError
console.log(x);        // ReferenceError: x is not defined
// BUT: typeof x       // "undefined" — safe even for undeclared variables

// SyntaxError — caught before code runs, can't be try-caught
try { eval("if ("); }  // SyntaxError inside eval CAN be caught
catch (e) { console.log(e instanceof SyntaxError); } // true
```

---

## Q125. Objects in JavaScript (Deep Dive)?

> Core covered in JSQA.md Q15. Key internals for 4.5 YOE:

### Property enumeration
```js
const obj = { a: 1, b: 2 };

// for...in iterates own AND inherited enumerable properties
for (const key in obj) {
  if (obj.hasOwnProperty(key)) {  // filter inherited
    console.log(key);
  }
}

// Object.keys/values/entries — own enumerable only (preferred)
Object.keys(obj);    // ["a","b"]
Object.values(obj);  // [1, 2]
Object.entries(obj); // [["a",1],["b",2]]

// Symbol keys are NOT included in any of the above
const sym = Symbol("id");
obj[sym] = 99;
Object.keys(obj);                    // ["a","b"] — no symbol
Object.getOwnPropertySymbols(obj);   // [Symbol(id)]
Reflect.ownKeys(obj);                // ["a","b", Symbol(id)] — all keys
```

---

## Q126. Ways to Create Objects?

> Core covered in JSQA.md Q15 and phase3.md Q99. Key for interviews:

### Private fields with classes (ES2022)
```js
class BankAccount {
  #balance = 0;  // private field — truly inaccessible outside class

  deposit(amount) { this.#balance += amount; }
  get balance()   { return this.#balance; }
}

const account = new BankAccount();
account.deposit(100);
account.balance;    // 100
account.#balance;   // SyntaxError — cannot access outside class
```

### Object.create() for prototypal inheritance
```js
const animal = {
  breathe() { return `${this.name} is breathing`; }
};

const dog = Object.create(animal);
dog.name = "Rex";
dog.bark = function() { return "Woof!"; };

dog.breathe(); // "Rex is breathing" — inherited
dog.bark();    // "Woof!" — own method
Object.getPrototypeOf(dog) === animal; // true
```

---

## Q127. Difference between Array and Object?

```js
// Array — ordered, indexed, for lists of similar items
const skills = ["JS", "React", "Angular"];
skills[0];             // "JS"
skills.push("Node");
skills.map(s => s.toLowerCase());

// Object — named keys, for structured entities
const user = { name: "Alice", age: 30, role: "admin" };
user.name;             // "Alice"
user["role"];          // "admin"

// typeof is the same for both
typeof [];   // "object"
typeof {};   // "object"

// Correct checks
Array.isArray([]);                          // true
Object.prototype.toString.call([]);         // "[object Array]"
Object.prototype.toString.call({});         // "[object Object]"
```

**When to use array vs object:**
- Multiple items of the same kind → Array: `users`, `products`, `orders`
- Structured data for one entity → Object: `user`, `config`, `form`
- Lookup by key → Object or Map: `{ [id]: user }` for O(1) lookup

---

## Q128. Add, Modify, Delete Object Properties?

```js
const user = { name: "Alice", age: 30 };

// Add
user.email = "alice@example.com";
user["role"] = "admin";

// Modify
user.age = 31;

// Delete
delete user.email;   // returns true, property removed
"email" in user;     // false

// Conditional add — only if not already present
user.createdAt ??= new Date();  // nullish assignment

// Batch update with Object.assign
Object.assign(user, { city: "Pune", country: "India" });

// Immutable update (React pattern)
const updated = { ...user, age: 31 };  // new object
```

**`delete` vs setting to `undefined`:**
```js
delete user.email;     // removes the key entirely — "email" in user → false
user.email = undefined; // key still exists — "email" in user → true, value is undefined
```

---

## Q129. Dot Notation vs Bracket Notation?

> Core covered in JSQA.md Q15 and phase3.md Q100. Key patterns:

```js
// Dynamic property access — bracket only
function getProperty(obj, key) { return obj[key]; }

// Nested dynamic access
function deepGet(obj, path) {
  return path.split(".").reduce((curr, key) => curr?.[key], obj);
}
deepGet(user, "address.city"); // safe nested access

// Common pattern with event.target
form.addEventListener("change", e => {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value })); // computed key
});
```

---

## Q130. Ways to Iterate Over Object Properties?

```js
const user = { name: "Alice", age: 30 };

// 1. for...in — own + inherited enumerable (use hasOwnProperty filter)
for (const key in user) {
  if (Object.hasOwn(user, key)) console.log(key, user[key]);
}

// 2. Object.keys().forEach() — own enumerable keys only
Object.keys(user).forEach(key => console.log(key, user[key]));

// 3. Object.entries() — pairs, chainable with array methods
Object.entries(user).map(([k, v]) => `${k}: ${v}`);

// 4. Object.values() — values only
Object.values(user).filter(v => typeof v === "string");

// Transform pattern
const uppercased = Object.fromEntries(
  Object.entries(user).map(([k, v]) => [k, String(v).toUpperCase()])
);
```

`Object.hasOwn(obj, key)` is the modern replacement for `obj.hasOwnProperty(key)` — it works even if `hasOwnProperty` has been overridden on the object.

---

## Q131. Check if a Property Exists in an Object?

```js
const user = { name: "Alice", age: undefined };

// 1. in operator — checks own AND prototype chain
"name" in user;        // true
"toString" in user;    // true — inherited from Object.prototype!

// 2. hasOwnProperty / Object.hasOwn — own properties only
user.hasOwnProperty("name");  // true
Object.hasOwn(user, "name");  // true (ES2022, prefer this)
Object.hasOwn(user, "toString"); // false — not own property

// 3. !== undefined check — unreliable!
user.age !== undefined;  // false — but "age" IS in the object!
user.age;                // undefined — age exists but value is undefined

// Best rule:
// "does the key exist?" → use `in` or `Object.hasOwn`
// "does it have a meaningful value?" → check value: user.name ?? "default"
```

---

## Q132. How to Clone/Copy an Object?

> Core covered in JSQA.md Q21. Comprehensive reference:

```js
const user = { name: "Alice", address: { city: "Pune" } };

// Shallow copies — nested objects still shared
const s1 = { ...user };
const s2 = Object.assign({}, user);

s1.address.city = "Mumbai";
console.log(user.address.city); // "Mumbai" — shared reference!

// Deep copies
// structuredClone — modern, handles Date/Map/Set/RegExp/circular refs
const d1 = structuredClone(user);

// JSON method — fast but loses: functions, undefined, Date→string, Symbol, Map, Set
const d2 = JSON.parse(JSON.stringify(user));

// Lodash deep clone — handles edge cases
const d3 = _.cloneDeep(user); // if lodash available

// Manual deep clone — interview implementation question
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (Array.isArray(obj)) return obj.map(deepClone);
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => [k, deepClone(v)])
  );
}
```

---

## Q133. Deep Copy vs Shallow Copy?

> Core covered in JSQA.md Q21. Visual explanation:

```
// Original object in memory
user = { name: "Alice", address: { city: "Pune" } }
         |name| → "Alice"    (primitive — always copied by value)
         |address| → [REF] → { city: "Pune" }

// Shallow copy — copies first level, shares nested
copy = { ...user }
         |name| → "Alice"     (new copy of string)
         |address| → [REF] → { city: "Pune" }  ← SAME object!

// Deep copy — fully independent
clone = structuredClone(user)
         |name| → "Alice"           (copy)
         |address| → [NEW REF] → { city: "Pune" }  ← different object
```

**React implication:** `setState({ ...prev })` is a shallow update — if you modify nested state, you must spread at every level that changes:
```js
setUser(prev => ({
  ...prev,
  address: {
    ...prev.address,
    city: "Mumbai"  // ✅ new reference at every level
  }
}));
```

---

## Q134. What is a Set?

> See phase5.md Q149 for full coverage. Quick reference:

```js
const set = new Set([1, 2, 2, 3, 3]);
set;           // Set {1, 2, 3} — duplicates removed
set.size;      // 3
set.has(2);    // true
set.add(4);
set.delete(2);

// Most common use — deduplicate array
const unique = [...new Set([1,2,2,3,3,4])]; // [1,2,3,4]

// Set operations
const a = new Set([1,2,3]);
const b = new Set([2,3,4]);
const union        = new Set([...a, ...b]);         // {1,2,3,4}
const intersection = new Set([...a].filter(x => b.has(x))); // {2,3}
const difference   = new Set([...a].filter(x => !b.has(x))); // {1}
```

---

## Q135. What is a Map?

> See phase5.md Q150 for full coverage. Quick reference:

```js
const map = new Map();
map.set("name", "Alice");
map.set({ id: 1 }, "user obj as key");  // any type as key
map.get("name");    // "Alice"
map.has("name");    // true
map.size;           // 2

// Iterate
map.forEach((value, key) => console.log(key, value));
for (const [key, value] of map) { }

// Object → Map
const obj = { a: 1, b: 2 };
const m = new Map(Object.entries(obj));

// Map → Object
const o = Object.fromEntries(map);
```

---

## Q136. Difference between Map and Object?

| Feature | Object | Map |
|---------|--------|-----|
| Key type | String or Symbol only | Any type (objects, functions, primitives) |
| Key order | Not guaranteed (mostly insertion order in modern JS) | Guaranteed insertion order |
| Size | `Object.keys(o).length` | `map.size` |
| Iteration | `Object.entries()` | `for...of` directly |
| Prototype pollution risk | ✅ Yes (inherited keys) | ❌ No |
| JSON serializable | ✅ Yes | ❌ Not directly |
| Performance (large dynamic) | OK | Better for frequent add/delete |

**Use Map when:** keys are not strings, you need guaranteed order, or you're doing frequent additions/deletions.  
**Use Object when:** you need JSON serialization, or you're modeling a simple data record.

---

## Q137. Event Handling in JavaScript (Deep Dive)?

> Core covered in JSQA.md Q84 and phase3.md Q84. Advanced patterns:

```js
// Event options object
el.addEventListener("click", handler, {
  once:    true,    // auto-removes after first fire
  passive: true,    // tells browser handler won't call preventDefault (improves scroll perf)
  capture: true,    // use capture phase instead of bubble phase
});

// AbortController — remove multiple listeners at once
const controller = new AbortController();
document.addEventListener("click", handler1, { signal: controller.signal });
document.addEventListener("keydown", handler2, { signal: controller.signal });
controller.abort(); // removes both listeners at once

// Custom events
const event = new CustomEvent("user:login", {
  detail: { userId: 123 },
  bubbles: true,
  cancelable: true
});
document.dispatchEvent(event);
document.addEventListener("user:login", e => console.log(e.detail.userId));
```

---

## Q138. Common DOM Events?

```js
// Mouse events
"click", "dblclick", "mousedown", "mouseup",
"mouseover", "mouseout", "mousemove", "contextmenu"

// Keyboard events
"keydown",  // fires on key press, repeats while held
"keyup",    // fires on key release
// "keypress" — deprecated, avoid

// Form events
"submit",    // form submission (use preventDefault to handle with JS)
"change",    // after input loses focus with new value
"input",     // every keystroke / value change (use for live updates)
"focus",     // element gained focus
"blur",      // element lost focus
"reset"

// Window/Document events
"load",            // all resources loaded (images, scripts)
"DOMContentLoaded",// HTML parsed, DOM ready (before images) — faster
"resize",
"scroll",
"beforeunload"    // user leaving page — show confirmation dialog

// DOMContentLoaded vs load
document.addEventListener("DOMContentLoaded", () => { /* DOM ready */ });
window.addEventListener("load", () => { /* everything including images loaded */ });
```

---

## Q139. First-Class Functions?

> Core covered in JSQA.md Q85 and phase3.md Q85. Additional patterns showing why it matters:

```js
// Functions as values enable the entire React paradigm
// props as functions:
<Button onClick={handleSave} onHover={trackHover} />

// render props pattern
<DataProvider render={data => <Table rows={data} />} />

// hooks return values that include functions
const [value, setValue] = useState("");
const [data, setData, clearData] = useCustomHook();

// Functions stored in objects — strategy pattern
const validators = {
  required: val => val !== "",
  email:    val => /\S+@\S+/.test(val),
  minLen:   n => val => val.length >= n,
};

function validate(value, rules) {
  return rules.every(rule => rule(value));
}
validate(email, [validators.required, validators.email]);
```

---

## Q140. Pure and Impure Functions?

> Core covered in phase3.md Q86. Additional for 4.5 YOE:

### React's reliance on purity
```js
// React components must be pure — same props = same output
function UserCard({ user }) {
  return <div>{user.name}</div>;  // pure — no side effects in render
}

// ❌ Impure component — reads external state
function BadComponent({ id }) {
  const user = globalUsers[id];  // reads external mutable state
  return <div>{user.name}</div>;
}

// Side effects → useEffect (separated from render)
function UserCard({ id }) {
  const [user, setUser] = useState(null);

  useEffect(() => {  // side effect isolated here
    fetch(`/users/${id}`).then(r => r.json()).then(setUser);
  }, [id]);

  return user ? <div>{user.name}</div> : <Spinner />;
}
```

---

## Q141. Function Currying?

> Core covered in phase3.md Q87 and JSQA.md Q87. Interview implementation:

```js
// Generic curry — works for any function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...more) {
      return curried.apply(this, args.concat(more));
    };
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

curriedAdd(1)(2)(3);    // 6
curriedAdd(1, 2)(3);    // 6
curriedAdd(1)(2, 3);    // 6
curriedAdd(1, 2, 3);    // 6
```

---

## Q142. `call()`, `apply()`, `bind()` (Deep Dive)?

> Core covered in JSQA.md Q88 and phase3.md Q88. Classic interview questions:

```js
// Classic output question
function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}
const user = { name: "Alice" };

greet.call(user, "Hello", "!");     // "Hello, Alice!"
greet.apply(user, ["Hello", "!"]); // "Hello, Alice!"
const boundGreet = greet.bind(user, "Hello");
boundGreet("!");                    // "Hello, Alice!" — punctuation filled later

// Partial application with bind
function multiply(a, b) { return a * b; }
const double = multiply.bind(null, 2);  // first arg fixed as 2
double(5);   // 10
double(10);  // 20
```

---

## Q143. What is a String? (Recap)

Covered thoroughly in Q101, Q102, Q103, Q104, Q105, Q106.

---

## Q144. Template Literals (Recap)

Covered in Q102.

---

## Q145. Single/Double Quotes vs Backticks (Recap)

Covered in Q103.

---

## Q146. String Operations (Recap)

Covered in Q104, Q105.

---

## Q147. String Concatenation (Recap)

Covered in Q106.

---

## Q148. String Immutability (Recap)

Covered in Q105.
