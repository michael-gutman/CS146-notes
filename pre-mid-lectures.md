CS146 Lecture 2 January 9, 2018 \# Reasoning about side-effects For pure
functional programming- substitution model Can the substitution model be
adapted? - "state of the world" is an extra input and output at each
step - Each reduction step transforms the program and also the state of
the world

How do we model "the state of the world"? - Modelling the entire world
is an impossible task - Can't even write something down without changing
that state - Simple cases - list of definitions - More complex cases -
Memory Model (RAM) - 32-bit RAM (Random Access Memory) - Memory
Addresses with 32bits of content each - Will use later

Modelling Output
================

-   Simplest kind of side-effect
-   "State of the world" is the sequence of chars that have been printed
    to the screen
-   Each step may or may not add to the sequence

Note: every string is just a sequence of characters (string-&gt;list)

Substitution Model:
===================

Each step is a version of the program by applying one reduction step to
the version before it (Pi\_0 -&gt; ... -&gt; Pi\_n)

Now also: w\_0 -&gt; w\_1 -&gt; w\_2 -&gt; ... w\_n - each w\_i is a
version of the output sequence - each w\_i is a prefix of w\_(i+1)
(can't "unprint" chars)

Combined: (Pi\_0, w\_0) -&gt; (Pi\_1, w\_1) -&gt; ... -&gt; (Pi\_n,
w\_n)

But some program reductions create definitions, e.g. local - Defined
values will eventually change

Better to seperate out the sequence of defns, D. (Pi\_0, D\_0, w\_0)
-&gt; (Pi\_1, D\_1, w\_1) -&gt; ... -&gt; (Pi\_n, D\_n, w\_n) D\_0, w\_0
empty

If Pi\_0 = (define id exp) ... - Reduce exp according to the usual
CS145/146 rules - May cause chars to be sent to w - exp now reduced to a
val - remove (define id val) from Pi and add to D

If Pi\_0 = exp ... - Reduce exp according to the usual CS145/146 rules -
May cause chars to be sent to w - exp now reduced to a val - Remove from
Pi - chars that make up value added to w

When Pi is empty - done

D, w - State - That which changes, other than the program itself. - w -
Relatively harmless, programs don't generally ask 'what's on the
screen?'. Changes to w don't affect the running of the program. - D -
Not a problem yet because variables are not yet changing (adding new
definitions not really side-effects)

Affecting w
===========

-   (display x) - outputs the value of x, no line break
-   (newline) - line break
-   (printf "The answer is \~a.\n" x) -&gt; (x = 20) The answer is 20
-   Formatted print
-   The value of x replaces \~a
-   \n = newline char (as a Racket char: \#\newline)

But then, what do display, newline, and printf return? - Special value
\#&lt; void &gt; - Not displayed in DrRacket - For fns that essentially
return nothing - Also the result of evaluating (void) - Fns that return
void are often called statements or commands (where imperative
programming gets its name)

Recall: map
===========

``` {.scheme}
(map f (list l_1 ... l_n))
```

produces

``` {.scheme}
(list (f l_1) ... (f l_n))
```

-   What if f is a statement
-   Needed for side-effects
-   Produces void
-   Then `Scheme   (map f (list l_1 ... l_n))` produces
    `Scheme   (list #<void> ... #<void>)`
-   Not useful

Now Consider: for-Each
======================

``` {.scheme}
(for-each f (list l_1 ... l_n))
```

performs

``` {.scheme}
(f l_1)
.
.
.
(f l_n)
```

and produces void

Example:

``` {.scheme}
(define (print-with-spaces lst)
  (for-each (lambda (x) (printf "~a " x)) lst))
```

Writing for-each:

``` {.scheme}
(define (for-each f lst)
  (cond [(empty? lst) (void)]
        [else (f (first lst))
              (for-each f (rest lst))]))
```

Makes use of the 'implicit begin' in the answer of cond

Using if

``` {.scheme}
(define (for-each f lst)
  (if (empty? lst)
      (void)
      (begin (f (first lst)) (for-each f (rest lst)))))
```

Doing nothing in one case of an if condition is common enough that there
is a specialized form:

``` {.scheme}
(define (for-each f lst)
  (unless (empty? lst)
  (f (first lst)) (for-each f (rest lst))))
```

-   (unless ...) evaluates body exprs if test is false
-   Similarly (when ...) evaluates body exprs if test is true

Reasoning about output, cont.
=============================

-   Before we had output,

1.  Order of operations didn't matter (assuming
    no crashes/non-termination)
2.  All non-terminating programs could be considered equivalent
    (not meaningful)

-   Now,

1.  Order of evaluation may affect order of output
2.  Non-terminating programs can do interesting things, e.g. print the
    digits of Pi

Semantic model should include the possibility of non-terminating
programs, meaning: - What the program would produce "in the limit" - O
(set of all possible values of w) would include finite and infinite
sequences of characters

What if you want to "save" the output?
======================================

``` {.scheme}
(with-output-to-file
  "myoutput.txt" (lambda () (printf "Test \n")))
```

-   "myoutput.txt" -&gt; name of file
-   "thunk" -&gt; fn with no args
-   with-output-to-file invokes the thunk and saves the output in the
    file

Better: - don't decide what to do with the output, let the user decide -
Linux shell - Output redirection

Why do we need output?
======================

-   Racket has a REPL (Read-Eval-Print Loop)
-   Can just call fns and see the result
-   Many languages
-   Compile-Link-Execute cycle
-   Program is translated by a compiler to native machine code and then
    executed from the command line
-   Only see output if the program prints it
-   Eg
    `C #include<stdio.h> int main(void){   printf("Hello world!\n")   return 0; }`

A use in Racket - Tracing programs

``` {.scheme}
(define (fact n)
  (printf "fact applied to argument ~a\n" n)
  (if (zero? n) 1 (* n (fact (sub1 n)))))
```

-   Can aid in debugging

CS146 Lecture 3 January 11, 2018

Modelling input
===============

-   Infinite sequence consisting of all chars the user will ever press
-   Model now (Pi, D, w, i)
-   Accepting an input char = removing a char from i

Problem:\
- The sequence may depend on the output - users decide what to press in
response to what is displayed on the screen - A more realistic model
might not assume that all the input is available at once

Alternative: A request for input yields a fn consuming one or more chars
and producing the next program Pi, with the input characters substituted
for the read request

Example: (read-line) -&gt; \\(line)line\
(user types abc) -&gt; "abc"

Entire program - reduces to a big nesting of input request fns, one fn
per "prompt" - Supply user input for each prompt - yields final result

Input in Racket
===============

(read-line) - produces a string of all chars pressed until the first
newline (string does not contain then newline)

Example:

    (string->list (read-line))

type Test -&gt;

``` {.scheme}
(list #\T #\e \s \t)
```

Example: Read in a list of lines

``` {.scheme}
(define (read-input)
  (define nl (read-line))
  (cond [(eof-object? nl) empty]
        [else (cons nl (read-input))]))
```

more primitive input:\
read-char - extracts one char from the input sequence peek-char -
examines the next character in the sequence, without removing it

``` {.scheme}
(define (my-read-line)
  (define (mrl-h acc)
    (define ch (read-char))
      (cond [(or (eof-object? ch) (char=? ch #\newline))
              (list->string (reverse acc))]
            [else (mrl-h (cons ch acc))]))
  (mrl-h empty))
```

less primitive input:\
read - consumes from input (and produces) an S-expression (no matter how
many chars or line it occupies)

``` {.scheme}
(define (repl)
  (define exp (read))
  (cond [(eof-object? exp) (void)]
        [else (display (interp (parse exp)))
              (newline)
              (repl)]))
(repl)
```

(have to fill in interp and parse)

Lets write our own read. Process typically happens in 2 steps. 1.
Tokenization\
Convert the sequence of raw characters to a sequence of tokens
(meaningful "words") eg. (, ), id, number\
- id - starts with a letter\
- number - starts with a digit - Key observation - peeking at the next
char tells us what kind of token we are getting, and what to look for to
complete the token

`Scheme   (struct token (type value))` Type: 'lp, 'rp 'id 'num\
"value" of the token (numeric value, name, etc)\
Helpers: \`\`\`Scheme (define (token-leftpar? x) (symbol=? (token-type
x) 'lp)) (define (token-rightpar? x) (symbol=? (token-type x) 'rp))

; read-id: -&gt; (listof char) (define (read-id) (define nc (peek-char))
(if (or (char-alphabetic? nc) (char-numeric? nc)) (cons (read-char)
(read-id)) empty))

; read-number: -&gt; (listof char) (define (read-number) (define nc
(peek-char)) (if (char-numeric? nc) (cons (read-char) (read-number))
empty)) `Main tokenizer:`Scheme ; read-token: -&gt; token (define
(read-token) (define fc (read-char)) (cond \[(char-whitespace? fc)
(read-token)\]\[(char=? fc \#( ) (token 'lp fc)\] \[(char=? fc \#) )
(token 'rp fc)\]\[(char-alphabetic? fc) (token 'id (list-&gt;symbol
(cons fc (read-id))))\] \[(char-numeric? fc) (token 'id (list-&gt;number
(cons fc (read-number))))\]\[else (error "lexical error")\])) \`\`\`

Note: list-&gt;symbol, list-&gt;number don't exist but easy to build

2.  Parsing

-   Are the tokens arranged in to a sequence that has the structure of
    an S-expression?
-   If so, produce the S-exp

Helper:

\`\`\` ; read-list: -&gt; (listof sexp) (define (read-list) ; assumes
left par has already been read (define tk (read-token)) (cond
\[(token-rightpar? tk) empty\]\[(token-leftpar? tk) (cons (read-list)
(read-list))\] \[else (cons (token-value tk) (read-list))\]))

; my-read: -&gt; sexp (define (my-read) (define tk (read-token)) (if
(token-leftpar? tk) (read-list) (token-value tk))) \`\`\`

Exercises - add support for symbols (symbols can start with numbers, as
long as they contain at least one letter) - handle arbitrary kinds of
brackets - (a \[b c {d e} f\]) - they have to match - generalizes
HTML/XML/JSON parser

CS146 Lecture 4 January 16, 2018

Input Cont'd
============

What have we lost by accepting input? - referential transparency - The
same expression has the same value whenever it is evaluated

Example:\
(f 4) always produces the same value

``` {.scheme}
(let ((z (f 4))) body)
```

-   every z in body can be replaced by (f 4) and vice versa. "equals can
    be substituted for equals"

#### Not true anymore!

-   (read) doesnt always produce the same value!
-   Makes it harder to reason about programs
-   Simple algebraic manipulation is no longer possible

Intro to C
==========

### Expressions, Statements, Blocks, Functions, Program

##### Expressions:

-   `1 + 2` - infix operator - precedence
-   `f(7) - function call`
-   `printf("%d\n", 5)`

Operator Precedence: - Usual mathematical conventions

Function calls: - `3 + f(x, y, z)` - `printf("%d/n", 5)` - function call
- substitute 5 for %d - %d = display as a decimal number - produces the
\# of chars printed

##### Statements:

-   `printf("%d\n", 5);` - **;** makes any expr. a statement (command)
-   Value produced by the statement is ignored
-   Expr is evaluated only for its side-effects\
    `1 + 2;` - legal, but useless
-   `return 0;` Produce the value 0 as the result of this function
    (control returns immediately to the caller)
-   `;` Empty statement (do nothing)
-   Other statement forms to come

##### Blocks:

-   Groups of statements, treated as one statement\
    `{stmt1 stmt2 ... stmtn}`

##### Functions:

``` {.c}
int f(int x, int y) {
  printf("x=%d, y=%d\n", x, y);
  return x + y;
}
```

Think:

``` {.scheme}
; f : Num Num -> num
(define (f x y)
  (printf "x=~a, y=~a\n", x y)
  (+ x y))
```

function call: - f(4, 3) - expr (produces 7) - f(4, 3); - statement

Note: Contracts (type signatures) are required and enforced

##### Program

-   Sequence of Functions
-   Starting point: function main

    ``` {.c}
    int main() {
    ...
    ...
    }
    ```

Example:

``` {.c}
int main() {
    f(4, 3);
    return 0;
}
int f(int x, int y) {...}
```

Doesn't compile. Why?\
- main doesn't know what f is.

C enforces declaration before use: - Cant use a function/vaiable/etc
until you tell C aout it

Solution 1 - Put f first

``` {.c}
int f(int x, int y) {...}
int main() {
    f(4, 3);
    return 0;
}
```

-   Ok, but more than necessary

    ``` {.c}
    int f(int x, int y) {...}
    ```

-   is both a decleration (tells C about the function)\
    and a definition (completely constructs the function)

C only requires declaration before use\
So, you can have

``` {.c}
int f(int x, int y);
```

-   function prototype or header
-   decl. only

``` {.c}
int f(int x, int y);

int main() {
    f(4, 3);
    return 0;
}

int f(int x, int y) {
  ...
}
```

Still doesn't compile. - What is printf? - no declaration for printf

``` {.c}
int printf(...);
int f(int x, int y);
int main() {
  ...
}
int f(int x, int y) {
  ...
}
```

Rather than declare every standard library function header before you
use it, C provides "header files":

``` {.c}
#include<stdio.h>

int f(int x, int y);
int main(){
  ...
}
int f(int x, int y) {
  ...
}
```

include - not part of the C language
====================================

-   Directive to the C preprocessor (which runs before the compiler)
-   Like macro expansion in Racket. (cond, if)
-   `#include <file.h>` "drop the contents of file.h right here"

stdio.h - contains declarations for printf and other I/O functions -
located in a "standard place"\
eg /usr/include

Now the compiler is satisified.

Still technically incomplete - Where is the code that implements printf?
- printf was written once, compiled once, and put in a "standard place"\
eg /usr/lib - Code for printf must be combined with this code -
"Linking" - A linker takes care of this - Linker runs automatically -
"Knows" to link in the code for printf

If you write your own modules, you need to tell the linker about them
(later)

``` {.c}
int main() {
  ...
  return 0;
}
```

-   Who does this 0 go to? The OS
-   Tells if the program succeeded or failed
-   echo \$?

##### Variables

``` {.c}
int f(int x, int y) {
  int z = x + y
  int w = 2;
  return z/w;
}
```

##### Input

``` {.c}
#include<stdio.h>
int main(){
  char c = getchar();
  return c;
}
```

-   Characters are just small ints

Read in a number

``` {.c}
#include<stdio.h>
int getIntHelper(int acc) {
  char c = getchar();
  if (c >= '0' && c <= '9') return getIntHelper(acc * 10 + c - '0');
  else return acc;
}
int getInt() {
  return getIntHelper(0);
}
```

CS146 lecture 5 January 18 2018

### Recall

``` {.c}
#include<stdio.h>
int getIntHelper(int acc) {
  char c = getchar();
  if (c >= '0' && c <= '9')
    return getIntHelper(acc * 10 + c - '0');
  else return acc;
} 
int getInt() {
  return getIntHelper(0);
}
```

OR

``` {.c}
int getIntHelper(int acc) {
  char c = getChar();
  return (c >= '0' && c <= '9') ? getIntHelper(acc * 10 + c - '0') : acc;
}
```

Dangling else problem

    if (test1)
      if (test2)
        stmt2
    else
      stmt3

else goes with the nearest unmatched if \#\#\# Conditional Operator ?:
(also called the ternary operator)\
- if-else is a statement - ?: creates an expression - `a ? b : c` has
value b if a is true and c if a is false

Note: No built in boolean type in C - 0 means false - Non-zero (often 1)
means true\
eg `if (0) {...}` - false condition - bool type, constants true, false :
stdbool.h - 'Lets let bool mean int', 'let false = 0', 'let true = 1'

### Characters

-   Restricted form of integer
-   Everything in memory is just a number anyway, why pretend its not
-   int - varies, but typically 32 bits (\~ 4x10\^9 distinct values)
-   char - 8 bits (256 distinct values)
-   '0' - the character 0, numerically 48
-   '9' = 57 (48+0)
-   `char c = '0'` identical to `char c = 48`

Everything in memory is a number, so every char must have a numeric code
that represents it.

##### (some useful) ASCII Code:

-   '\n' = 10
-   '\[space\]' = 32
-   '0' - '9' = 48 - 57
-   'A' - 'Z' = 65 - 90
-   'a' - 'z' = 91 - 122

Convert a digit char c to its numeric value: `c - '0'`\
Convert a \# (0-9) to ASCII: `c + '0'`

### A second look at getchar();

The prototype for getchar is actually `int getchar()`\
Why *int* if it's supposed to produce a char?\
What if there are no chars (EOF)? If getchar() returned a char, there
would be no way to indicate EOF. (Every possible returned value
represents a valid character)\
If there are no chars (EOF), getchar produces an int that can't possibly
be a char (0..255)\
The Constant EOF dontes the value getchar prodces on EOF (often, EOF =
-1)

##### Next Question:

getInt burns a character after reading an int. - does C have a f'n like
Racket's peek-char?

No, but it has ungetc - stuffs a char back into the input stream

``` {.c}
int peekchar() {
  int c = getchar();
  return c == EOF ? EOF : ungetc(c, stdin);
}
```

Improved getInt - doesn't burn a char

``` {.c}
#include <ctype.h> //character predicates
int getIntHelper (int acc) {
  int c = peekchar();
  return isdigit(c) ? getIntHelper(10 * acc + getchar() - '0');
}
```

The consant struggle that every programmer faces between abstraction and
efficiency.\
More efficient - doesn't call getchar twice per char:

``` {.c}
int getIntHelper(int acc) {
  int c = getchar();
  return isdigit(c) ? getIntHelper(10 * acc + 'c' - '0')
                    : (ungetc(c, stdin), acc);
}
```

`a, b` evaluate a, then evaluate b, the result is the value of b\
like (begin a b)

What if there is whitespace before we reach the int?

``` {.c}
void skipws() {
  int c = getchar();
  if (isspace(c)) {
    skipws();
  }
  else ungetc(c, stdin);
}
```

(doesn't take into account EOF)

A void function - returns nothing - Cannot be used in an expression\
eg `void x = skipws(); //ILLEGAL` - There are no void variables - Only
good for side effects

Returning from void functions: - Reach the end, or, - return; (no expr)

So get int becomes

``` {.c}
int getInt() {
  skipws();
  return getIntHelper(0);
}
```

### Mutation

Basic mutation: `set!`

    (define x 3)
    (set! x 4)
    x
    ; -> 4

-   Produces (void)
-   Changes D
-   x must have been previously defined

Example:

    (lookup 'Brad)
    ; -> false
    (add 'Brad 36484)
    (lookup 'Brad)
    ; -> 36484

-   Not possible in pure Racket
-   Same expression can't produce different results

In Impure racket:

    (define address-book empty)

-   Global variable
-   Visible throughout the program

<!-- -->

    (define (add name number)
      (set! address-book (cons (list name number) address-book)))

CS146 lecture 6 January 23 2018 \#\#\# Recall

    (lookup 'Brad)
    ; -> false
    (add 'Brad 36484)
    (lookup 'Brad)
    ; -> 36484

    (define address-book empty)
    (define (add name number)
      (set! address-book (cons (list name number) address-book)))

### Global Data

-   Good for defining constants to be used repeatedly
-   *But*, not good with mutation
-   Any part of a program could change a global variable
-   Therefore, a change affects the entire program
-   Hidden dependencies between different parts of the program
-   Can't tell just by looking at one function that it affects another
-   ie, lookup and add are dependent but don't call each other
-   Makes programs harder to reason about

#### Application: Memoization

-   Caching
-   Saving the result of a computation to avoid repeating it
-   Memoization
-   Maintaining a table of cached values

##### Consider

    (define (fib n)
      (cond [(= n 0) 0]
            [(= n 1) 1]
            [else (+ (fib (- n 1))
                      (fib (- n 2)))]))

Inefficient because recursive calls repeated - (fib 98) called twice -
(fib 97) called 3x - etc

Avoid repetition - Keep an association of pairs (n, Fn)

    (define fib-table empty)

    (define (memo-fib n)
      (define result assoc n fib-table)
      (cond [result => second]
            [else
              (define fib-n
                (cond [(<= n 1) n]
                      [else (+ (memo-fib (- n 1))
                               (memo-fib (- n 2)))]))
              (set! fib-table (cons (list n fib-n)
                                    fib-table))
              fib-n]))

##### Notes

-   assoc - built-in fn for association list lookup
-   (assoc x lst) returns the pair (list x y) from lst, or \#f
-   any value can be used as a test
-   false is false; anything else is true
-   (cond \[x =&gt; f\] - if x passes, i.e. is not false, produces (f x)
-   Calls to (fib n) happen only once

fib-table - global variable - Access from outside can make memo-fib
wrong - Can we hide it?

    (define memo-fib
      (local [(define fib-table empty)
              (define (memo-fib n) ...)]
            memo-fib))

Doesn't quite work for address book - two fns need access to the global
var - but it can be done --- \#\#\# Mutation in C Operator = performs
mutation ("assignment operator")

``` {.c}
int main() {
  int x = 3; printf("%d\n", x);
  x = 4; printf("%d\n", x);
}
```

prints:\
3\
4

##### Note: = is an operator

-   x = y is an expression
-   it has a value as well as an effect
-   its value is the value assigned
-   x = 4 = sets x to 5 and has value 4

eg

``` {.c}
int main() {
  int x = 3
  printf("%d\", x);
  printf("%d\n", x = 4);
}
```

prints:\
3\
4 - Advantages - almost none - Disadvantages - many

eg - x = y = z = 7 - sets all x, y, z to 7

``` {.c}
int main() {
  int x = 5;
  if (x = 4) {
    printf("x is 4\n");
  }
  x = 0;
  if (x = 0) {
    printf("x is 0\n");
  }
}
```

-   x = 4 sets x to 4, has value 4, non-zero
-   always prints x is 4
-   x = 0 sets x to 0 again, value is 0 -&gt; false
-   never prints x is 0
-   Easy to confuse assignment with equality check
-   if (x == 4) ...
-   if (x == 0) ...
-   Usually best to use assignment only as a statement

Can leave vars uninitialized and assign later

``` {.c}
int x; //uninitialized
x = 4;
```

-   Not a good idea
-   Do only with a very good reason

``` {.c}
int x;
if (x == 0) {
  ...
}
```

-   Will it run or not?
-   The value of an uninitialized var is undefined
-   x most likely holds whatever the memory happened to hold before

### Global Variables

``` {.c}
in c = 0; //global variable

int f() { //returns 0, then 1, then 2, ...
  int d = c;
  c = c + 1;
  return d;
}

int main() {
  printf("%d\n", f());
  printf("%d\n", f());
  printf("%d\n", f());
}
```

0\
1\
2

##### Careful:

``` {.c}
printf("%d\n%d\n%d\n", f(), f(), f());
```

-   Could produce 0 1 2, or 2 1 0 or others\
-   Order of evaluation is unspecified
-   Never mutate and access the same variable in the same expression

As with the Racket fib table, can interfere with f by mutating c - Can
we protect c from access by fns other than f?

``` {.c}
int f() {
  static int c = 0; //A global variable that only f can see
  int d = c;
  c = c + 1;
  return d;
}
```

### Repetition

``` {.c}
void sayHiNTimes(int n) {
  if (n > 0) {
    printf("Hi\n");
    sayHiNTimes(n-1);
  }
}
```

Tail recursion - is just repetition

CS146 Lecture 7 Jan 25 2018

### Recall

``` {.c}
void sayHiNTimes(int n) {
  if (n > 0) {
    printf("Hi\n");
    sayHiNTimes(n-1);
  }
}
```

-   Tail recursion is just repetition
-   Expressed more idiomatically as

``` {.c}
void sayHiNTimes(int n) {
  while (n > 0) {
    printf("Hi\n");
    n = n - 1
  }
}
```

-   a loop- the body of the loop is executed repeatedly as long as the
    condition remains true

##### In General

``` {.c}
void f(int c) {
  if (cont(c)) { //continuation condition
    body(c);
    f(update(c));
  }
}
```

becomes

``` {.c}
void f(int c) {
  while (cont(c)) {
    body(c);
    c = update(c);
  }
}
```

-   may not need to be in its own fn anymore, if used only once

### Accumulators

``` {.c}
int f(int c, int acc) {
  if (cont(c)) {
    body(c, acc);
    return f(update1(c), update2(c, acc));
  }
  return g(acc)
}
f(acc0)
```

becomes

``` {.c}
int acc = acc0
while (cont(c)) {
  body(c, acc);
  acc = update2(acc, c);
  c = update1(c);
}
acc = g(acc)
```

-   It is always a tail recursive computation that is turned into a loop
-   Much harder to do with a computation that is not tail recursive

##### Eg

``` {.c}
int getIntHelper(int acc) {
  char c = getchar();
  if (isdigit(c))
    return getIntHelper(10 * acc + c - '0');
  return acc;
}
int getInt() {
  return getIntHelper(0);
}
```

-   This is tail recursive, so this should be writeable as a loop

``` {.c}
int acc = 0;
char c = getchar();
while (isdigit(c)) {
  acc = 10 * acc + c - '0';
  c = getchar();
}
```

-   This version doesn't technically burn a character since 'c' is still
    accessible

#### Common patterns emerge

When writing these functions, a common pattern emerges

    (init variables)
    while (condition) {
      (body)
      (update variables)
    }

Alternative 'for'mat

    for (init ; condition ; update) {
      body
    }

Let's write getInt like This

``` {.c}
int acc = 0
char c;
for (c = getchar(); isdigit(c); c=getchar()) {
  acc = 10 * acc + c - '0';
}
```

or even:

``` {.c}
int acc = 0
for (char c = getchar(); isdigit(c); c = getchar()) {
  acc = 10 * acc + c - '0';
}
```

-   Even more succinct, and nice to create and init together
-   But changes the meaning of the program slightly
-   Second version has limited the scope of c to loop only, c is gone
    after the loop terminates

or even

``` {.c}
int acc = 0;
for (char c = getchar(); isdigit(c); acc = 10 * acc + c - '0', c = getchar());
```

-   comma operator
-   semicolon indicating empty loop body
-   Not good Usually, hard to read, not point
-   could be useful if body depends on two counters

##### Peekchar version

``` {.c}
int acc = 0;
for (char c = peekchar(); isdigit(c); acc = 10 * acc + getchar() - '0', c = peekchar());
```

-   When using peekchar, have to use getchar again after

or

``` {.c}
int acc = 0;
for (char c = peekchar(), isdigit(c); c = (getchar(), peekchar())) {
  acc = 10 * acc + c - '0';
}
```

-   Most explicit, showing you need to use getChar() and peekChar()
    again

#### Updating Counters

-   c = c + 1;
-   c = c - 2;
-   c = 10 \* c;
-   c = c/2;
-   c = c + d;

Specialized syntax: - c += 1; - c -= 2; - c \*= 10; - c /= 2; - c += d;

Increment/decrement by 1: - ++c; - --c;

These are expressions - they have a value as well as an effect - ++c;
increments c and produces the value of c - --c; similarly - Which value?
The old one or the new one?

``` {.c}
int i = 1;
printf("%d\n", ++1);
```

-   prints 2, the new value
-   postfix versions, i++, i--
-   increment and decrement - but produce the old value
-   implies old value must be remembered
-   if you use increment/decrement operators, prefer the prefix versions

### More on global Data

-   global variables
-   int i = 0;
-   avoid where possible, creates hidden dependencies
-   Global constants still useful, can force a varable to remain
    constant in cs
-   const int passingGrade = 50;
-   cannot be mutated \*\*\* \#\#\# Intermediate Mutation (Racket) What
    if we want to work with multiple address books?

        (define work '(("Manager" 12345)
                   ("director" 23456)))
        (define home '())

    When adding an entry, want to indicate which address book to add it
    to \`\`\` (define (add-entry abook name number) (set! abook (cons
    (list name number) abook)))

(add-entry home 'Neighbour 34567) home -&gt; '()

    - Nothing changed, doesn't work
    - Not clear how to make it work
    - substitution model doesn't explain behaviour
    - substitution doesn't even make sense

(add-entry home "neighbour" 34567) ; subst '() for home in body -&gt;
(set! '() (cons (list "neighbour" 34567) '()))


    To make this work...  
    Recall from CS145 - simulation of structs using lambda  
    Do the same thing to create a struct with one field
    - Called a box
    - two operations
      - get the value in the box
      - set the value to a new value

(define (make-box v) (lambda (msg) (cond \[(equal? msg 'get) v\])))

(define (get b) (b 'get))

(define b1 (make-box 7)) (get b1) -&gt; (define b1 (lambda (msg) (cond
\[(equal? msg 'get) 7\]))) (get b1) -&gt; (get (lamda (msg) (cond ...)))
-&gt; (lambda (msg) (cond ....) 'get) -&gt; (cond \[(equal? 'get 'get)
7\]) -&gt; 7

    Also need to support set - introduce a local copy of v so that we can mutate

(define (make-box v) (define val v) (lambda (msg) (cond \[(equal? msg
'get) val\])))

(define (get b) (b 'get))

(define b1 (make-box 7)) (get b1) -&gt; (define val\_1 7) (define b1
(lambda (msg) (cond \[(equal? msg 'get) val\_1\]))) (get b1) -&gt;
....... -&gt; 7

    Adding set
    - requires an extra param
    - achieve by having the box return a fn

(define (make-box v) (define val v) (define (msg) (cond \[(equal? msg
'get) val\]\[(equal? msg 'set) (lambda (newv) (set! val newv))\])))

(define (get b) (b 'get)) (define (set b v) ((b 'set) v))

(define b1 (make-box 7)) (set b1 4) -&gt; (define val\_1 7) (define b1
(lambda (msg) .... val\_1 ...)) (set b1 4) -&gt; (define val\_1 7) ...
((b1 'set) 4) -&gt; (define val\_1 7) ... ((lambda (newv) (set! val\_1
newv)) 4) -&gt; (define val\_1 7) ... (set! val\_1 4) -&gt; (define
val\_1 4) ... (void)

    How does this fix the addr book?
    Before

(add home ... ...) -&gt; (add '() ... ...) -&gt; (set! '() ...) cant
update home

    Now

(define home (make-box '()))

(define (add abook name num) (set abook (cons (list name num) (get
abook))))

(add home ... ...)


    CS146 Lecture 7 Jan 30 2018

    ### Recall: Boxes/addressbook

    Boxes are built in to Racket,
    Syntax:

exp = (box exp) &lt;- make-bx | (unbox exp) &lt;- get | (set-box! exp
exp) &lt;- set

    Redefining addressbook using built-in Boxes

(define home (box '((Mom 45678) (sis 56789)))) (define work (box '((Boss
67890))))

(define (add abook name phone) (set-box! abook (cons '(name phone)
(unbox abook))))

    #### Semantics

(box v) -&gt; (define \_u v)

    Convention (Not Racket) - When we write an underscore before a var name it means the var's value is not looked up during expression evaluation, unless (unbox \__) is called on it.

(unbox \_n) -&gt; find (define \_n v) -&gt; v (set-box \_n v) -&gt; find
(define *n *) -&gt; (define \_n v) -&gt; (void)

    eg

(define box1 (box 4)) (unbox box1) (set-box! box1 true) (unbox box1)

-&gt; (define \_u1 4) (define box1 \_u1) (unbox box1) (set-box! box1
true) (unbox box1)

-&gt; (define \_u1 4) (define box1 \_u1) (unbox \_u1) (set-box! box1
true) (unbox box1)

-&gt; (define \_u1 4) (define box1 \_u1) 4 (set-box! box1 true) (unbox
box1)

-&gt; (define \_u1 4) (define box1 \_u1) 4 (set-box! \_u1 true) (unbox
box1)

-&gt; (define \_u1 true) (define box1 \_u1) 4 (void) (unbox box1)

-&gt; (define \_u1 true) (define box1 \_u1) 4 (unbox \_u1)

-&gt; (define \_u1 true) (define box1 \_u1) 4 true \`\`\` - a bit messy
- subst. model doesn't work well with mutation - This problem we
encounter is not an issue of Racket or C but an issue of imperative
programming

#### The same problem in C

Suppose we want to write a fn:

``` {.c}
void inc(int x) {
  x = x + 1;
}
int main() {
  int x = 1;
  inc(x);
  printf("%d\n", x);
}
```

-   Want: 2
-   Get: 1
-   Why? Imagine a substitution
-   There is no association between the passed x and the x in the fn
-   It is not x itself, but only the value is passed

#### Racket soln

-   Put the variable in a box
-   C doesn't have boxes, or lambdas

##### What is the C equivalent?

-   One field structures

### Structs in C

``` {.c}
struct Posn {
  int x;
  int y;
};
```

-   Ending semicolon *needed*
-   Could create Posns while defining

    ``` {.c}
    struct Posn {
      int x;
      int y;
    } p1, p2, p3;
    ```

-   Almost literally nobody does this

So:

``` {.c}
int main() {
  struct Posn p;
  p.x = 3;
  p.y = 4;
  printf("p=(%d, %d)\n", p.x, p.y)
}
```

OR

``` {.c}
int main() {
  struct Posn p = {3, 4};
  printf("p=(%d, %d)\n", p.x, p.y)
}
```

Now let's try something with structures

``` {.c}
void swap(struct Posn p) {
  int temp = p.x;
  p.x = p.y;
  p.y = temp;
}
int main() {
  struct Posn p = {3, 4};
  swap(p)
  printf("p=(%d, %d)\n", p.x, p.y)
}
```

-   Want: p=(4, 3)
-   Get: p=(3, 4) - still doesn't work
-   The value of the struct is still passed, swap has no access to the
    original variable

#### The heart of the problem:

-   C (and also Racket) passes paramteres by a mechanism called
    call-by-value
-   The fn operates on a **copy** of the argument, not the argument
    itself
-   The Racket substn model naturally implements call-by-value

<!-- -->

    (define x 3)
    (f x) -> (f 3) - 3 is the value of x, not x itself

In C

``` {.c}
void inc(int x) {
  ++x;
}
```

-   x really does get mutated
-   but it is a copy of x, not the original from the caller
-   original remains the same

Similarly:

``` {.c}
void swap(struct Posn p) {
  int temp = p.x;
  p.x = p.y;
  p.y = temp;
}
```

-   The entire struct is copied into the function
-   Original does not change

#### So there is something special about boxes

-   The Racket substitution model does not explain them
-   What we know about C so far cannot emulate them
-   boxes are not equal to the value they hold
-   but they tell you (know?) how to find the value (unbox)

#### Finding a value - where is it located?

-   In memory (RAM)
-   Every value in RAM has an address
-   Given the address, can "find" the value
-   So addresses could function as boxes
-   instead of passing a value to a fn, pass an address

``` {.c}
int x = 1;
inc(&x);
printf("%d\n", x);
```

-   & -&gt; "address of" operator
-   Passes x's address, not its value

``` {.c}
void inc(int x) { // Wrong now, didn't get int, got address
  x = x + 1;
}
```

-   Maybe `void inc(address x) {...}`? No.
-   Want a guarantee that the address points to an int
-   Need more info, what type of data is stored at that address
-   Need to say "x is the adress of an int"

``` {.c}
void inc(int *p) { // p is called a pointer to an int
  p = p + 1 // Also wrong
}
```

-   Legal, Will compile, Wrong
-   Don't want to add 1 to the address
-   Want to add 1 to the value stored at the address

#### Finally

``` {.c}
void inc(int *p) {

  *p = *p + 1
}
```

-   \* = dereference operator (unbox)
-   fetch the value stored at the addr (sort of)
-   LHS of assign (\*x = exp) -&gt; Store the value of exp at address x

##### Alternatives

``` {.c}
void inc(int *p) {
  *p += 1;
}
```

``` {.c}
void inc(int *p) {
  ++*p;
}
```

but

``` {.c}
void inc(int *p) {
  *p++; //wrong
}
```

-   Postfix before prefix in C
-   ++ happens first, then \*
-   means \*(p++) - increase the address/pointer and fetch the value at
    the *original* address (and do nothing with it)
-   No change to the original variable

#### Now Consider

``` {.c}
void swap(struct Posn p) {
  int temp = p.x;
  p.x = p.y;
  p.y = temp;
}
```

Fix by passing a pointer

``` {.c}
void swap(Struct posn *p){
  int temp = *p.x;
  *p.x = *p.y;
  *p.y = temp;
  // WRONG again, post-fix before pre-fix
}
```

-   \*p.x = \*p.y means \*(p.x) = \*(p.y) = nonsense

Needs parens

``` {.c}
void swap(Struct posn *p){
  int temp = (*p).x;
  (*p).x = (*p).y;
  (*p).y = temp;
}
```

-   (\*p).x is common enough that it has its own notation
-   p-&gt;x means (\*p).x

``` {.c}
void swap(Struct posn *p){
  int temp = p->x;
  p->x = p->y;
  p->y = temp;
}
```

CS146 Lecture 9 Feb 1 2018

#### Recall: Pointers in C

More sophisticated input: scanf - scanf("%d", x); (wrong) - reads x as a
decimal int - skips leading whitespace - scanf is a fn, cant modify x

Instead: `scanf("%d", &x)` - `scanf("%d %d", &x, &y);` - space
in-between skips any amount of whitespace, including zero - scanf
returns the number of objects actually read - lots of options - scanf is
very complicated

### Advanced Mutation

-   Mutating Structures and Lists

#### In Scheme:

-   can mutate parts of a cons with set-car! and set-cdr!

#### In Racket:

-   cons fields are **immutable**
-   cannot be mutated

#### For mutable pairs:

-   Racket provides mcons
-   mset-car!, mset-cdr!

#### For structs:

-   `(struct posn (x y) #:mutable)`

##### Semantics

-   (posn v1 v2) cannot be a simple value if it is mutable
-   structs were values, just like numbers
-   cannot change value of 3 to 4
-   has to behave more like a box

##### How?

-   is a struct automatically boxed?
-   is a struct a box?

#### Some experimentation

    (struct posn (x y) #:mutable #:transparent)

    (define (mutate-posn p)
      (set-posn-x! p (+ 1 (posn-x p))))

    (define (mutate-posn2 p)
      (set! p (posn (+ 1 (posn-x p)) (posn-y p))))

    (define p (posn 1 2))
    (define q (posn 1 2))

    (mutate-posn p)
    (mutate-posn2 q)

    p -> (posn 2 2)
    q -> (posn 1 2)

-   Posns themselves do not sit inside of boxes
-   Posns DO act as boxes for their fields

So, a struct is not automatically boxed, but it does box its contents.\
So we can write:

    (posn v1 v2)

as

    (define _u1 v1)
    (define _u2 v2)
    (posn _u1 _u2)

-   Recall - no expansion of \_u
-   We know from C this really means posn has 2 pointers inside of it
-   (posn-x p) -&gt; find the defn for \_u1, fetch the value
-   (set-posn-x! p v) -&gt; find (define \_u1 ..), replace .. with v

<!-- -->

    (define p (posn 3 4))
    (set-posn-x! p 5)

    -> (define _u1 3)
       (define _u2 4)
       (define p (posn _u1 _u2))
       (set-posn-x! p5)

    -> ...
       (set-posn-x! (posn _u1 _u2) 5)

    -> (define _u1 5)
       (define _u2 4)
       (define p (posn _u1 _u2))
       (void)

-   generalize to any mutable struct, mcons

#### Consider

    (define lst1 (cons (box 1) empty))
    (define lst2 (cons 2 lst1))
    (define lst3 (cons 3 lst1))

    (set-box! (first (rest lst2)) 4)

    (unbox (first (rest lst3)))

-   By the CS145 model - should produce `1`
-   But in fact you get `4`

Why: - lst2 = `(cons 2 (cons (box 1) empty))` - lst3 =
`(cons 2 (cons (box 1) empty))` - But the two boxes are actually the
same object - lst2 and lst3 share the same tail

Could never tell this before, need mutation to see

    (define _u1 1)
    ls2 = (cons 2 (cons _u1 empty))
    ls3 = (cons 3 (cons _u1 empty))

-   old rules: get the wrong answer 1
-   our rules - boxes rewritten as a separate define with deferred
    lookup

#### Rethinking define

    (define x 3) (set! x 7) x
    -> (define x 7) (void) x
    -> 7

-   Not just a replacement of all x's with 3
-   otherwise we would have gotten a 3

##### x is not just a value

-   something we can mutate
-   an entity we can access
-   x must denote a *location*, and the location contains the value

### So:

-   We don't just have one lookup 𝛿: var -&gt; value
-   we have *two* lookups:
-   var -&gt; location
-   location -&gt; value
-   set! changes the location -&gt; value map
-   But nothing changes the var -&gt; location map
-   Similarly, set-box! changes the location -&gt; val mapping
-   (define ...) creates a location, fills it with a value

In C:
-----

``` {.c}
int main() {
  int x = 1;
  int *y = &x;
  int *z = y;
  *y = 2;
  *z = 3;
  printf("%d %d %d\n", x, *y, *z);
}
```

#### Output: `3 3 3`

##### Why?

-   y is initialized to x's address
-   y points to the location of x
-   z is initialized equal to y
-   Thus, z is also equal to x's address
-   \*y = 2 -&gt; store 2 at x's location -&gt; `x == *y == *z == 2`
-   \*z = 3 -&gt; store 3 at x's location -&gt; `x == *y == *z == 3`
-   x, *y, *z, are three different names for same data

##### Called Aliasing

-   Accessing the same data by different names

Can be subtle:

``` {.c}
void f(int *x, int *y) {
  *y == *x + 1;
  if (*y == *x) {
    printf("How could this ever print?\n");
  }
}

int main() {
  int z = 1;
  f(&z, &z);
  printf("%d\n, z");
}
```

-   Makes \*x and \*y aliases
-   Will print
-   Makes programs very difficult to understand

CS146 Lecture 10 Feb 6 2018

### Memory and Vectors

#### Recall:

-   Memory - set of numbered "slots"
-   address and contents
-   each 'box': 8 bits, (one byte)
-   usually treated in groups of 4-byte words

#### Primitive data structure: the array

-   a "slice" of memory
-   sequence of consecutive memory locations

#### Racket (also Scheme): The Vectors

-   used much like a traditional array
-   unlike traditional arrays - can hold items of any size (unlimited
    integers, strings, whatever)

### Vectors in Racket

    (define x (vector 'blue true "you"))

-   3-item vector

<!-- -->

    name  address   contents
      x     0       .......
            1       .......
            2       .......
            3       .......
          ....
          length chosen by the program
                  <--------> width - any value?

    (define y (make-vector 100))

-   make a vector of length 100

        -> (vector 0 0 0 0 ....)

<!-- -->

    (define z (build-vector 100 sqr))
    -> 0 1 4 .... 99^2

    (define y (make-vector 100 5))
    -> 5 5 5 5 5 ... x 100

#### Working with Vectors

    (define y (make-vector 100 5))
    (vector-ref y 7) -> 5
    (vector-set! y 7 42)
    (vector-ref y 7) -> 42

-   access and mutate items by index (starting at 0)

##### Main advantage of vectors over lists - constant time

-   `vector-ref`, `vector-set!` run in O(1) time

##### Disadvantages

-   Size is fixed
-   difficult to add or remove items
-   `vector-set!` tends to force an imperative style

Making our own build vector

    (define (my-build-vector n f)
      (define res (make-vector n))
      (define (mbv-h i)
        (cond [(= i n) res]
              [else (vector-set! res i (f i))
                    (mbv-h (+ i 1))]))
      (mbv-h 0))

-   Very imperative
-   There isn't a good way to do it not imperatively

##### Vectors work well with imperative-style algorithms

-   `for`, `for/vector` "macros" facilitate this

eg

    (define (my-build-vector n f)
      (define res (make-vector n))  
      (for ([i n])
        (vector-set! res i (f i)))
      res)

or, just:

    (define (my-build-vector n f)
      (for/vector ([i n]) (f i)))

sum the elements of a vector

    (define (sum-vector vec)
      (define (sv-h i acc)
        (cond [(= i ((vector-length vec)) acc)]
              [else (sv-h (+ 1 i)
                          (+ acc (vector-ref vec i)))]))
      (sv-h 0 0))

-   very loop-like

So:

    (define (sum-vector vec)
      (define sum 0)
      (for ([i (vector-length vec)]
           (set! sum (+ sum (vector-ref vec i)))))
      sum)

-   Looks very similar to what it would look like in C (minus
    the brackets)
-   Not pure functional - uses mutation
-   But it **looks** pure functional
-   use of mutation confined to the internals of sum-vector
-   can't be detected outside the fn (no printing, not clear)
-   so outsiders could consider it pure functional

##### Provides a strategy for keeping the problems with mutation under control - hide it behind a pure functional interface

#### Recall: Memory vs Racket Vectors

-   Memory only holds fixed-size data
-   unlimited integers? strings?
-   How does this work?

##### Recall (an analogy):

    (define (mutate-posn p)
      (set-posn-x! p (+ 1 (posn-x p))))

    (define p (posn 3 5))
    (mutate-posn p)
    (posn-x p) -> 4

vs

``` {.c}
void mutate(struct Posn p) {
  p.x += 1;
}
int main(){
  struct Posn p = {3,5};
  mutate(p);
  printf("%d\n", p.x); -> 3
}
```

-   Rackets structs and C structs are different
-   Passing a Racket struct to mutate-posn works, but not in C
-   The struct is copied but changes to the field still persist
-   The fields of a Racket struct are 'boxed'
-   They're pointers

Similarly, the items in a Racket vector are *addresses* that point to
the actual contents (which can be of any size)

Similarly, the fields of a cons are pointers

Also, since Racket is dynamically typed, the values `1`, `'blue`, `true`
must somehow include type information (more later).

### "Vectors" in C: Arrays

-   An array is a sequence of consecutive memory locations

``` {.c}
int main() {
  int grades[10]; //An array of 10 ints
  for (int i = 0; i < 10; ++i) {
    scanf("%d", &grades[i]);
  }
  int acc;
  for (int i = 0; i < 10; i++){
    acc += grades[i];
  }
  printf("%d\n", acc/10);
}
```

-   `a[i]` = access the ith element of array a
-   `int grades[10]` - valid entries are `grades[0], ... , grades[9]`

What happens if you go out of bounds? - undefined behaviour - in
principle, anything is possible - certain behaviours are more likely -
could be nothing happens, could be your program works, could be you
change other variables, could be your program crashes (more likely) -
expect a crash, or maybe not

Will it stop you? No. - C says "You're an adult. You know how long the
array is. If you want to walk off the end of a cliff, go ahead." -
program may or may not crash - if not, data may be corrupted; no way to
detect it.

Can give the bound implicitly:

``` {.c}
int main() {
  int grades[] = {0, 0, 0, 0, 0}
  printf("%zd\n", sizeof(grades)/sizeof(int));
}
```

-   sizeof = the amount of memory occupied - evaluated by the compiler
-   `sizeof(grades)` should be 20 bytes
-   `sizeof(int)` = 4 bytes
-   data type = size\_t, %zd

#### Functions on arrays:

``` {.c}
int sum(int arr[], int size) {
  int res = 0;
  for (int i = 0; i < size; ++i) res += arr[i];
  return res;
}
```

-   sizeof doesn't work here
-   Can't know how long an array is, have to tell it

Passing arrays by value -&gt; - Copying the entire array - Expensive

C will not do this.

``` {.c}
int main() {
  int myArray[100];
  ...
  int total = sum(myArray, 100);
  ...
}
```

-   How is this not a copy?
-   'Most confusing rule in all of C':
-   The name of an array is shorthand for a pointer ot its first element

CS146 Lecture 11 Feb 7 2018

#### Recall:

-   The name of an array is shorthand for a poointer to its
    first element.
-   `myArray` is shorthand for `&myArray[0]`

But sum is expecting an array not a ptr: `int sum(int a[], int size);`\
Why not `int sum(int *a, int size)`;? - `int *arr` and `int arr[]` are
identical *in parameter declarations*

Let's take a look at sum again:

``` {.c}
int sum(int *arr, int size) {
  int res = 0;
  for (int i = 0; i < size; ++i) res += arr[i];
  return res;
}
```

-   If its expecting a pointer, can we still do `arr[i]`?
-   Yes

### Pointer Arithmetic

-   Let t be a type
-   `t arr[10];`
-   `sizeof(arr) = 10*sizeof(t)`
-   `arr` is shorthand for `&arr[0]`
-   `*arr` is equivalent to `arr[0]`

What expression produces a pointer to `arr[1]`? - `arr + 1` is shorthand
for `&arr[1]` - `arr + 2` is shorthand for `&arr[2]` - etc - Numerically
`arr + n` produces the address equal to `arr + n*sizeof(t)`

If `arr + k` is shorthand for `&arr[k]` then `*(arr[k])` means `arr[k]`

Therefore we could do:

``` {.c}
int sum (int *arr, int size) {
  int res = 0;
  for (int i = 0; i < size; ++i) res += *(arr + i);
  return res;
}
```

-   In fact, `a[k]` is just shorthand for `(a+k)`
-   Note: addition is commutative, so:
-   `*(a + k) = *(k + a) = k[a]`

Now:

``` {.c}
int sum(int *arr, int size) {
  int res = 0;
  for (int *cur = arr; cur < arr + size; ++cur) res += *cur;
  return res;
}
```

-   point at the array, and walk the pointer through it

##### Any pointer can be thought of as pointing to the beginning of an array

-   Same syntax for accessing items through an array as through a
    pointer
-   So: arrays and pointers are the same thing?
-   **NO!**

``` {.c}
int f(int arr[]) {
  printf("%zd\n", sizeof(arr));
}

int main() {
  int myArray[10];
  printf("%zd\n", sizeof(myArray));
  f(myArray);
}
```

Prints:\
40 -size of the array\
8 -size of a ptr to the array

##### Compiler:

`myArray[i]` - fetch myArray location in env - add i\*sizeof(int) -
fetch this address from store

`arr[i]` - fetch arr loc in env - fetch my array addr from store - add
i\*sizeof(int) to addr - fetch item value from store

------------------------------------------------------------------------

We saw that a Racket struct `(struct posn x y)` is like a C struct whose
fields are pointers. How can we achieve this in C?

Well, we can do:

``` {.c}
struct Posn {
  int *x;
  int *y
};

int main() {
  struct Posn p;
  //What are p.x and p.y pointing at?
  *p.x = 3;
  *p.y = 4;
}
```

-   Crashes (most likely)
-   p.x and p.y are uninitialized pointers
-   They point at arbitrary locations, dictated by whatever value they
    happen to hold

So (posn 3 4) must also reserve memory for x and y to point at, to hold
the 3 and 4. - Need to do the same in C

``` {.c}
#include <stdlib.h>
struct Posn makePosn(int x, int y) {
  struct Posn p;
  p.x = malloc(sizeof(int));
  p.y = malloc(sizeof(int));
  *p.x = x;
  *p.y = y;
  return p;
}
```

-   `malloc(n)` = request n bytes of memory

``` {.c}
struct Posn p = makePosn(3, 40); //OK
```

CS146 Lecture 12 Feb 8 2018 \#\#\#\# Recall:

``` {.c}
Struct Posn makePosn(int x, int y) {
  struct Posn p;
  p.x = malloc(sizeof(int))
  p.y = malloc(sizeof(int))
  *p.x = x;
  *p.y = y;
  return p;
}
```

### Memory Layout (applies to C and Racket)

### RAM:

-   Your program's binary code
-   Your Program's Data
-   Static Area
-   Heap
-   Stack

#### Static Area

-   where global/static vars are stored
-   lifetime: entire program

(skipping over the heap for now)

#### Stack

##### What is a stack?

-   An ADT with LIFO semantics
-   LIFO = Last in, first out
-   Can only Remove the most recently inserted item
-   Operations:
-   Push - Add an item to the Stack
-   Top - What is the most recently inserted item?
-   Pop - Remove the most recently inserted item
-   Empty? - Is the stack empty?

##### Racket lists are stacks

-   Push = cons
-   Top = first/car
-   Pop = rest/cdr

##### Program Stack

-   Stores local variables

``` {.c}
int fact(int n) {
  int rec =  0;
  if (n == 0) return 1;
  rec = fact(n-1);
  return n*rec;
}

int main() {
  int f = fact(3);
}
```

-   Each call to fact gets added on top of the stack
-   The last function call is the first to return
-   The stack then shrinks as functions return

-   Each function *call* gets a *stack frame*
-   also return address - where to go when the function returns
-   each call to a function gets its own version of the local variables
-   When a function returns - it's stack frame is popped
-   all local vars in that frame are released
    -   The variables are **not** typically erased
    -   "top of stack pointer" is moved to top of next frame
    -   will be overwritten the next time a frame is pushed onto the
        stack
-   Lifetime of a stack data - duration of the function call/block/scope

##### So what if you have data that must persist after a fn returns?

e.g, What's wrong with:

``` {.c}
struct Posn makePosn(int x, int y) {
  struct Posn p;
  int a = x, b = y;
  p.x = &a;
  p.y = &b;
  return p;
}
```

-   a, b on the stack, so as soon as p returns, p is pointing into the
    wilderness

##### So: malloc requests memory from the heap

-   Pool of memory from which you can explicitly request "chunks"
-   Lifetime is arbitrary

-   When make posn returns
-   p (incl p.x, p.y) popped off the stack
-   3 and 4 on the heap
-   p from makePosn copied back to main's frame
-   main has access to 3 and 4 on the heap
-   these outlive makePosn

##### What is the lifetime of heap-allocated data?

-   arbitrarily long
-   If heap allocated data never goes away, program will eventually run
    out of memory
-   even if most of the data in memory is no longer in use

##### Racket soln

-   a run-time process detects memory that is no longer accessible (has
    nothing pointing at it)

<!-- -->

    (define (f x)
      (define p (posn 3 4))
      ...
      (+ x 1))

-   Posn p not needed after fn returns
-   Automatically reclaims memory - "garbage collection"

##### C soln: Heap memory is freed when you free it

``` {.c}
int *p = malloc(...);
...
free(p); //release p's memory back to the heap
```

-   Failing to free all allocated memory - "Memory leak"
-   Programs that leak will eventually fail (if they run long enough)

Now:

``` {.c}
int *p = malloc(sizeof(int));
free(p);
*p = 7; //Will this crash?
```

-   probably doesn't crash
-   free(p) doesn't change p
-   p still points to that memory
-   storing something there probably still works
-   But p is not pointing at a valid location
-   that location may be assigned to another ptr by another malloc call
-   Called a "dangling pointer" (BAD)

Better: after free(p), assign p to point to a guaranteed-invalid
location:

``` {.c}
int *p = malloc(...)
free(p);
p = NULL;
```

##### NULL

-   Not really part of the C language
-   defined constant equal to 0
-   could equally well say `p = 0;`

Derefrencing NULL - undefined behaviour - program *may* crash

``` {.c}
int main() {
  int *p = NULL;
  *p;
}
```

Consider again

``` {.c}
int *f() {
  int x = 4;
  return &x;
}

int g() {
  int y = 5;
  return y;
}

int main() {
  int *p = f();
  g();
  printf("%d\n", *p);
}
```

-   p pts to dead Memory
-   when f returns, x no longer a valid location
-   a *dangling pointer*
-   Program may not crash
-   May behave badly
-   g occupies f's old stack frame
-   y now occupies x's old slot
-   \*p is now 5 (still a dangling ptr)

#### Lesson: Never return a ptr to a local variable

-   If you want to return a ptr - it should point to static, heap, or
    non-local stack data

CS146 Lecture 13 Feb 13 2018

### Recall:

-   Never return a ptr to a local variable
-   If you want to return a ptr - it should point to static, heap, or
    non-local stack data
-   Valid ways of returning pointers:

``` {.c}
int *pickOne (int *x int *y) {
  return ......... ? x : y;
}

struct Posn *getMeAPtr() {
  struct Posn *p = malloc(sizeof(struct Posn));
  return p;
}

int z = 5;
int $f(){return &z;}
```

#### Use the heap for:

1.  For data that should outlive the scope that creates it
2.  For data whose size is not known at compile time
3.  For large local arrays

#### Examples

-   1 as above (posn)
-   2

``` {.c}
int numSlotsNeeded;
scanf("%d", &numSlotsNeeded);
int *p = malloc(numSlotsNeeded * sizeof(int))
```

-   can access p\[0\],...,p\[numSlotsNeeded-1\];
-   dynamic array (heap-allocated)
-   ... free(p);
-   3
-   Programs typically have more heap memory available than stack memory

``` {.c}
int recursiveFunction(int n) {
  ..
  int tempArray[10000];
  ...
  recursveFunction(n-1);
}
```

-   don't be surprised if this crashes
-   eats up stack space for each recursive call

### C arrays mimic Racket vectors

-   Can we get the behaviour of a Racket list?
-   `(cons x y)` -&gt; produces a pair
-   Recall: Racket is dynamically typed: list items can have different
    types
-   C is statically typed -&gt; list itmes would ned to have the same
    type
-   (if not - headaches)
-   so - no real need for ptrs to data fields

``` {.c}
struct Node {
  int data;
  struct Node *next;
}

struct Node* cons(int data, struct Node* next) {
  struct Node* result = malloc(sizeof(struct Node));
  result->data = data;
  result->next = next;
  return result;
}

int main() {
  struct Node* lst = cons(1, cons(2, cons(3, 0)));
  ...
}
```

-   called a linked list

Processing a linked list:

-   (like Racket)

``` {.c}
int length (struct Node* lst) {
  if (!lst) return 0; //if (list==NULL)
  return 1 + length(lst->next);
}
```

-   With a loop

``` {.c}
int length (struct Node* lst) {
  int res = 0;
  for (struct Node* cur = lst; cur; cur = cur->next) {
    ++res;
  }
  return res;
}
```

#### Can we write map?

``` {.c}
int f(int n) { ... }

int main() {
  struct Node* lst = ...;
  struct Node* lst2 = map(f, lst);
}
```

-   The name of a function is shorthand for a pointer to its code

``` {.c}
struct Node* map(....., struct Node* lst) {}
```

-   So what goes in the `....`?
-   What type do we use for for f
-   ptr to a function so maybe: `int* f(int)` but this is wrong
-   post-fix before pre-fix
-   means fn returning a ptr, not a ptr to a function
-   So:

``` {.c}
struct Node* map(int (*f)(int), struct Node* lst) {
  if (!lst) return 0;
  return cons(f(list->data), map(f, lst->data));
}
```

-   Don't need to dereference f (\*f), C lets you get away with this

#### Freeing the list:

``` {.c}
int main() {
  struct Node* lst = cons(1, cons(2, cons(3, 0)));
  ...
  free(lst);
}
```

#### leaks

``` {.c}
int main() {
  ...
  for (struct Node* cur = lst; cur; cur = cur->next) {
    free(cur);
  }
  //BAD
}
```

-   `cur = cur->next` happens AFTER free(cur)
-   cur is dangling
-   need to grab next ptr before you free

``` {.c}
for (struct Node* cur = lst; cur;) {
  struct Node* temp = cur;
  cur = cur->next;
  free(temp);
}
```

##### OR

``` {.c}
void freeList(struct Node* lst) {
  if (!lst) {
    freeList(list->next);
    free(lst);
  }
}
```

### Application of Vectors

#### ADT Map/Dictionary (Mutable)

-   make-map: no params, pre: true
-   add:
-   params: map M, key K, value v
-   Pre: true
-   Produces no value
-   Post: if there exists a v1 st (k, v1) in M,\
    then M &lt;- M\\{(k, v1)} U {(k, v)}\
    else M &lt;- M U {(k, v)}
-   remove:
-   params Map m, Key k
-   Pre: true
-   produces no value
-   Post: if there exists v st (k, v) in M, M&lt;-M\\{(k, v)}\
    else M unchanged
-   search:
-   params Map m, Key k
-   Pre: true
-   value produced is v st (k, v) in M,\
    else something outside value domain

#### Implementation

-   Assume keys are integers
-   For simplicity - omit values
-   If we use an association list
-   Accessing an item takes time proportional to its position in the
    list (O(length L) worst case)
-   If we use a BST
-   same worst case running time
-   If we use a balanced BST (e.g. AVL tree)
-   O(logn) worst case time
-   Difficult implementation

If we use vectors instead - O(1) for any access\
But how big should the vector be?

CS146 Lecture 14 Feb 14 2018

### ADT Dictionary

-   Use a vector? O(1) retrieval time
-   Size of vector = maximum key?
-   Will waste space

#### Use a combination - Create a vector of association lists

-   Called a hash table

<!-- -->

    (define (create-hashtable size) (make-vector size empty))

-   To which association list should we add (k, v)?
-   Need to map k to a vector index
-   mapping called a hash function
-   for simplicity (albeit inefficient) - use the remainder of k by the
    length of the vector
-   for this idea to work well, the hash fn must distribute keys evenly
    over the indices

<!-- -->

    (define (ht-search table key)
      (define index (modulo key (vector-length table)))
      (define hashlist (vector-ref table index))
      (define lookup (assoc key hashlist))
      (if lookup (second lookup) false))

    (define (ht-add table key val)
      (define index (module key (vector-length tabke)))
      (define hashlist (vector-ref table index))
      (define lookup (assoc key hashlist))
      (if lookup
          (when (not (equal? (second lookup) val))
                (vector-set! table index
                  (cons (list key val)
                        (remove lookup hashlist))))
          (vector-set! table index (cons (list key val) hashlist))))

-   If keys are not numbers - need a hash fn that maps keys to numeric
    values
-   Not actually O(1), really more like O(n/k)

### ADTs in C

-   C doesn't have modules. C has files.

#### Implement ADT sequence in C

Operations: empty sequence - insert(s, i, e) - insert in e at index i in
s - Pre: 0 &lt;= i &lt;= size(s) - size(s) - \# of elements in s -
remove(s, i) - remove item from index i - Pre: 0 &lt;= i &lt;= size(s) -
1 - index(s, i) returns the ith element of s - Pre: 0 &lt;= size(s) - 1

WANT: No limits on size - sequence can grow as needed

##### Implementation Options

-   Linked List
-   Easy to grow
-   Slow index
-   Array
-   fast index
-   hard to grow

Going with arrays, and worrying about growing later

##### Approach: partially-filled heap-array

``` {.c}
Struct Sequence {
  int* theArray;
  int size; //How many items are in use?
  int cap; //How many items can I hold?
}
```

##### How to structure:

-   sequence.h

``` {.c}
struct Sequence {
  int size, cap;
  int* theArray;
}
struct Sequence emptySeq();
void add(struct Sequence* s, int i, int e);
int seqSize(struct Sequence* s, int i);
void remove(struct Sequence* s, int i);
int index(struct Sequence s, int i);
void freeSeq(struct Sequence* s);
```

-   sequence.c

``` {.c}
#include "sequence.h"
```

CS146 Lecture 15 Feb 15 2018 \#\#\# Recall: partially-filled heap-array
\#\#\#\#\# How to structure: - sequence.h

``` {.c}
struct Sequence {
  int size, cap;
  int* theArray;
}
struct Sequence emptySeq();
void add(struct Sequence* s, int i, int e);
int seqSize(struct Sequence* s, int i);
void remove(struct Sequence* s, int i);
int index(struct Sequence s, int i);
void freeSeq(struct Sequence* s);
```

-   sequence.c

``` {.c}
#include "sequence.h"
Struct Sequence emptySeq() {
  struct Sequence res;
  res.size = 0;
  res.cap = 10;
  res.theArray = malloc(10*sizeof(int));
  return res;
}

int seqSize(struct Sequence s) {return s.size;}

void add(struct Sequence* s, int i, int e) {
  for (int n = s->size; n > i; --n) {
    ++s->size;
    s->theArray[i] = e;
  }
}
remove(){} //exercise
int index(struct Sequence s, int i) {return s.theArray[i];}
freeSeq(){} //exercise
```

-   main.c

``` {.c}
#include "sequence.h"

int main() {
  struct Sequence s = emptySeq();
  add(s, 0, 4);
  add(s, 1, 7);
  ...
}
```

-   OK, but not immune to tampering/forgery

``` {.c}
s.size = 8; //tampering!

//forgery!
struct Sequence t;
t.size = 10;
t.cap = 20;
```

##### Can we stop this?

-   keep the fields of struct sequence hidden
-   declar, bit not define, the struct?

-   sequence.h

``` {.c}
struct Sequence;
struct Sequence emptySeq();
...
```

-   Sequence.c

``` {.c}
#include "sequence.h"
struct Sequence {
  int size, cap;
  int* theArray;
}
...
```

Then, - main.c

``` {.c}
int main() {
  struct Sequence s; //Nope
  ...
}
```

-   Doesn't compile
-   Compiler doesn't know enough about struct Sequence
-   Can't allocate space without knowing size

-   sequence.h

``` {.c}
struct SeqImpl;
typdef struct SeqImpl* Sequence;

Sequence emptySeq();
void add (Sequence s, int i, int e);
...
```

-   main.c

``` {.c}
#include "sequence.h"

int main() {
  Sequence s; //ptr - OK!
  ...
}
```

What happens when the array is full?

``` {.c}
void add(Sequence s, int i, int e) {
  if (s->size == s->cap) {
    //make the array bigger
    s->theArray = realloc(s->theArray, ...)
    ...
  }
}
```

-   realloc - increases a block of memory to a new size
-   if necessary, allocates a new, larger block and frees the old block
    (data copied over)

How big should we make it? - one bigger? - Must assume that each call to
realloc causes a copy - O(n)

If we have a sequence of adds (at the end, so no shuffling cost) - \# of
steps = n + (n + 1) + (n + 2) + (n + 3) + ... + (n + k) + ... - O(n\^2)
total cost, O(n) per add

What if instead, we double the size? - Each add still O(n) worst case

##### But - Amortized Analysis

-   places a bound on a dequence of operations, even if an indibidual
    operation may be expensive
-   If an array has a cap of n and is empty:
-   n inserts cost O(1) each
-   1 insert costs O(n) - cap now 2n
-   n - 1 inserts cost O(1) each
-   1 insert costs 'O(2n)' - cap now 4n
-   2n - 1 inserts cos O(1) each
-   1 insert costs O(4n)
-   Total cost: n + n + n - 1 + 2n + 2n - 1 + 4n = n - 2
-   Total inserts: 4n + 1
-   cost per insertion ~=~ 11/4 - O(1)

``` {.c}
void increaseCap(Sequence s) {
  if (s->size == s->cap) {
    s->theArray = realloc(s->theArray, 2*s->cap);
    s->cap = 2 * s->cap;
  }
}

void add(Sequence s, int i, int e) {
  increaseCap(s);
  ...
}
```

-   Helperfunction - increaseCap - main should not be calling this
-   How do we prevent it?
-   Leave it out of the header file

-   But
-   main.c

``` {.c}
#include "sequence.h"
void increaseCap(sequence s); //What if main declares its own header?

int main() {
  ...
}
```

-   if REALLY necessary:
-   sequence.c

``` {.c}
static void increaseCap(Sequence s) {...}
```

-   static means: only visible in **this** file
-   prevents othe files from having access even if the write their own header
    -------------------------------------------------------------------------

    Interpreting Mutation
    ---------------------

### Recall:

-   The deferred substitution interpreter in Haskell for Faux Racket:

``` {.haskell}
exp = number
    | (op exp exp)
    | (fun (id) exp)
    | (with ((id exp)) exp)
    | (exp exp)
    | id

op = + | *
```

Actual Haskell:

``` {.haskell}
data Op = Plus | Times

opTrans Plus = (+)
opTrans Times = (*)

data Ast = Number Integer
         | Bin Op Ast Ast
         | Fun String Ast
         | App Ast Ast
         | Var String
```

-   Where did (with...) go?
-   (with ((id exp1)) exp2) == ((fun (id) exp1) exp2)

``` {.haskell}
data Value = Numb Integer
           | Closure String Ast Env

type Env = [(String, Val)]

interp: Ast -> Env -> Val
interp (Number v) _ = Numb v
interp (Fun p b) e = Closure p b e
interp (Bin op x y) e = Numb (OpTrans op v w)
    where
      (Numb v) = interp x e
      (Numb w) = interp y e
```

-   TBC after the break

