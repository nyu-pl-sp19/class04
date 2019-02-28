# Class 4

## Subroutines and Function/Procedural Abstraction

Subroutines provide the basic abstraction mechanism in programs. They
can be used to abstract over the specific values bound to variables
when evaluating an expression. One often distinguishes between
*functions* and *procedures* as two different kinds of subroutines:

* *Functions* correspond to the mathematical notion of computation,
  i.e. they are viewed as mapping from input to output values. They
  can be viewed as abstractions of side-effect free expressions.

* *Procedures* can be viewed as abstractions over statements. That is,
  they affect the environment (mutable variables, hard disk, network,
  ...), and are called for their side-effects.

* A pure functional model is possible but rare (e.g. Haskell)

* Hybrid model is most common: functions can have (limited) side effects

### Activation Records

Recall that subroutine calls rely heavily on use of the
*stack*.

Each time a subroutine is called, space on the stack is allocated for
the objects needed by the subroutine. This space is called a *stack
frame* or *activation record*.

The *stack pointer* contains the address of either the last used
location or the next unused location on the stack.

The *frame pointer* points into the activation record of a subroutine
so that any objects allocated on the stack can be referenced with a
static offset from the frame pointer.

Question: Why not use an offset from the stack pointer to reference
subroutine objects?

Answer: There may be objects that are allocated on the stack whose
size is unknown at compile time.

These objects get allocated last so that objects whose size is known
at compile time can still be accessed quickly via a known offset from
the frame pointer. Example:

```c
 void foo (int size) {
   char arr[size]; // allocates char array of length size on the stack
   ...
 }
```

### Managing Activation Records

When a subroutine is called, a new activation record is created and
populated with data.

The management of this task involves both the *caller* and the
*callee* and is referred to as the *calling sequence*.

* The *prologue* refers to activation record management code executed
  at the beginning of a subroutine call.

* The *epilogue* refers to activation record management code executed
  at the end of a subroutine call.

#### Calling Sequence

Upon calling a subroutine, the prologue has to perform the
following tasks:

* Pass parameters
* Save return address
* Update static chain (only needed for certain forms of nested subroutines)
* Change program counter
* Move stack pointer
* Save register values, including frame pointer
* Move frame pointer
* Initialize objects

Upon returning from the subroutine, the epilogue has to perform the
following tasks:

* Finalize (destroy) objects
* Pass return value(s) back to caller
* Restore register values, including frame pointer
* Restore stack pointer
* Restore program counter

Question: Are there advantages to having the caller or callee perform various tasks?

Answer: If possible, have the callee perform tasks: task code needs to
occur only once, rather than at every call site.

Some tasks (e.g. parameter passing) must be performed by the caller.


Typical calling sequence:

Stack (before call to subroutine):

```
|                          |
├──────────────────────────┤<- stack pointer
| Caller activation record |
├──────────────────────────┤<- frame pointer
| ...                      |
```

Step 1: Save caller-save registers:

```
|                          |
├──────────────────────────┤<- stack pointer
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
├==========================|<- frame pointer
| ...                      |
```

Step 2: Push arguments on stack:

```
|                          |
├==========================|<- stack pointer
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
├──────────────────────────┤<- frame pointer
| ...                      |
```

Step 3: Jump to subroutine, saving return address on stack

```
|                          |
├==========================|<- stack pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
├──────────────────────────┤<- frame pointer
| ...                      |
```

Step 4: Save caller's fp, set new fp

```
|                          |
├──────────────────────────┤<- stack pointer
| Saved fp (dynamic link)  |
├==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
├==========================|
| ...                      |
```

Step 5: Save callee-saved registers

```
|                          |
├──────────────────────────┤<- stack pointer
| Callee-saved registers   |
├──────────────────────────┤
| Saved fp (dynamic link)  |
|==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|
| ...                      |
```

Step 7: Allocate and initialize locals

```
|                          |
├──────────────────────────┤<- stack pointer
| Local variables          |
├──────────────────────────┤
| Callee-saved registers   |
├──────────────────────────┤
| Saved fp (dynamic link)  |
|==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|
| ...                      |
```

Step 7: Push and pop temporaries

