![cljs](img/cljs.png)

> __If you just wanna try something__, check out the [interactive tutorial](http://chimeces.com/cljs-browser-repl/).

Hello, this is my attempt at a very concise guide to ClojureScript's syntax!
ClojureScript is a Lisp dialect for front-end web development.  It compiles to
JavaScript for use in the browser.

ClojureScript is fundamentally different from JavaScript and other
compile-to-JS languages like Dart, CoffeeScript, and TypeScript.  It uses a
more powerful yet simpler syntax.  There are other differences not related
to syntax, such as default immutability to combat the "new spaghetti code"
that is mutatable stateful objects, and sane state management allowing language-level data-binding.

I believe that ClojureScript's largest barrier to entry for beginners is
probably the foreign nature of its syntax.  I hope to explain it as plainly and
succinctly as possible with this guide.

> __Also__, check out [Parinfer] if you want a simpler way to manage parentheses in ClojureScript.

[Parinfer]:http://shaunlebron.github.io/parinfer/

## Syntax

There is __literal data__:

```clj
; number
1.23

; string
"foo"

; keyword (like strings, but used as map keys)
:foo

; vector (array)
[:bar 3.14 "hello"]

; map (associative array)
{:msg "hello" :pi 3.14 :primes [2 3 5 7 11 13]}

; set (distinct elements)
#{:bar 3.14 "hello"}
```

And there is __symbolic data__:

```clj
; symbol (represents a named value)
foo

; list (represents a "call")
(foo :bar 3.14)
```

#### Evaluation

ClojureScript can evaluate data to create a new "value" from it.

1. Literal data evaluates to itself, of course:

    ```clj
    1.23                 ; => 1.23
    "foo"                ; => "foo"
    [:bar 3.14 "hello"]  ; => [:bar 3.14 "hello"]
    ```

1. A __symbol__ evaluates to the value bound to it:

    ```clj
    foo                  ; => 3
    ```

1. A __list__ evaluates to the return value of a "call".

    ```clj
    (+ 1 2 3)            ; => 6
    (= 1 2)              ; => false
    (if true "y" "n")    ; => "y"
    ```

#### Calls

If the first element of a list is a __function__, then the rest of the elements
are evaluated and passed to it ([prefix notation](http://en.wikipedia.org/wiki/Polish_notation)).

```clj
; String concatenate function
(str "Hello " "World")  ; => "Hello World"

; Arithmetic functions
(= a b)     ; equality (true or false)
(+ a b)     ; sum
(- a b)     ; difference
(* a b c)   ; product
(< a b c)   ; true if a < b < c

; Evaluation Steps
(+ k (* 2 4))   ; assume k evalutes to 3
(+ 3 (* 2 4))   ; (* 2 4) evaluates to 8
(+ 3 8)         ; (+ 3 8) evalutes to 11
11
```

If the first element of a list is one of the language's few __special forms__,
then the rest of the elements are passed to it unevaluated.  (There are only [22
special forms](https://clojure.org/reference/special_forms).)

```clj
(if (= a b c)   ; <-- determines if a=b=c
    (foo 1)     ; <-- only evaluated if true
    (bar 2)     ; <-- only evaluated if false
    )

; define k as 3
(def k 3)       ; <-- notice that k is not evaluated here
                ;     (def needs the symbol k, not its value)

; make a greeting function
(fn [username]              ; <-- expected parameters vector
  (str "Hello " username))

; oops, give the function a name
(def greet (fn [username]
  (str "Hello " username)))

(greet "Bob")   ; => "Hello Bob"
```

If the first element of a list is a __macro__, then the rest of the elements
are passed to it unevaluated, but the resulting value of the call is evaluated.
Let's illustrate that difference with the following diagram:

![calls](img/calls.png)

This difference in evaluation allows macros to act as code-generating
functions.  For example, the `defn` macro expands to `def` and `fn`, as we used
separately in a previous example:

```clj
; create a named function using the defn macro
(defn greet [username]
  (str "Hello " username))

; the definition for the defn macro (over-simplified)
(defmacro defn [name args body]
  `(def ~name (fn ~args ~body)))
```

__App developers rarely need to create their own macros__, but it is an
indispensible tool for the library developer to give app developers the full
flexibility of the language.

#### Simple substitutions

There are a few [macro characters](http://clojure.org/reader#The%20Reader--Macro%20characters) that help make the language succinct
by performing simple substitutions (not full macros):

```clj
; short-hand for creating a simple function:
; #(...) => (fn [args] (...))

#(* 3 %)         ; => (fn [x] (* 3 x))

#(* 3 (+ %1 %2)) ; => (fn [x y] (* 3 (+ x y)))
```

## That's it for Syntax

You need to know more than syntax to be proficient in a language, of course.
But you should now know enough to be comfortable looking around at examples and
reasoning about how data is being evaluated and passed around:

```clj
; printing to the javascript console
(js/console.log "Hello World!")

; creating local bindings (constants)
(let [a (+ 1 2)
      b (* 2 3)]
  (js/console.log "The value of a is" a)
  (js/console.log "The value of b is" b))

; generate a sequence of numbers
(range 4) ; => (0 1 2 3)

; generate first four multiples of 3
(map #(* % 3) (range 4))           ;=> (0 3 6 9)

; count elements in a sequence
(count "Bob")     ; => 3
(count [4 5 2 3]) ; => 4

; select three letter names from a list
(def names ["Bob" "David" "Sue"])
(filter #(= (count %) 3) names) ; => ("Bob" "Sue")
```

(Don't worry about getting lost in parentheses.  All modern text editors will
highlight the corresponding ones and will indent your code automatically for
readability, as is standard in every other language.)

## ClojureScript vs JSX in HTML templating

See [Jumping from HTML to ClojureScript](https://github.com/shaunlebron/jumping-from-html-to-clojurescript) to see how ClojureScript's syntax solves the verbosity/flexibility problems faced in the JS community by [JSX].

[Jumping from HTML to ClojureScript]:https://github.com/shaunlebron/jumping-from-html-to-clojurescript
[JSX]:https://facebook.github.io/react/docs/jsx-in-depth.html

## A complete reference

The syntax section of the [ClojureScript API reference] is a comprehensive look at
the possible syntax forms and even shows the source code for how each are read/parsed.

[syntax section]:https://github.com/cljsinfo/cljs-api-docs/blob/catalog/INDEX.md#syntax
[ClojureScript API reference]:http://cljs.github.io/api

## Useful Resources

Here are the resources and steps that I took while learning ClojureScript.
(Most resources covering Clojure also apply to ClojureScript, since they
share a significant subset with each other.)

1. Reading through the [ClojureScript Wiki](https://github.com/clojure/clojurescript/wiki)
1. Reading the book [ClojureScript Up and Running](http://synrc.com/publications/cat/Functional%20Languages/Clojure/Oreilly.ClojureScript.Up.and.Running.Oct.2012.pdf)
1. Reading the book [Clojure Programming](http://bit.ly/clojurebook)
1. Doing [ClojureScript Koans](http://clojurescriptkoans.com)
1. Reading [Clojure Style Guide](https://github.com/bbatsov/clojure-style-guide)
1. Reading [Clojure Programming By Example](http://en.wikibooks.org/wiki/Clojure_Programming/By_Example)
1. Reading [Clojure Functional Programming](http://clojure.org/functional_programming)
1. Thumbing through [Clojure Core API](http://clojure.github.io/clojure/clojure.core-api.html)
1. Reading [ClojureScript - Differences from Clojure - Host Interop](https://github.com/clojure/clojurescript/wiki/Differences-from-Clojure#wiki-host-interop) for accessing javascript properties like `(.-Infinity js/window)` and functions like `(.sqrt js/Math 25)`.
1. Reading [JavaScript to ClojureScript synonyms](http://kanaka.github.io/clojurescript/web/synonym.html)
1. Experimenting in `lein repl` for Clojure REPL.
1. Experimenting in <http://clojurescript.net/> for ClojureScript REPL with a browser context.
1. Reading docstrings of functions I encounter with `(doc <funcname>)` in REPL.
1. [Miscellaneous ClojureScript things to know](https://github.com/shaunlebron/ClojureSheet#clojurescript-stuff)

