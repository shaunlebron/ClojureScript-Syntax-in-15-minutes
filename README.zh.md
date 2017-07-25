![cljs](img/cljs.png)

> __如果你只是想试试__, 可以看看这个[交互式教程](http://chimeces.com/cljs-browser-repl/).

Hello, 我在尝试给 ClojureScript 语法写一份简明的教程！
ClojureScript 是一门适用于 Web 前端开发的 Lisp 方言, 它被编译到 JavaScript 在浏览器中使用.

ClojureScript 在根本上就和 JavaScript 和其他诸如 Dart, CoffeeScript 和 TypeScript 之类的编译到 JavaScript 的语言不一样.
它使用了更强大而简单的语法. 也有一些其他一些语法之外的区别,
比如说默认采用不可变性, 用来解决状态可变的对象造成"新的意面式代码",
同时提供正常的状态管理功能用来实现语言级别的数据绑定.

我相信对于新手来说 ClojureScript 最大的障碍在于他令人感到陌生的语法.
在这份教程里, 我希望能尽可能清晰而简明地说明它.

> __同时__, 如果你需要为 ClojureScript 找一个更简单的管理括号的方案, 请了解一下 [Parinfer].

[Parinfer]:http://shaunlebron.github.io/parinfer/

## 语法

这是 __literal data(字面量数据)__:

```clj
; number(数字)
1.23

; string(字符串)
"foo"

; keyword(关键字, 就像字符串, 但是用于 map 的键)
:foo

; vector(向量, 或者说数组, array)
[:bar 3.14 "hello"]

; map(关联数组, associative array)
{:msg "hello" :pi 3.14 :primes [2 3 5 7 11 13]}

; set(distinct elements, 元素唯一)
#{:bar 3.14 "hello"}
```

还有 __symbolic data(符号化数据)__:

```clj
; symbol(符号, 表示一个有名字的值)
foo

; list(链表, 表示一次"调用")
(foo :bar 3.14)
```

#### 求值

ClojureScript 可以对数据求值从而创建出新的"值".

1. 字面量数据求值之后得到自身, 显然地:

    ```clj
    1.23                 ; => 1.23
    "foo"                ; => "foo"
    [:bar 3.14 "hello"]  ; => [:bar 3.14 "hello"]
    ```

1. __符号__ 求值之后得到绑定在上面的数值:

    ```clj
    foo                  ; => 3
    ```

1. __list__ 求值之后得到"调用"的结果.

    ```clj
    (+ 1 2 3)            ; => 6
    (= 1 2)              ; => false
    (if true "y" "n")    ; => "y"
    ```

#### 调用

如果列表的第一个元素是一个 __函数__, 那么其余元素会被求值并传给它 ([prefix notation](http://en.wikipedia.org/wiki/Polish_notation)).

```clj
; 字符串合并函数
(str "Hello " "World")  ; => "Hello World"

; 算术函数
(= a b)     ; 等式(true 或者 false)
(+ a b)     ; 求和
(- a b)     ; 求差
(* a b c)   ; 求积
(< a b c)   ; true if a < b < c

; 求值步骤
(+ k (* 2 4))   ; 这里假设 k 求值得到 3
(+ 3 (* 2 4))   ; (* 2 4) 求值得到 8
(+ 3 8)         ; (+ 3 8) 求值得到 11
11
```

如果列表的第一个元素是一个 __特殊形式(Special Form)__, 那么其余元素传递给它时不做求值.
(总共有 [22 的特殊形式](https://clojure.org/reference/special_forms).)

```clj
(if (= a b c)   ; <-- 判断是否 a=b=c
    (foo 1)     ; <-- 只在 true 的时候求值
    (bar 2)     ; <-- 只在 false 的时候求值
    )

; 定义(define) k 为 3
(def k 3)       ; <-- 注意这个 k 不会被求值
                ;     (def 需要 k 这个符号, 而不是它的值)

; 创建一个打招呼的函数
(fn [username]              ; <-- 所期望的参数向量
  (str "Hello " username))

; 哦对了，给这个函数取个名字
(def greet (fn [username]
  (str "Hello " username)))

(greet "Bob")   ; => "Hello Bob"
```

如果列表的第一个元素是个 __Macro__, 那么其余元素传给它时不做求值,
但是调用之后的结果是经过求值的. 用下面这个示意图来说明它们的区别:

![calls](img/calls.png)

求值过程上的区别使得 Macro 可以作为生成代码的函数使用.
比如 `defn` 这个 Macro 展开成了 `def` 和 `fn`，前面例子中我们是分开使用的:

```clj
; 用 defn 这个 Macro 创建一个具名函数
(defn greet [username]
  (str "Hello " username))

; defn 这个 Macro 的定义(极度简化版本)
(defmacro defn [name args body]
  `(def ~name (fn ~args ~body)))
```

__应用开发者极少需要为他们自己创建 Macros__, 但这项功能对于类库开发者来说是不可或缺的,
通过 Macro 他们可以为应用开发者提供语言本身最大的灵活性.

#### 简单的替换

有几个 [Macro 字符](http://clojure.org/reader#The%20Reader--Macro%20characters)被用来进行简单的替换, 这使语言变得更简洁(不全是 Macro).

```clj
; 简单函数的一种简写
; #(...) => (fn [args] (...))

#(* 3 %)         ; => (fn [x] (* 3 x))

#(* 3 (+ %1 %2)) ; => (fn [x y] (* 3 (+ x y)))
```

## 就这些语法了

要熟练使用 ClojureScript 仅仅了解语法是不够的.
不过了解了上面这些语法, 你应该可以比较轻松地查看各种例子了,
也可以推理数据是怎么被求值的, 怎么被到处传递的.

```clj
; 在 JavaScript Console 中打印
(js/console.log "Hello World!")

; 创建局部的绑定(常量)
(let [a (+ 1 2)
      b (* 2 3)]
  (js/console.log "The value of a is" a)
  (js/console.log "The value of b is" b))

; 创建数字的序列
(range 4) ; => (0 1 2 3)

; 生成前 4 个自然数乘以 3 的结果
(map #(* % 3) (range 4))           ;=> (0 3 6 9)

; 序列中的元素个数
(count "Bob")     ; => 3
(count [4 5 2 3]) ; => 4

; 从列表中筛选出三个字母组成的名字
(def names ["Bob" "David" "Sue"])
(filter #(= (count %) 3) names) ; => ("Bob" "Sue")
```

(不需要担心圆括号太多而迷失. 所有的现代文本编辑器都可以做到配对的括号的高亮, 甚至自动缩进代码来保证可读性, 就跟其他语言标准的做法一样.)

## HTML 模板语法中的 ClojureScript 对比 JSX

阅读 [Jumping from HTML to ClojureScript](https://github.com/shaunlebron/jumping-from-html-to-clojurescript) 来了解 ClojureScript 语法怎样解决 JavaScript 社区中 JSX 解决的冗长性(verbosity?)和灵活性的问题.

[Jumping from HTML to ClojureScript]:https://github.com/shaunlebron/jumping-from-html-to-clojurescript
[JSX]:https://facebook.github.io/react/docs/jsx-in-depth.html

## 完整的手册

[ClojureScript API 手册]的语法章节可以找到所有可能的语法形式, 甚至还显示了源码怎样进行读取和解析.

[syntax section]:https://github.com/cljsinfo/cljs-api-docs/blob/catalog/INDEX.md#syntax
[ClojureScript API 手册]:http://cljs.github.io/api

## 实用资源

下面是我在学习 ClojureScript 时候的一些资源和步骤. (大部分介绍 Clojure 的资源对于 ClojureScript 也适用, 因为两者共享了很大的交集.)

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
