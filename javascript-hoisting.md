Alright, let's corral this concept with a visual aid using Mermaid. We'll show the two main phases the JavaScript engine goes through: **Parsing/Hoisting** and **Execution**, and how different types of declarations are handled.

```mermaid
graph TD
    %% Define node styles
    classDef phase fill:#ffc,stroke:#333,stroke-width:2px;
    classDef declaration fill:#d3d3d3,stroke:#333;
    classDef memoryState fill:#e0e0e0,stroke:#666;
    classDef error fill:#f9f,stroke:#333,stroke-width:2px;
    classDef success fill:#9f9,stroke:#333,stroke-width:2px;
    classDef outcome fill:#cff,stroke:#333;

    %% Nodes for phases
    A[Your JavaScript Code] --> B{1. Parsing / Hoisting Phase};
    B --> C{2. Execution Phase};

    %% What happens during Hoisting
    B --> D1[var myVar;]:::declaration
    D1 -- Variable declared --> E1(Memory:<br/>myVar = <span style='color: DodgerBlue;'>undefined</span>):::memoryState;

    B --> D2[let myLet;]:::declaration
    D2 -- Variable declared,<br/>placed in --> E2(Memory:<br/>myLet <span style='color: GoldenRod;'>(TDZ)</span>):::memoryState;

    B --> D3[const myConst;]:::declaration
    D3 -- Variable declared,<br/>placed in --> E3(Memory:<br/>myConst <span style='color: GoldenRod;'>(TDZ)</span>):::memoryState;

    B --> D4[function myFunction() {...}]:::declaration
    D4 -- Declared &<br/>fully defined --> E4(Memory:<br/>myFunction = <span style='color: DarkViolet;'>function definition</span>):::memoryState;

    B --> D5[var funcExpVar = function() {...}]:::declaration
    D5 -- Variable declared --> E5(Memory:<br/>funcExpVar = <span style='color: DodgerBlue;'>undefined</span>):::memoryState;
    %% The function definition itself is NOT assigned yet for function expressions

    B --> D6[let funcExpLet = function() {...}]:::declaration
    D6 -- Variable declared,<br/>placed in --> E6(Memory:<br/>funcExpLet <span style='color: GoldenRod;'>(TDZ)</span>):::memoryState;
    %% The function definition itself is NOT assigned yet

    %% What happens during Execution (accessing before declaration line)
    C --> F1["Attempt: Access myVar<br/>(before var line)"]
    F1 -- Value is --> G1[<span style='color: DodgerBlue;'>undefined</span>]:::outcome;

    C --> F2["Attempt: Access myLet/myConst<br/>(before let/const line)"]
    F2 -- Throws --> H1(<span style='color:red;'>ReferenceError</span><br/>(In TDZ)):::error;

    C --> F3["Attempt: Call myFunction()<br/>(before function declaration line)"]
    F3 -- Executes --> I1(<span style='color:green;'>Success!</span><br/>Function runs):::success;

    C --> F4["Attempt: Access funcExpVar<br/>(before var line)"]
    F4 -- Value is --> G2[<span style='color: DodgerBlue;'>undefined</span>]:::outcome;

    C --> F5["Attempt: Call funcExpVar()<br/>(before var line)"]
    F5 -- Throws --> H2(<span style='color:red;'>TypeError</span><br/>(undefined is not a function)):::error;

    C --> F6["Attempt: Access funcExpLet<br/>(before let line)"]
    F6 -- Throws --> H3(<span style='color:red;'>ReferenceError</span><br/>(In TDZ)):::error;

    C --> F7["Attempt: Call funcExpLet()<br/>(before let line)"]
    F7 -- Throws --> H4(<span style='color:red;'>ReferenceError</span><br/>(In TDZ)):::error;

    %% What happens during Execution (when declaration/assignment lines are reached)
    C --> J1[...Reach `myVar = 99;`] --> K1(Memory:<br/>myVar = 99):::memoryState;
    C --> J2[...Reach `let myLet = 100;`] --> K2(Memory:<br/>myLet = 100<br/>(Exits TDZ)):::memoryState;
    C --> J3[...Reach `const myConst = 200;`] --> K3(Memory:<br/>myConst = 200<br/>(Exits TDZ)):::memoryState;
    C --> J4[...Reach `var funcExpVar = function() {...};`] --> K4(Memory:<br/>funcExpVar = function):::memoryState;
    C --> J5[...Reach `let funcExpLet = function() {...};`] --> K5(Memory:<br/>funcExpLet = function<br/>(Exits TDZ)):::memoryState;


    %% Style the phase nodes
    class B,C phase;

    %% Add a title (Mermaid doesn't have a dedicated title, use a subgraph or comment)
    %% title: JavaScript Hoisting Illustration
```

**Explanation of the Diagram:**

1.  **Original Code:** This is your script as written.
2.  **Parsing / Hoisting Phase (B):** Before any code runs, the engine scans the script.
    *   `var` declarations (D1, D5) lead to variables being created in memory and initialized with `undefined` (E1, E5).
    *   `let` and `const` declarations (D2, D3, D6) lead to variables being created, but they are explicitly *not* initialized and are placed in the **Temporal Dead Zone (TDZ)** (E2, E3, E6).
    *   `function` declarations (D4) lead to the entire function (name and body) being created in memory and ready (E4).
3.  **Execution Phase (C):** The engine now runs the code line by line.
    *   If you try to access a `var` variable *before* the assignment line in the code, its value will be the `undefined` it got during hoisting (F1 -> G1, F4 -> G2). If you try to *call* that `undefined` variable (thinking it's a function expression), you get a `TypeError` (F5 -> H2).
    *   If you try to access a `let` or `const` variable *before* the line where it's declared and assigned (while it's in the TDZ), you get a `ReferenceError` (F2 -> H1, F6 -> H3, F7 -> H4).
    *   If you call a `function` declaration *before* the line where it's written in the code, it works just fine because the function was fully ready during hoisting (F3 -> I1).
    *   As the execution reaches the actual declaration/assignment lines (`=` or `let`/`const` definition) (J1-J5), the variables in memory are updated with their assigned values, and `let`/`const` variables exit the TDZ (K1-K5). After this point, accessing them works as expected.

This diagram aims to show *what the engine sets up in memory* during the hoisting phase, which dictates the behavior you see during the execution phase when accessing things early.