```
| Temporaries              |
├──────────────────────────┤<- stack pointer
| Local variables          |
├──────────────────────────┤
| Callee-saved registers   |
├──────────────────────────┤
| Saved fp (dynamic link)  |
|==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|
| ...                      |
```

Step 8: Pop locals

```
|                          |
├──────────────────────────┤<- stack pointer
| Callee-saved registers   |
├──────────────────────────┤
| Saved fp (dynamic link)  |
|==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|
| ...                      |
```


Step 9: Restore callee-saved registers

```
|                          |
├──────────────────────────┤<- stack pointer
| Saved fp (dynamic link)  |
|==========================|<- frame pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|
| ...                      |
```


Step 10: Restore caller's fp

```
|                          |
|==========================|<- stack pointer
| Return address           |
├──────────────────────────┤
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|<- frame pointer
| ...                      |
```

Step 11: Jump to return address

```
|                          |
├──────────────────────────┤<- stack pointer
| Arguments                |
├──────────────────────────┤
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|<- frame pointer
| ...                      |
```

Step 12: Pop arguments

```
|                          |
├──────────────────────────┤<- stack pointer
| Caller-saved registers   |
├──────────────────────────┤
| Caller activation record |
|==========================|<- frame pointer
| ...                      |
```

Step 13: Restore caller-saved registers

```
|                          |
├──────────────────────────┤<- stack pointer
| Caller activation record |
|==========================|<- frame pointer
| ...                      |
```


Summary of calling sequence:

Prologue (caller)

1. Save caller-save registers      
2. Push arguments on stack         
3. Jump to subroutine, saving
   return address on stack

Prologue (callee)

4. Save old fp, set new fp
5. Save callee-save registers
6. Allocate and initialize locals Execute callee
7. Push and pop temporaries

Epilogue (callee)

8. Pop locals
9. Restore callee-save registers
10. Restore frame pointer
11. Jump to return address

Epilogue (caller)

12. Pop arguments
13. Restore caller-save registers


##### Saving Registers

One difficult question is whether the caller or callee should be in
charge of saving registers.

Question: What would the caller have to do to ensure proper saving of
registers?

Answer: Save all registers currently being used by caller.

Question: What would the callee have to do to ensure proper saving of
registers?

Answer: Save all registers that will be used by callee.

Question: Which is better?

Answer: Could be either one -- no clear answer in general. In
practice, many processors (including MIPS and x86) compromise: half
the registers are caller-save and half are callee-save.

To see the rational for this design, let us first recall some
background from basic computer systems architecture: a CPU has a
limited number of registers for storing the operands and results of
the executed machine instructions (arithmetic and logical operations,
comparisons, etc.) as well as for storing intermediate results of
computations (some of which may correspond to local variables in 
high-level code).

 
An important step during compilation of high-level language code to
machine language code is *register allocation*. This is the phase
where the compiler decides for each value computed in a function in
which register it should be held, respectively, if there are not
enough registers to hold all values, whether the value should be
stored on the stack (i.e. in RAM) and only be loaded into registers
when it is needed. The compiler will optimize for avoiding the use of
the stack as much as possible because RAM accesses are much slower
than register accesses. On a typical modern architecture, a register
access can be in the order of 100 times faster than a RAM access,
depending on the state of caches.

Now, when the compiler does register allocation for a particular
function `f`, one of the factors that enters the equation is the callee
portion of the calling sequence for when `f` itself is called, as well
as the caller portion of the calling sequences for calls that `f` makes
to other functions. All of this calling sequence code is going to be
part of the binary code that make up the compiled function `f`.

 What does the compiler know when it has decided on a particular
 register allocation for `f`:

1. it knows all the registers whose values the execution of `f` may modify

2. at each point where another function is called in `f`, it knows which
   values that are stored in registers before the call will still be
   needed by `f` after the call

Thus, suppose we consider a scenario where the calling sequence leaves
all the work of saving registers to the callees. In this case, the
callee portion of the calling sequence code in `f` would be responsible
for saving and restoring all registers that are modified by the
execution of `f`. This is because if we look at `f` in isolation, we don't
know which of the original values stored in the registers modified by
`f`, the caller of `f` will still need after `f` returns. In the worst case,
it will need all of them. So `f` would have to save and restore them
all. If the caller only needs a subset of the modified registers, `f`
would potentially be doing a lot of unnecessary work (involving RAM
accesses which are expensive)

