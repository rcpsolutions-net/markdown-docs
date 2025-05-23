```markdown
# JavaScript Hoisting: It's Not Magic, Just Weird

Hoisting isn't some mystical force; it's how JavaScript's parser works before execution. Essentially, `var` declarations and function declarations are "lifted" to the top of their containing scope. Assignments, however, are not. Get it straight.

## 1. `var` - The Old Guard

`var` declarations are hoisted and initialized with `undefined`. Assignments stay put. This is why people complain about JS.

```javascript
console.log(myVar); // undefined
var myVar = 'I was hoisted!';
console.log(myVar); // I was hoisted!

// What JS "sees" at runtime:
// var myVar; // Declared and initialized to undefined
// console.log(myVar);
// myVar = 'I was hoisted!';
// console.log(myVar);
```

## 2. `let` & `const` - The New Hotness (with TDZ)

`let` and `const` *are* hoisted, but they aren't initialized. Accessing them before their declaration results in a `ReferenceError` – this is the Temporal Dead Zone (TDZ). It's a good thing, preventing stupid mistakes.

```javascript
console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
let myLet = 'I\'m also hoisted, but you can\'t touch me yet!';

// What JS "sees" at runtime:
// let myLet; // Hoisted, but uninitialized and in TDZ
// console.log(myLet); // TDZ prevents access
// myLet = 'I\'m also hoisted, but you can\'t touch me yet!';
```

## 3. Functions - Declarations vs. Expressions

*   **Function Declarations** are fully hoisted. You can call them before they appear in the code.
*   **Function Expressions** (like `const myFunction = function() {}`) behave like `var`, `let`, or `const` variables. Only the variable declaration is hoisted, not the function assignment.

```javascript
hoistedFunction(); // "I'm a hoisted function declaration!"

function hoistedFunction() {
  console.log("I'm a hoisted function declaration!");
}

// This will error if myExpression is 'let' or 'const', or be undefined if 'var'
try {
  notHoistedFunction();
} catch (e) {
  console.log(e.message); // 'notHoistedFunction is not a function' or 'Cannot access...' 
}

const notHoistedFunction = function() {
  console.log("I'm a function expression, behave like a variable!");
};

notHoistedFunction(); // "I'm a function expression, behave like a variable!"
```

## The Gist (Simplified)

```mermaid
graph TD
    A[Code Parsing]
    B{Hoisting Phase}
    C[var declarations & Function declarations]
    D[let/const declarations]
    E[Assignments & Executable Code]

    A --> B
    B --> C
    B --> D
    B --> E

    C -- moved to top of scope --> F[Initialized with undefined / Fully accessible]
    D -- moved to top of scope --> G[In Temporal Dead Zone (TDZ)]
    E -- stays in place --> H[Executes in order]

    F --> I[Execution]
    G --> J{Access before declaration?}
    J -- Yes --> K[ReferenceError]
    J -- No --> I
    H --> I
```

    subgraph Legend
        direction LR
        L1[var: undefined, accessible]
        L2[let/const: TDZ, not accessible]
        L3[Function Decl: fully accessible]
        L4[Function Expr: variable rules apply]
    end
```

It's not that hard. Understand the rules, and you won't write garbage code.
```
