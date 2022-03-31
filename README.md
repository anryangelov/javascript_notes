# JavaScript Notes

---

#### 1. `+` operator

When we use `+` operator with string and number javascript will automatically convert the number to string this is called - coercion - automatic type conversion.

```javascript
> '35' + 3
'353'

> 35 + 3
38
```
 #### 2. `-` operator 
Interestingly if we use `-` operator with strings javascript automatically convert them to numbers. Somehow the opposite of + operator.

```javascript
> '4' - '3'
1
> '4' - 3
1
```
#### 3. falsy values
- undefined
- NaN
- 0
- ""
- null

Be aware that empty Object, Map, Array, etc do not return false but true.  
```javascript
> Boolean({})
true
> Boolean([])
true
```
#### 4. Number('foo') return NaN

#### 5. Comparison operators
`===` strict comparison no coercion.  
`==` loose comparison - unpredictable result, avoid this.

#### 6. Array is not a separate type. It is made from Object type
```javascript
> typeof {}
'object'
> typeof []
'object'
```
Also be aware of that typeof null return object (legacy bug)
```javascript
> typeof null
'object'
```

#### 7. Conditional operator
```javascript
> let age = 15
undefined
> age >= 18 ? 'wine': 'water'
'water'
```

#### 8. Defining function
There 3 ways to define function:
- function statement  
```javascript
function foo() {}
```
- function expression  
```javascript
const foo = function () {}
```
- arrow function expression (no this keyword)
```javascript
const foo = () => expr or {}
```

#### 9. Array operation
```javascript
> let foo = ['a', 'b', 'c']
undefined
> foo.push('foo')
4
> foo
[ 'a', 'b', 'c', 'foo' ]
```

#### 10. Runtime in the browser
It contains:
- javascript engine - program that executes javascript code contain call stack (where the code actually executed using execution context) and heap (unstructured memory pull contains all the objects)
- web APIs - DOM, Fetch API, Timers, etc. Not the part of the language itself. JS get access to this APIs through the global window object
- callback queue - contains all callback function that are ready to be executed, for example callback function from DOM event listener.
  
#### 11. Execution Context
Every function is executed in the call stack having execution context abstraction which contains:
- variable environment
    - `var`, `let`, `const` variables - all variables declared inside the function
    - Functions
    - argument object - contains function arguments
- chain scope - references to the variables that are located to the outside of the current function
- this keyword

Execution context is generated during "creating face" just before execution.

Important detail is that arrow functions do not have their own argument object and this keyword!

Call stack is actually execution contexts get stacked on top of each other to keep track of where are in the execution.

#### 12. Scope chain

Every function has scope chain - every outer parent function and finally the global scope.  
`let` and `const` variables are block scoped but `var` variables are not block scoped.
We can override variables accessed in function and declared somewhere in scope chain outside of the function.  
<sub><sup>
That's differ from python for example where we have to use `global` or `nonlocal`  in order to override variable created outside of the function.
</sub></sup>

#### 13 Hoisting

Makes some types of variables accessible/usable in the code before they are actually declared. "Variables lifted to the top of their scope". Because the code is parsed for variables and function statement before executing.

- function statement can be declared after calling them.
- `var` variable can used before declaring them but they will be undefined. That leads to a lot of bugs. Declaring variable with `var` should be avoided!
- `let` and `const` they are also use hoisting but also they go TDZ (Temporal Dead Zone) until declaration line of the variable is reached. If the variable is in TDZ error is raise different error from undeclared variable at all.
  
`const` variables work thanks to the TDZ

#### 14 This keyword
Special variable that is created for every execution context (every function). Takes the value of (points to) the owner of the function in which the `this` keyword is used. It is **NOT** static. It depends on how the function is called and its value is assigned when the function is actually called.  
Here are 4 different ways in which function can be called:
- as a method in that case `this` points to the object
```javascript
> const Joe = {
...   birthYear: 1983,
...   getAge: function () {
.....     return 2040 - this.birthYear;
.....   },
... };
> 
> console.log(Joe.getAge());
57

> const Maria = {
...   birthYear: 1999,
... };
> Maria.getAge = Joe.getAge;
> // this keyword always point to the object that calls the method
> console.log(Maria.getAge());
41
```
- simple function call - `this` will be window object but if strict mode is used `this` will be undefined 
- arrow function do not have their own `this` keyword and through out of the scope chain you will access `this` of parent scope.
```javascript
//we use strict mode
> function foo () {console.log(this)}
> foo()
undefined

> const bar = () => console.log(this)
> bar()
Object [global] {
  global: [Circular],
  ...}
```
In the second case a long global object is return since arrow function has no own this keyword and therefore we access this from global scope which points to global object
- event listener - `this` = DOM element that the handler is attached to

