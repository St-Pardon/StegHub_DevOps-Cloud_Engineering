# Basic Javascript Syntax

**1. Variables and Data Types:**

* Variables store data and are declared using keywords like `let`, `const`, or `var` (though `var` is generally discouraged due to scoping issues).
* Data types specify the kind of data a variable holds:
    * Numbers (integers or decimals) - `let age = 25;`
    * Strings (text) - `let name = "Alice";`
    * Booleans (true or false) - `let isLoggedIn = true;`
    * Arrays (ordered lists of items) - `let fruits = ["apple", "banana", "orange"];`
    * Objects (unordered collections of key-value pairs) - `let person = { name: "Bob", age: 30 };`

**2. Operators:**

* JavaScript provides various operators for performing calculations, comparisons, and logical operations:
    * Arithmetic operators: `+`, `-`, `*`, `/`, `%` (modulo)
    * Comparison operators: `==`, `!=`, `<`, `>`, `<=`, `>=`
    * Logical operators: `&&` (AND), `||` (OR), `!` (NOT)

**3. Statements and Expressions:**

* Statements are complete lines of code that perform actions. They end with a semicolon (;).
* Expressions evaluate to a value. They can be used within statements:
    * `let sum = 10 + 20;` (statement with an expression)
    * `5 * (2 + 3)` (expression calculating a value)

**4. Control Flow:**

* Control flow structures allow you to control the execution order of your code:
    * `if` statements: execute code conditionally based on a boolean expression.
    * `else` statements: provide alternative code if the `if` condition is false.
    * `for` loops: repeat a block of code a certain number of times.
    * `while` loops: repeat a block of code as long as a condition is true.

**5. Functions:**

* Functions are reusable blocks of code that perform specific tasks. They can take arguments and return values:
    ```javascript
    function greet(name) {
        console.log("Hello, " + name + "!");
    }

    greet("Charlie");
    ```

**6. Comments:**

* Comments are lines of code ignored by the browser but help developers understand the code. They can be single-line (`//`) or multi-line (`/* */`).

**7. Semicolons:**

* While technically optional in some cases, using semicolons at the end of statements is considered good practice for readability and maintainability.


**Learning Resources:**

* [https://developer.mozilla.org/en-US/docs/Web/JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
* [https://www.javascript.com/](https://www.javascript.com/)
* [https://www.freecodecamp.org/news/full-javascript-course-for-beginners/](https://www.freecodecamp.org/news/full-javascript-course-for-beginners/)


Remember, this is a basic introduction. As you progress, you'll explore more advanced concepts like object-oriented programming, asynchronous programming, and the Document Object Model (DOM) for interacting with web pages.  Feel free to ask if you have any questions about specific parts of JavaScript syntax!