The other extreme scenario is to leave all the work of saving the
registers to the callers. In this case, for every call to another
function that `f` makes, the caller portion of the calling sequence for
that call would be responsible for saving and restoring all the
registers whose values `f` still needs after the call. Again, this is
because if we look at `f` in isolation, we don't know which of those
registers will be modified by the callee. In the worst case, all of
them will be modified. So `f` would have to save and restore them
all. If the callee only modifies a subset of the registers that `f`
still needs after the call, `f` would again be doing a lot of
unnecessary work.

 
For this reason, many architectures make a compromise where some of
the registers are caller-saved and others are callee-saved. This way,
the compiler can optimize register allocation so that it minimizes the
number of registers that need to be saved onto the stack while `f` is
executed (there are other optimization goals factoring into the
equation, but this is one of them). In particular, if we view `f` from
the perspective of being a callee, we'd like to minimize the number of
callee-saved registers that `f` modifies because those are the ones that
`f` would have to save and restore. If `f` only modifies caller-saved
registers, then `f` does not need to save/restore any registers in the
callee portion of its calling sequence (because all the work now falls
onto `f`'s caller). 

On the other hand, if we now view `f` as a caller
of another function `g`, then we want to
minimize the number of caller-saved registers whose values `f` will
still need after the call to `g` returns. If `f` no longer needs any of the
values in these registers after the call, then it will not have to do
any work for saving/restoring them in `f`'s caller portion of the
calling sequence for that call to `g`. So it is now possible for `f` to both
modify registers and use registers across calls that it makes itself
without ever having to save/restore any of these registers.

You might ask: why not just solve register allocation for the whole
program at once instead of looking at one function at a time (making
the distinction between caller/callee-saved registers mood)? The
compiler will do a little bit of this kind of global
optimization. However, compilation is inherently modular. For
instance, a program that you write will often be linked against
precompiled libraries. So register allocation for the function in the
library code happens at a different time than register allocation for
the program's code. The two can't be solved at the same time.

Even if one compiles all code (program + libraries) at the same time,
a global optimization is usually not possible. The register allocation
problem can be reduced to the graph coloring problem, which is
NP-complete. So solving register allocation precisely and globally for
a large program is computationally infeasible. This is why a more
modular approach needs to be used to keep the time needed for
compilation reasonable. One approach is to solve register allocation
per function in isolation. By distinguishing between caller-saved and
callee-saved registers, the compiler can make informed decisions when
doing the optimization modularly, i.e., where it does not have perfect
information about the entire program.


*Register windows* offer an alternative to architectures that split
between caller/callee-saved registers: each routine has access only to
a small *window* of a large number of registers; when a subroutine is
called, the window moves, overlapping a bit to allow parameter
passing.


#### Optimizations

##### Leaf routines

A *leaf routine* is one which does not call any subroutines.

Leaf routines can avoid pushing the return address on the stack: it
can just be left in a register.

If a leaf routine is sufficiently simple (no local variables), it may
not even need a stack frame at all.

##### Inlining

Another optimization is to *inline* a function: inserting the code for
the function at every call site.

Question: What are advantages and disadvantages of inlining?

* Advantages: avoid overhead, enable more compiler optimizations

* Disadvantages: increases code size, can't always do inlining
  (e.g. recursive procedures)

### Evaluation Strategy and Parameter Passing Modes

We distinguish between two types of parameters:

* *Formal parameters*: these are the names that appear in the
declaration of the subroutine. 

* *Actual parameters* or *arguments*: these refer to the expressions
  passed to a subroutine at a particular call site.

```scala
  // formal parameters: a, b, c
  def f (a: Int, b: Int, c: Int): Unit = ...

  // actual parameters: i, 2/i, g(i,j)
  f(i, 2/i, g(i,j))
```

An important consideration for the execution of a subroutine call is:
what does a reference to a formal parameter in the subroutine mean in
terms of the actual parameters?

The answer to this question depends on the *evaluation strategy* for
the actual parameter. We distinguish between two basic strategies:

* *Strict evaluation*: the actual parameter is evaluated before the
call to the function

* *Lazy evaluation*: the actual parameter is evaluated only if and
when its value is needed during execution of the call.

Each evaluation strategy additionally supports different *parameter
passing modes*. Many languages support multiple parameter passing
modes. These modes are specified per parameter of a subroutine, either by
adding *qualifiers* to the parameter that indicate the mode or by
dedicated parameter types.

Parameter passing modes for strict evaluation:

* *by value*: 

  * formal is bound to fresh memory location that holds a copy of
    the value of the actual parameter
  * assignment to formal, if allowed, changes value at location of
    local copy, not at location of the actual that stores the
    original value.

  Most languages (including Scala/Java/C) use call-by-value semantics
  for parameters by default. Here is an example in `C`:

  ```c
  void incr (int x) {
    x = x + 1;
  }
    
  int counter = 0;
    
  incr(counter); /* passes copy of value stored in counter */
  printf("%d", counter); // prints 0 since incr only increments local copy
  ```

* *by reference*: 

  * only applicable if actual parameter is an l-value
  * formal is bound to the memory location that holds the original
    actual parameter value, forming an alias
  * assignment to formal, if allowed, also affects memory location of actual

  An example of a language that supports call-by-reference parameters
  is C++. Here, the fact that a parameter is passed by reference is
  indicated using a so-called *reference type*:

  ```c++
  void incr (int& x) {
    x = x + 1;
  }

  int counter = 0;

  // compiler knows declaration of incr,
  // builds reference
  incr(counter);
  std::cout << counter; // prints 1
  ```

  The type `int&` of parameter `x` in `incr` indicates that `x` is a
  reference to a memory location that holds a value of type `int`.

* *by copy-return*: 

  * formal is bound to copy of value of actual
  * upon return from routine, actual gets copy of formal
  
  Pass by cop-return is fairly uncommon in modern programming
  languages. We include it here only for completeness.
  
Parameter passing modes for lazy evaluation:
  
* *by name*: 

  * formal is bound to expression for actual
  * expression is (re)evaluated each time formal is read when
    executing callee
  * can be viewed as textually substituting every occurrence of the
    formal parameter in the body of the subroutine by the expression
    of the actual parameter
  * formal parameter cannot be assigned in body of subroutine
    
  Scala supports call-by-name parameters which similar to reference
  parameters in C++ are indicated by dedicated parameter types:
    
  ```scala
  var debugEnabled = false;
  
  def debug(msg: => String): Unit =
    if (debugEnabled) println(msg)
  
  // some expensive operation that generates a debug message
  def complexAnalysis(): String = ...
  
  // call to complexAnalysis is only executed
  // if debugEnabled is set to true
  debug(complexAnalysis())

  ```
  
  The `=>` in front of the argument type of `msg` in `debug` indicates
  that the parameter is passed by name.
    
* *by need*: 

  * formal is bound to expression for actual
  * expression is evaluated the first time its value is needed
  * subsequent reads from the formal will use the value computed earlier
  * formal parameter cannot be assigned in body of subroutine



Question: What are the advantages and disadvantages of passing by need?

* Advantage: The argument is only evaluated if it is actually used while
  executing the call.

* Disadvantage: implementation of parameter passing is more complex;
  behavior can be confusing if evaluation of actual has side
  effects. Hence, it is usually only used in purely functional languages.
  
Programming languages differ in their evaluation strategies and the
specific parameter passing modes that they support. As we have already
seen, some languages support multiple strategies and modes. Here is a
brief overview for some languages:

#### C

* Evaluation strategy is strict. The order in which actuals are
  evaluated is unspecified.

* Parameter passing is always by value: assignment to formal is
  assignment to local copy.

* Passing by reference can be simulated by using pointers

  ```c
    void incr (int *x) {
      *x = *x + 1;
    }
    
    int counter = 0;
    
    incr(&counter); /* pass pointer to counter */
    printf("%d", counter); // prints 1
  ```

* No need to distinguish between functions and procedures: return type
  `void` indicates side-effects only.

#### C++

* Evaluation strategy is strict. The order in which actuals are
  evaluated is unspecified.

* Default mode is by-value (same semantics as C)
    
* Explicit by-reference parameters are also supported.
      
* Semantic intent such as read-only references can be indicated by
  adding appropriate qualifiers:
    
  ```c++
  // 'x' is passed by reference, but 'f' cannot modify it
  void f(const double& x);
  ```

#### Java

* Evaluation strategy is strict. The order in which actuals are
  evaluated is left to right.

* Parameters are passed by value.

* However, object types have *reference semantics*. 

* That is, if the type of an argument is a class or interface, it will
  evaluate to a reference pointing to a heap-allocated object.
  
* This reference is passed by value (similar to a C pointer).
  
* Consequence: 

  * Methods can modify objects through the passed reference.
  * Assignment to the formal only affects the local copy of the
    reference, not the actual (like a C pointer).
  * Confusingly, this is sometimes referred to as call-by-reference
    even though it is not (because the reference stored in the actual
    cannot be modified by assignment to the formal).
  * Sometimes, this semantics is referred to as call-by-sharing.

* For parameters with object types: `final` means that formal is
  read-only.


#### Scala

* Default is same as in Java (strict evaluation with
  call-by-value/call-by-sharing, evaluation order left to right).

* However, formals cannot be assigned (their declarations are `val`
  rather than `var`).

* By-name parameters are also supported (see above).

* By-need parameters can be simulated using by-name parameters and
  *lazy values*.

#### OCaml

* Evaluation strategy is strict.  The order in which actuals are
  evaluated is unspecified.

* Parameters are passed by value/sharing (as in Java/Scala).

* Lazy evaluation can be simulated using higher-order functions.


#### Ada

* Goal: separate semantic intent from implementation

* Parameter modes:

  * `in`: read-only in subroutine
  * `out`: write-only in subroutine
  * `in out`: read-write in subprogram

* Mode is independent of whether binding by value, by reference, or by
  copy-return
 
  * `in`: bind by value or reference
  * `out`: bind by reference or copy-return
  * `in out`: bind by reference or by value/copy-return

* Functions can only have `in` parameters. Otherwise, they must be
  declared as procedures.

#### Haskell

* Evaluation strategy is lazy.

* Default parameter passing mode is by-need.


### Variable Number of Parameters

Some languages allow functions with a variable number of parameters. 

Example (in C):

```c
printf("this is %d a format %d string", x, y);
```

* Within body of `printf`, need to locate as many actuals as
  placeholders in the format string

* Solution: place parameters on stack in *reverse* order

  ```
  | ...                      |
  ├──────────────────────────┤
  | return address           |
  ├──────────────────────────┤
  | actual 1 (format string) |
  ├──────────────────────────┤
  | ...                      |
  ├──────────────────────────┤
  | actual n-1               |
  ├──────────────────────────┤
  | actual n                 |
  ├──────────────────────────┤
  | ...                      |
  ```
  
### Passing Subroutines as Parameters

C and C++ allow parameters which are pointers to subroutines:

```c
 void (*pf) (int);
 // pf is a pointer to a function that takes
 // an int argument and returns void
    
 typedef void (*PROC)(int);
 // type abbreviation clarifies syntax
    
 void do_it (int d) { ... }

 void use_it (PROC p) { ... }

 PROC ptr = &do_it;

 use_it(ptr);
 use_it(&do_it);
```

Question: Are there any implementation challenges for this kind of
subroutine call?

Not really. This feature can be implemented in the same way as a usual
subroutine call: in particular the *reference environment* remains the
same for the point where the subroutine is called and the point where
it was defined. Here, the reference environment refers to the binding
(i.e. mapping) of variables that are in the scope of the function
definition to the actual values that these variables hold at run-time
(respectively the memory locations that store these values).

The situation is more complex for languages where subroutines can
*escape* the environment where they are created. A subroutine escapes
the environment where it is created if it can somehow be called in a
different environment where the variables that where in scope at the
point of its definition are bound to different memory
locations/values. This can happen if the language allows nested
subroutines to be returned by the surrounding subroutine or passed to
other subroutines.

Consider the following Scala program:

```scala
def outer(i: Int, f: () => Unit): Unit = {
  def inner(): Unit = println(i)
    
  if (i > 1) f()
  else outer(2, inner)
}

def foo(): Unit = ()

outer(1, foo)
```

Note that the type `() => Unit` of the parameter `f` in function `outer`
indicates that `f` itself is a function that takes no arguments and
returns a value of type `Unit`. An example of such a function is `foo`
which on the last line is passed to `outer` as argument for the
function parameter `f`. Another example is the nested function `inner`
inside of `outer` which is passed to the parameter `f` of `outer` in
the recursive call in the *else* branch of the conditional.

The reference environment of function `inner` includes the binding of the
variable `i`, which is used in the print statement inside of the body
of `inner`.

Note that when this program executes, we first execute the call `outer(1,
foo)` which will take the *else* branch of the conditional inside `outer`
because the condition `i > 1` is false for `i=1`. In the recursive
call, however, we take the *then* branch because now `i=2`. The *then*
branch then calls the passed function `f`, which is `inner` from the
previous call to `outer`. 

What does this call to `inner` print? That is, does `i` in the body of `inner`
now use the binding in the original reference environment where `inner`
was defined, i.e. `i=1`, or does it use the binding in the environment
where `inner` is called, i.e. `i=2`?

There are two possible semantics:

* *Deep Binding*:

  * When a subroutine is passed as argument to another subroutine, a
    *closure* must be created and passed in place of the subroutine.

  * A closure is a pair of a subroutine together with its reference
    environment.

  * When a subroutine is called through a closure, the reference
    environment from when the closure was created is restored as part
    of the calling sequence.

  * With deep binding, the above program prints `1` because in the
    reference environment where the closure for `inner` was created when
    it was passed to the recursive call of `outer`, the variable `i` was
    bound to `1`.

* *Shallow Binding*:

  * When a subroutine is called, it uses the current reference
    environment at the call site.

  * With shallow binding, the program prints `2` because at the point
    where `inner` is called via `f`, the variable `i` is now bound to `2`.

Passing a nested subroutine to another subroutine is just one example
how a subroutine can escape the reference environment of its
definition site. Another way this can happen is if subroutines are
allowed to return nested subroutines. 

Consider the following variant of the example above:

 
```Scala
def outer(i: Int): Int => Unit = {
  def inner(j: Int): Unit = println(i + j)
  
  inner
}

var i = 2

outer(1)(2)
```

Note the return type `Int => Unit` of the function `outer` indicates
that `outer` returns a function that takes an `Int` as argument and
produces a return value of type `Unit`. The last line in the body of
`outer` determines the return value of `outer` which is the function
`inner` (note that we are not calling `inner` here but merely
returning the function `inner` itself to the caller of `outer`).

The call `outer(1)` in the last line will thus evaluate to the
function `inner` defined inside of `outer` which is then immediately
called with value `2`.

With static scoping and deep binding, the occurrence of `i` inside of
`inner` will always refer to the parameter `i` of `outer` which is
bound to `1` when `inner` is returned. Hence, this program will print
`3` in this case.

On the other hand, if the language uses dynamic scoping and shallow
binding, the `i` inside of `inner` will always refer to the current
`i` in the reference environment where `inner` is called. When
we call `inner` returned by `outer(1)` the `i` in the reference
environment is the global variable `i` whose value is `2`. Thus, in
this case the program prints `4`.

In general, static scoping demands the use of deep binding for nested
subroutines. Hence, Scala uses deep binding semantics. Shallow binding
is typically the default in languages with dynamic scoping that
support nested subroutines.
    

#### First-class functions: implementation issues

Functional programming languages treat functions as first-class values
(i.e. they can be passed to and returned by other functions).  When
using deep binding, this entails that the current activation record
must be allocated on the heap when a closure is created.

* Environment of function definition must be preserved until the point
  of call: activation record cannot be reclaimed if it creates
  functions.

* Functional languages therefore require more complex run-time
  management.

* Higher-order functions: functions that take (other) functions
  as arguments and/or return functions
  
  * very powerful 
  
  * but complex to implement efficiently
  
  * imperative languages (traditionally) restrict their use.
  
