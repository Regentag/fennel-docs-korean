# Getting Started with Fennel
 * [OK, so how do you do things?](#ok-so-how-do-you-do-things)
   * [Functions and lambdas](#functions-and-lambdas)
   * Locals and variables
   * Numbers and strings
   * Tables
   * Sequential Tables
   * Iteration
   * Looping
   * Conditionals
 * Back to tables just for a bit
 * Error handling
 * Variadic Functions
 * Strict global checking
 * Gotchas
 * Other stuff just works
 * Modules and multiple files
 * Relative require
   * Compile-time relative include
   * Requiring modules from modules other than `init.fnl`

A programming language is made up of **syntax** and **semantics**. The semantics of Fennel vary only in small ways from Lua (all noted below). The syntax of Fennel comes from the lisp family of languages. Lisps have syntax which is very uniform and predictable, which makes it easier to [write code that operates on code](https://stopa.io/post/265) as well as [structured editing](http://danmidwood.com/content/2014/11/21/animated-paredit.html).

If you know Lua and a lisp already, you'll feel right at home in Fennel. Even if not, Lua is one of the simplest programming languages in existence, so if you've programmed before you should be able to pick it up without too much trouble, especially if you've used another dynamic imperative language with closures. The [Lua reference manual](https://www.lua.org/manual/5.4/) is a fine place to look for details, but Fennel's own [Lua Primer](https://fennel-lang.org/lua-primer) is shorter and covers the highlights.

If you've already got some Lua example code and you just want to see how it would look in Fennel, you can learn a lot from putting it in [antifennel](https://fennel-lang.org/see).

## OK, so how do you do things?
### Functions and lambdas
Use `fn` to make functions. If you provide an optional name, the function will be bound to that name in local scope; otherwise it is simply an anonymous value.

> A brief note on naming: identifiers are typically lowercase separated by dashes (aka "kebab-case"). They may contain digits too, as long as they're not at the start. You can also use the question mark (typically for functions that return a true or false, ex., `at-max-velocity?`). Underscores (`_`) are often used to name a variable that we don't plan on using.

The argument list is provided in square brackets. The final value in the body is returned.

(If you've never used a lisp before, the main thing to note is that the function or macro being called goes inside the parens, not outside.)

```lisp
(fn print-and-add [a b c]
  (print a)
  (+ b c))
```

Functions can take an optional docstring in the form of a string that immediately follows the argument list. Under normal compilation, this is removed from the emitted Lua, but during development in the REPL the docstring and function usage can be viewed with the `,doc` command:

```lisp
(fn print-sep [sep ...]
  "Prints args as a string, delimited by sep"
  (print (table.concat [...] sep)))
,doc print-sep ; -> outputs:
;; (print-sep sep ...)
;;   Prints args as a string, delimited by sep
```

ike other lisps, Fennel uses semicolons for comments.

Functions defined with `fn` are fast; they have no runtime overhead compared to Lua. However, they also have no arity checking. (That is, calling a function with the wrong number of arguments does not cause an error.) For safer code you can use `lambda` which ensures you will get at least as many arguments as you define, unless you signify that one may be omitted by beginning its name with a `?`:

```lisp
(lambda print-calculation [x ?y z]
  (print (- x (* (or ?y 1) z))))

(print-calculation 5) ; -> error: Missing argument z
```

Note that the second argument `?y` is allowed to be `nil`, but `z` is not:

```lisp
(print-calculation 5 nil 3) ; -> 2
```

Like `fn`, lambdas accept an optional docstring after the argument list.