#### 15. Regular function vs Arrow function
Arrow function do not have their own `this` keyword. So if we use arrow function in object and use this inside this function we will access `this` of the parent scope
```javascript
> const Joe = {
...   birthYear: 1983,
...   getAge: () => {
.....     return 2040 - this.birthYear;
.....   },
... };
> Joe.getAge();
NaN
```
Yes, pretty unexpected result. We try to access `this` in  `Joe.getAge` but since this is an arrow function no dedicated `this` keyword is created. So through out the scope chain the next environment scope is the global scope where `this` is defined and point to global object.

Another case where if we defined method and within defined function and call it within the method `this` keyword will be undefined in strict mode because it is a simple function call no object is attached here.
```javascript
> const Joe = {
...   birthYear: 1983,
... 
...   outer: function () {
...     function inner() {
.....       console.log('in the inner');
.....       console.log('this is: ' + this);
.....     }
...     inner();
...   },
... };
> Joe.outer();
in the inner
this is: undefined
```

If we want in inner function `this` to point to the object Joe we can use arrow function since `this` will point to `this` of the parent scope which is the outer function.

```javascript
> const Joe = {
...   birthYear: 1983,
... 
...   outer: function () {
...     const inner = () => {
.....       console.log('in the inner');
.....       console.log('this is: ' + this);
.....     };
...     inner();
...   },
... };
> Joe.outer();
in the inner
this is: [object Object]
```
Here `this` point to Joe

Other option is:
```javascript
...
  outer: function () {
    self = this;
    function inner() {
      // here we can use self which point ot the outer this
...
````

#### 16. Primitives and Objects
primitives:
- Number
- String
- Boolean
- Undefined
- Null
- Symbol
- BigInt  

Object types (live in the heap) also called reference types:
- Object literal
- Arrays
- Function
- etc

Object types are stored in the memory heap whereas primitive types are stored in the call stack in the execution context. If we have this `let foo = {bar: 5}` that means 
foo point to address in the stack which value is another address point to the address of the object in the heap. If we do this `let ror = foo` that means that new variable is created in the stack which point to the same object in the heap. That's why if we write `roo.bar = 3` `console.log(foo.bar)` will print 3. Since we change the same object with two references pointed to him. Objects are mutable.

Function arguments are passed by value similar as assigning value to a variable if we have the following snippet
```javascript
function example (param1) {.....}

let foo = 'bar'

example(foo)
```
Calling `example` func create a new copy of `foo`. This means in the call stack will have identifier `foo` pointing to address in the stack which address holds the value `bar` and another identifier `param1` pointing to another address in the stack holding the same value `bar`. That means **changing `param1` inside the function will NOT affect `foo` in anyway**, This would happened if it was passed by reference.  
**BUT** this is the case with primitive types if we have **`let foo = {age: 30}`**  we pass an object type which lives in the heap therefore `param1` holds in the stack not the actual object but an address which leads to the heap where the object lives. **That means that changes (for example: `param1.age = 50`) to that object inside the function throughout the `param1` will be visible throughout the `foo` since it is the same object**. Actually this objected is pointed by two different places in the stack(`foo` and `param1`).  
<sub><sup>
In python is similar but every type lives in the heap even string type. Everything is object and the address of the object can be got by `id` function. But string for example are immutable. They intentionally are not allowed to be updated in place. Some other immutable objects are integer, float, tuple, and bool. Of course Dict, List, Set are mutable.
</sub></sup>  
Above can help to explain why objects are not equal in the following snippet.
```javascript
> let obj1 = {foo: 3}
> let obj2 = {foo: 3}
> obj1 === obj2 // compare object addresses in the heap not the actual data in that case
false
```

#### 17 More for functions

Functions are first class citizen which means that they can be passed as arguments to a function and function can return function. They are also objects therefore they have methods:
- call - the first argument replace this.
- apply - same as call but we pass array after first argument but not used so much today because of the spread `...` operator.
- bind - return function with embedded `this` provided as first argument when bind is called. Same as partial update in Python but also the first argument replace this.
```javascript
const module = {
  x: 42,
  getX: function() {
    return this.x;
  }
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // The function gets invoked at the global scope
// expected output: undefined

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX());
// expected output: 42
```
