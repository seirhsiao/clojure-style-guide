
# Clojure 代码规范

> Role models are important. <br/>
> -- Officer Alex J. Murphy / RoboCop

原文地址：https://github.com/bbatsov/clojure-style-guide

这份Clojure代码规范旨在提供一系列的最佳实践，让现实工作中的Clojure程序员能够写出易于维护的代码，并能与他人协作和共享。一份反应真实需求的代码规范才能被人接收，而那些理想化的、甚至部分观点遭到程序员拒绝的代码规范注定不会长久——无论它有多出色。

这份规范由多个章节组成，每个章节包含一组相关的规则。我会尝试去描述每条规则背后的理念（过于明显的理念我就省略了）。

这些规则并不是我凭空想象的，它们出自于我作为一个专业软件开发工程师长久以来的工作积累，以及Clojure社区成员们的反馈和建议，还有各种广为流传的Clojure编程学习资源，如《Clojure Programming》、《The Joy of Clojure》等。

这份规范还处于编写阶段，部分章节有所缺失，内容并不完整；部分规则没有示例，或者示例还不能完全将其描述清楚。未来这些问题都会得到改进，只是请你了解这一情况。

你可以使用[Transmuter](https://github.com/TechnoGate/transmuter)生成一份本规范的PDF或HTML格式的文档。

本指南的翻译可在以下几种语言中：

* [Japanese](https://github.com/totakke/clojure-style-guide/blob/ja/README.md)
* [Korean](https://github.com/kwakbab/clojure-style-guide/blob/master/README-koKO.md)

## 目录

* [源代码的布局和组织结构](#source-code-layout--organization)　
* [语法](#syntax)
* [命名](#naming)
* [集合](#collections)
* [可变](#mutation)
* [字符串](#strings)
* [异常](#exceptions)
* [宏](#macros)
* [注释](#comments)
  + [注释中的标识](#comment-annotations)
* [惯用法](#existential)
* [工具](#tooling)

## <a name="source-code-layout--organization"></a>源代码的布局和组织结构

> 几乎所有人都认为任何代码风格都是丑陋且难以阅读的，除了自己的之外。
> 把这句话中的“除了自己之外”去掉，那差不多就能成立了。 <br/>
> —— Jerry Coffin 关于代码缩进的评论

* <a name="spaces"></a> 使用两个**空格**进行缩进，不使用制表符。
<sup>[[链接](#spaces)]</sup>

* <a name="body-indentation"></a>使用2个空格来缩进含参数部分的形式，
。这些形式包括所有的 `def` 形式 ，特实形式和宏，
 以及本地绑定形式 (例如： `loop`, `let`,
`when-let`) 和许多像 `when`, `cond`, `as->`, `cond->`, `case`,
`with-*`等的宏。
<sup>[[链接](#body-indentation)]</sup>

    ```Clojure
    ;; 很好
    (when something
      (something-else))

    (with-out-str
      (println "Hello, ")
      (println "world!"))

    ;; 糟糕 - 四个空格
    (when something
        (something-else))

    ;; 糟糕 - 一个空格
    (with-out-str
     (println "Hello, ")
     (println "world!"))
    ```

* <a name="vertically-align-fn-args"></a>
  垂直排列函数参数。
<sup>[[链接](#vertically-align-fn-args)]</sup>

    ```Clojure
    ;; 很好
    (filter even?
            (range 1 10))

    ;; 糟糕
    (filter even?
      (range 1 10))
    ```

* <a name="one-space-indent"></a>
使用一个空格缩进函数（宏）参数
当函数没有参数独占一行。
<sup>[[链接](#one-space-indent)]</sup>

    ```Clojure
    ;; 很好
    (filter
     even?
     (range 1 10))

    (or
     ala
     bala
     portokala)

    ;; 糟糕 - 两个空格缩进
    (filter
      even?
      (range 1 10))

    (or
      ala
      bala
      portokala)
    ```

* <a name="vertically-align-let-and-map"></a>
  对齐let绑定，以及map类型中的关键字。
<sup>[[链接](#vertically-align-let-and-map)]</sup>
    ```Clojure
    ;; 很好
    (let [thing1 "some stuff"
          thing2 "other stuff"]
      {:thing1 thing1
       :thing2 thing2})

    ;; 糟糕
    (let [thing1 "some stuff"
      thing2 "other stuff"]
      {:thing1 thing1
      :thing2 thing2})
    ```

* <a name="optional-new-line-after-fn-name"></a>
  针对没有文档字串的 defn，选择性忽略函数名与参数向量之间的新行。
<sup>[[链接](#optional-new-line-after-fn-name)]</sup>
    
    ```Clojure
    ;; 很好
    (defn foo
      [x]
      (bar x))

    ;; 很好
    (defn foo [x]
      (bar x))

    ;; 糟糕
    (defn foo
      [x] (bar x))
    ```

* <a name="multimethod-dispatch-val-placement"></a>
  将一个多重方法的`dispatch-val` 与函数名放置在同一行。
<sup>[[链接](#multimethod-dispatch-val-placement)]</sup>

    ```Clojure
    ;; 很好
    (defmethod foo :bar [x] (baz x))

    (defmethod foo :bar
      [x]
      (baz x))

    ;; 糟糕
    (defmethod foo
      :bar
      [x]
      (baz x))

    (defmethod foo
      :bar [x]
      (baz x))
    ```

* <a name="docstring-after-fn-name"></a>
  当为采用上述形式的函数添加字符串文档 - 
  注意正确应放置到函数名后面，而不是参数列表
  后面。后者虽然不是无效语法，不会造成错误，
  但是这仅仅只是将字符串作为函数体的一种形式，而不将其链接为
  该变量的文档。
<sup>[[链接](#docstring-after-fn-name)]</sup>

  ```Clojure
    ;; 好
    (defn foo
      "docstring"
      [x]
      (bar x))

    ;; 差
    (defn foo [x]
      "docstring"
      (bar x))
  ```

* <a name="oneline-short-fn"></a>
  选择性忽略短的参数向量与函数体之间的新行。
<sup>[[链接](#oneline-short-fn)]</sup>
    ```Clojure
    ;; 很好
    (defn foo [x]
      (bar x))

    ;; 适合简单的函数
    (defn foo [x] (bar x))

    ;;适合包含多元参数列表的函数
    (defn foo
      ([x] (bar x))
      ([x y]
       (if (predicate? x)
         (bar x)
         (baz x))))

    ;; 糟糕
    (defn foo
      [x] (if (predicate? x)
            (bar x)
            (baz x)))
    ```

* <a name="multiple-arity-indentation"></a>
  多元函数定义，各元数形式垂直对齐参数。
  <sup>[[链接](#multiple-arity-indentation)]</sup>

    ```Clojure
    ;; 很好
    (defn foo
      "I have two arities."
      ([x]
       (foo x 1))
      ([x y]
       (+ x y)))

    ;; 糟糕 - 多出的缩进
    (defn foo
      "I have two arities."
      ([x]
        (foo x 1))
      ([x y]
        (+ x y)))
    ```

* <a name="multiple-arity-order"></a> 
  按照从少到多的参数，排序函数的多元数形式。多元素的情况下，共同
  功能是某个k参数完全指定函数的
  行为，并且元素个数 Ñ < K 部分地应用在K元数，和
  元素 N> K提供在K元数超过可变参数的实现。
  <sup>[[链接](#multiple-arity-order)]</sup>

    ```Clojure
    ;; 很好 - 很容易扫描第n个参数形式
    (defn foo
      "I have two arities."
      ([x]
       (foo x 1))
      ([x y]
       (+ x y)))

    ;; 还好 - 其他元素应用两倍元数形式
    (defn foo
      "I have two arities."
      ([x y]
       (+ x y))
      ([x]
       (foo x 1))
      ([x y z & more]
       (reduce foo (foo x (foo y z)) more)))

    ;; 糟糕 - 无序的，毫无理由这样
    (defn foo
      ([x] 1)
      ([x y z] (foo x (foo y z)))
      ([x y] (+ x y))
      ([w x y z & more] (reduce foo (foo w (foo x (foo y z))) more)))
    ```

* <a name="align-docstring-lines"></a>
  缩进多行的文档字串。
<sup>[[链接](#align-docstring-lines)]</sup>

    ```Clojure
    ;; 很好
    (defn foo
      "Hello there. This is
      a multi-line docstring."
      []
      (bar))

    ;; 糟糕
    (defn foo
      "Hello there. This is
    a multi-line docstring."
      []
      (bar))
    ```

* <a name="crlf"></a>
   使用Unix风格的换行符（*BSD、Solaris、Linux、OSX用户无需设置，Windows用户则需要格外注意了） 
<sup>[[链接](#crlf)]</sup>

    * 如果你使用 Git ，你也许会想加入下面这个配置，来保护你的项目被 Windows 的行编码侵入：
    ```
    bash$ git config --global core.autocrlf true
    ```


* <a name="bracket-spacing"></a>
若有任何文字在左括号、中括号、大括号前（`(`, `[`, `{`），或是在右括号、中括号、大括号之后（`)`, `]`, `}`），将文字与括号用一个空格分开。反过来说，在左括号后、右括号前不要有空格。
<sup>[[链接](#bracket-spacing)]</sup>

    ```Clojure
    ;; 很好
    (foo (bar baz) quux)

    ;; 糟糕
    (foo(bar baz)quux)
    (foo ( bar baz ) quux)
    ```

> Syntactic sugar causes semicolon cancer. <br/>
> -- Alan Perlis
    
* <a name="no-commas-for-seq-literals"></a>
  不要在序列化的集合类型的字面常量语法里使用逗号。
<sup>[[链接](#no-commas-for-seq-literals)]</sup>

    ```Clojure
    ;; 很好
    [1 2 3]
    (1 2 3)

    ;; 糟糕
    [1, 2, 3]
    (1, 2, 3)
    ```

* <a name="opt-commas-in-map-literals"></a>
 明智的使用逗号与换行来加强 map 的可读性。
<sup>[[链接](#opt-commas-in-map-literals)]</sup>

    ```Clojure
    ;; 很好
    {:name "Bruce Wayne" :alter-ego "Batman"}

    ;; 很好， 且会增强可读性
    {:name "Bruce Wayne"
     :alter-ego "Batman"}

    ;; 很好， 且较为紧凑
    {:name "Bruce Wayne", :alter-ego "Batman"}
    ```

* <a name="gather-trailing-parens"></a>
  将所有尾括号放在同一行。
<sup>[[链接](#gather-trailing-parens)]</sup>

    ```Clojure
    ;; 很好; 同一行
    (when something
      (something-else))

    ;; 糟糕; 不同行
    (when something
      (something-else)
    )
    ```


    
* <a name="empty-lines-between-top-level-forms"></a>
 顶层形式用空行间隔开来。
<sup>[[链接](#empty-lines-between-top-level-forms)]</sup>

    ```Clojure
    ;; 很好
    (def x ...)

    (defn foo ...)

    ;; 糟糕
    (def x ...)
    (defn foo ...)
    ```

    一个例外是相关`def`分组在一起。

    ```Clojure
    ;; 很好
    (def min-rows 10)
    (def max-rows 20)
    (def min-cols 15)
    (def max-cols 30)
    ```

* <a name="no-blank-lines-within-def-forms"></a>
  函数或宏定义中间不要放空行。一个例外，可制成以指示分组
比如发现成对结构` let`和` cond`。
<sup>[[链接](#no-blank-lines-within-def-forms)]</sup>

 * <a name="no-blank-lines-within-def-forms"></a>
  
* <a name="80-character-limits"></a>
  可行的场合下，避免每行超过 80 字符。
<sup>[[链接](#80-character-limits)]</sup>

* <a name="no-trailing-whitespace"></a>
  避免尾随的空白。
<sup>[[链接](#no-trailing-whitespace)]</sup>

* <a name="one-file-per-namespace"></a>
  一个文件、一个命名空间。
<sup>[[链接](#one-file-per-namespace)]</sup>


* <a name="comprehensive-ns-declaration"></a>
 每个命名空间用 `ns` 形式开始，加上 `refer`、`require`、`use` 以及 `import`。
<sup>[[链接](#comprehensive-ns-declaration)]</sup>

    ```Clojure
    (ns examples.ns
      (:refer-clojure :exclude [next replace remove])
      (:require [clojure.string :as s :refer [blank?]]
                [clojure.set :as set]
                [clojure.java.shell :as sh])
      (:import java.util.Date
               java.text.SimpleDateFormat
               [java.util.concurrent Executors
                                     链接edBlockingQueue]))
    ```

* <a name="prefer-require-over-use"></a>
  在 `ns` 宏中优先使用 `:require :as` 胜于 `:require :refer` 胜于 `:require
  :refer :all`. 优先使用 `:require` 胜于 `:use`; 后者的形式应该是
考虑使用新的代码。
<sup>[[链接](#prefer-require-over-use)]</sup>


    ```Clojure
    ;; 很好
    (ns examples.ns
      (:require [clojure.zip :as zip]))

    ;; 很好
    (ns examples.ns
      (:require [clojure.zip :refer [lefts rights]))

    ;; 可以接受的
    (ns examples.ns
      (:require [clojure.zip :refer :all]))

    ;; 糟糕
    (ns examples.ns
      (:use clojure.zip))
    ```

* <a name="no-single-segment-namespaces"></a>
  避免单段的命名空间。
  <sup>[[链接](#no-single-segment-namespaces)]</sup>

    ```Clojure
    ;; 很好
    (ns example.ns)

    ;; 糟糕
    (ns example)
    ```

  

* <a name="namespaces-with-5-segments-max"></a>
  避免使用过长的命名空间（不超过五段）。
<sup>[[链接](#namespaces-with-5-segments-max)]</sup>

* <a name="10-loc-per-fn-limit"></a>
  函数避免超过 10 行代码。事实上，大多数函数应保持在5行代码以内。
<sup>[[链接](#10-loc-per-fn-limit)]</sup>

* <a name="4-positional-fn-params-limit"></a>
  参数列表避免超过 3 个或 4 个位置参数。
<sup>[[链接](#4-positional-fn-params-limit)]</sup>

* <a name="forward-references"></a>
  避免向前引用。它们偶尔必要的，但这样的场合
  实际上很罕见。
<sup>[[链接](#forward-references)]</sup>


## <a name="syntax"></a>语法

* <a name="ns-fns-only-in-repl"></a>
  避免使用操作命名空间的函数，像是：`require` 与 `refer`。他们在 REPL 之外完全用不到。
<sup>[[链接](#ns-fns-only-in-repl)]</sup>

* <a name="declare"></a>
  使用declare实现向前引用。
<sup>[[链接](#declare)]</sup>

* <a name="higher-order-fns"></a>
  优先使用`map`这类高阶函数，而非`loop/recur`。
<sup>[[链接](#higher-order-fns)]</sup>

* <a name="pre-post-conditions"></a>
  优先使用前置、后置条件来检测函数参数和返回值。
<sup>[[链接](#pre-post-conditions)]</sup>

    ```Clojure
    ;; 很好
    (defn foo [x]
      {:pre [(pos? x)]}
      (bar x))

    ;; 糟糕
    (defn foo [x]
      (if (pos? x)
        (bar x)
        (throw (IllegalArgumentException. "x must be a positive number!")))
    ```

* <a name="dont-def-vars-inside-fns"></a>
  不要在函数中定义变量。
<sup>[[链接](#dont-def-vars-inside-fns)]</sup>

    ```Clojure
    ;; 非常糟糕
    (defn foo []
      (def x 5)
      ...)
    ```

* <a name="dont-shadow-clojure-core"></a>
  本地变量名不应覆盖clojure.core中定义的函数。
<sup>[[链接](#dont-shadow-clojure-core)]</sup>

    ```Clojure
    ;; 糟糕 - 这样一来函数中调用`clojure.core/map`时就需要指定完整的命名空间了。
    (defn foo [map]
      ...)
    ```

* <a name="alter-var"></a>
  使用 `alter-var-root` 替代 `def` 去改变变量的值。

    ```Clojure
    ;; good
    (def thing 1) ; value of thing is now 1
    ; do some stuff with thing
    (alter-var-root #'thing (constantly nil)) ; value of thing is now nil

    ;; bad
    (def thing 1)
    ; do some stuff with thing
    (def thing nil)
    ; value of thing is now nil
    ```

* <a name="nil-punning"></a>
  使用 `seq` 来判断一个序列是否为空（这个技巧有时候称为 *nil punning）。
<sup>[[链接](#nil-punning)]</sup>

    ```Clojure
    ;; 很好
    (defn print-seq [s]
      (when (seq s)
        (prn (first s))
        (recur (rest s))))

    ;; 糟糕
    (defn print-seq [s]
      (when-not (empty? s)
        (prn (first s))
        (recur (rest s))))
    ```

* <a name="to-vector"></a>
  需要将序列转换向量，优先使用 `vec` 而不是 `into` 。
<sup>[[链接](#to-vector)]</sup>

    ```Clojure
    ;; 很好
    (vec some-seq)

    ;; 糟糕
    (into [] some-seq)
    ```
    
* <a name="when-instead-of-single-branch-if"></a>
  使用 `when` 替代 `(if ... (do ...)`。
<sup>[[链接](#when-instead-of-single-branch-if)]</sup>

    ```Clojure
    ;; 很好
    (when pred
      (foo)
      (bar))

    ;; 糟糕
    (if pred
      (do
        (foo)
        (bar)))
    ```

* <a name="if-let"></a>
  使用 `if-let` 替代 `let` + `if`。
<sup>[[链接](#if-let)]</sup>

    ```Clojure
    ;; 很好
    (if-let [result (foo x)]
      (something-with result)
      (something-else))

    ;; 糟糕
    (let [result (foo x)]
      (if result
        (something-with result)
        (something-else)))
    ```

* <a name="when-let"></a>
  使用 `when-let` 替代 `let` + `when`。
<sup>[[链接](#when-let)]</sup>

    ```Clojure
    ;; 很好
    (when-let [result (foo x)]
      (do-something-with result)
      (do-something-more-with result))

    ;; 糟糕
    (let [result (foo x)]
      (when result
        (do-something-with result)
        (do-something-more-with result)))
    ```

* <a name="if-not"></a>
  使用 `if-not` 替代 `(if (not ...) ...)`。
<sup>[[链接](#if-not)]</sup>

    ```Clojure
    ;; 很好
    (if-not pred
      (foo))

    ;; 糟糕
    (if (not pred)
      (foo))
    ```

* <a name="when-not"></a>
  使用 `when-not` 替代 `(when (not ...) ...)`。
<sup>[[链接](#when-not)]</sup>

    ```Clojure
    ;; 很好
    (when-not pred
      (foo)
      (bar))

    ;; 糟糕
    (when (not pred)
      (foo)
      (bar))
    ```

* <a name="when-not-instead-of-single-branch-if-not"></a>
  使用 `when-not` 替代 `(if-not ... (do ...)`。
<sup>[[链接](#when-not-instead-of-single-branch-if-not)]</sup>

    ```Clojure
    ;; 很好
    (when-not pred
      (foo)
      (bar))

    ;; 糟糕
    (if-not pred
      (do
        (foo)
        (bar)))
    ```

* <a name="not-equal"></a>
  使用 `not=` 替代 `(not (= ...))`。
<sup>[[链接](#not-equal)]</sup>

    ```Clojure
    ;; 很好
    (not= foo bar)

    ;; 糟糕
    (not (= foo bar))
    ```
    
* <a name="printf"></a>
  使用 `printf` 替代 `(print (format ...))`。
<sup>[[链接](#printf)]</sup>

    ```Clojure
    ;; 很好
    (printf "Hello, %s!\n" name)

    ;; 好
    (println (format "Hello, %s!" name))
    ```


* <a name="multiple-arity-of-gt-and-ls-fns"></a>
  在做比较，请考虑， Clojure的函数`<`
  `>`等，接受可变数量的参数的函数。
<sup>[[链接](#multiple-arity-of-gt-and-ls-fns)]</sup>

    ```Clojure
    ;; 很好
    (< 5 x 10)

    ;; 糟糕
    (and (> x 5) (< x 10))
    ```

* <a name="single-param-fn-literal"></a>
  当匿名函数只有一个参数时，优先使用 `%` ，而非 `%1` 。
<sup>[[链接](#single-param-fn-literal)]</sup>

    ```Clojure
    ;; 很好
    #(Math/round %)

    ;; 糟糕
    #(Math/round %1)
    ```

* <a name="multiple-params-fn-literal"></a>
  当匿名函数有多个参数时，优先使用 `%1`，而非 `%` 。
<sup>[[链接](#multiple-params-fn-literal)]</sup>

    ```Clojure
    ;; 很好
    #(Math/pow %1 %2)

    ;; 糟糕
    #(Math/pow % %2)
    ```

* <a name="no-useless-anonymous-fns"></a>
  只有在必要的时候才使用匿名函数。
<sup>[[链接](#no-useless-anonymous-fns)]</sup>

    ```Clojure
    ;; 很好
    (filter even? (range 1 10))

    ;; 糟糕
    (filter #(even? %) (range 1 10))
    ```

* <a name="no-multiple-forms-fn-literals"></a>
  若函数体由一个以上形式组成，不要使用匿名函数。
<sup>[[链接](#no-multiple-forms-fn-literals)]</sup>

    ```Clojure
    ;; 很好
    (fn [x]
      (println x)
      (* x 2))

    ;; 糟糕 (你需要明确得使用到do)
    #(do (println %)
         (* % 2))
    ```

* <a name="complement"></a>
  在特定情况下优先使用`complement`，而非匿名函数。
<sup>[[链接](#complement)]</sup>

    ```Clojure
    ;; 很好
    (filter (complement some-pred?) coll)

    ;; 糟糕
    (filter #(not (some-pred? %)) coll)
    ```

    这个规则应该在函数有明确的反函数时忽略（如：even? 与 odd?）。 

* <a name="comp"></a>
  某些情况下可以用 `comp` 使代码更简洁。
<sup>[[链接](#comp)]</sup>

    ```Clojure
    ;; Assuming `(:require [clojure.string :as str])`...

    ;; 很好
    (map #(str/capitalize (str/trim %)) ["top " " test "])

    ;; 更好
    (map (comp str/capitalize str/trim) ["top " " test "])
    ```
  
* <a name="partial"></a>
  某些情况下可以用 `partial` 使代码更简洁。
<sup>[[链接](#partial)]</sup>

    ```Clojure
    ;; 很好
    (map #(+ 5 %) (range 1 10))

    ;; (或许) 更好
    (map (partial + 5) (range 1 10))
    ```


* <a name="threading-macros"></a>
  当遇到嵌套调用时，建议使用 `->` 宏和 `->>` 宏。
<sup>[[链接](#threading-macros)]</sup>

    ```Clojure
    ;; 很好
    (-> [1 2 3]
        reverse
        (conj 4)
        prn)

    ;; 不够好
    (prn (conj (reverse [1 2 3])
               4))

    ;; 很好
    (->> (range 1 10)
         (filter even?)
         (map (partial * 2)))

    ;; 不够好
    (map (partial * 2)
         (filter even? (range 1 10)))
    ```
    
    
* <a name="dot-dot-macro"></a>
  当需要连续调用Java类的方法时，优先使用 `..` ，而非 `->` 。
<sup>[[链接](#dot-dot-macro)]</sup>

    ```Clojure
    ;; 很好
    (-> (System/getProperties) (.get "os.name"))

    ;; 更好
    (.. System getProperties (get "os.name"))
    ```
    
* <a name="else-keyword-in-cond"></a>
  在 `cond` 和 `condp` 中，使用 `:else` 来处理不满足条件的情况。
<sup>[[链接](#else-keyword-in-cond)]</sup>

    ```Clojure
    ;; 很好
    (cond
      (< n 0) "negative"
      (> n 0) "positive"
      :else "zero"))

    ;; 糟糕
    (cond
      (< n 0) "negative"
      (> n 0) "positive"
      true "zero"))
    ```

* <a name="condp"></a>
  当比较的变量和方式相同时，优先使用 `condp` ，而非 `cond` 。
<sup>[[链接](#condp)]</sup>

    ```Clojure
    ;; 很好
    (cond
      (= x 10) :ten
      (= x 20) :twenty
      (= x 30) :forty
      :else :dunno)

    ;; 更好
    (condp = x
      10 :ten
      20 :twenty
      30 :forty
      :dunno)
    ```

* <a name="case"></a>
  当条件是常量时，优先使用 `case` ，而非 `cond` 或 `condp` 。
<sup>[[链接](#case)]</sup>

    ```Clojure
    ;; 很好
    (cond
      (= x 10) :ten
      (= x 20) :twenty
      (= x 30) :forty
      :else :dunno)

    ;; 更好
    (condp = x
      10 :ten
      20 :twenty
      30 :forty
      :dunno)

    ;; 最佳
    (case x
      10 :ten
      20 :twenty
      30 :forty
      :dunno)
    ```
* <a name="shor-forms-in-cond"></a>
   如果不能在视觉上使用注释与空行两两分组提示，则在 `cond` 相关的的形式中，使用短形式。
<sup>[[链接](#shor-forms-in-cond)]</sup>

    ```Clojure
    ;; 很好
    (cond
      (test1) (action1)
      (test2) (action2)
      :else   (default-action))

    ;; 还行
    (cond
      ;; test case 1
      (test1)
      (long-function-name-which-requires-a-new-line
        (complicated-sub-form
          (-> 'which-spans multiple-lines)))

      ;; test case 2
      (test2)
      (another-very-long-function-name
        (yet-another-sub-form
          (-> 'which-spans multiple-lines)))

      :else
      (the-fall-through-default-case
        (which-also-spans 'multiple
                          'lines)))
    ```

* <a name="set-as-predicate"></a>
  某些情况下，使用 `set` 作为判断条件。
<sup>[[链接](#set-as-predicate)]</sup>

    ```Clojure
    ;; 很好
    (remove #{1} [0 1 2 3 4 5])

    ;; 糟糕
    (remove #(= % 1) [0 1 2 3 4 5])

    ;; 很好
    (count (filter #{\a \e \i \o \u} "mary had a little lamb"))

    ;; 糟糕
    (count (filter #(or (= % \a)
                        (= % \e)
                        (= % \i)
                        (= % \o)
                        (= % \u))
                   "mary had a little lamb"))
    ```

    
* <a name="inc-and-dec"></a>
  使用 `(inc x)` 和 `(dec x)` 替代 `(+ x 1)` 和 `(- x 1)`。
<sup>[[链接](#inc-and-dec)]</sup>

* <a name="pos-and-neg"></a>
  使用 `(pos? x)`、`(neg? x)` 、以及`(zero? x)` 替代 `(> x 0)` 、`(< x 0)` 、和`(= x 0)`。
<sup>[[链接](#pos-and-neg)]</sup>

* <a name="list-star-instead-of-nested-cons"></a>
  Useinstead of a series of nestedinvocations.
  使用 `list*` 替代内部嵌套多个 `cons` 。
<sup>[[链接](#list-star-instead-of-nested-cons)]</sup>

    ```Clojure
    # 很好
    (list* 1 2 3 [4 5])

    # 糟糕
    (cons 1 (cons 2 (cons 3 [4 5])))

    ```

* <a name="sugared-java-interop"></a>
  进行Java交互时，优先使用Clojure提供的语法糖。
<sup>[[链接](#sugared-java-interop)]</sup>

    ```Clojure
    ;;; 创建对象
    ;; 很好
    (java.util.ArrayList. 100)

    ;; 糟糕
    (new java.util.ArrayList 100)

    ;;; 调用静态方法
    ;; 很好
    (Math/pow 2 10)

    ;; 糟糕
    (. Math pow 2 10)

    ;;; 调用实例方法
    ;; 很好
    (.substring "hello" 1 3)

    ;; 糟糕
    (. "hello" substring 1 3)

    ;;; 访问静态属性
    ;; 很好
    Integer/MAX_VALUE

    ;; 糟糕
    (. Integer MAX_VALUE)

    ;;; 访问实例属性
    ;; 很好
    (.someField some-object)

    ;; 糟糕
    (. some-object someField)
    ```
    
* <a name="compact-metadata-notation-for-true-flags"></a>
  Use the compact metadata notation for metadata that contains only
  slots whose keys are keywords and whose value is boolean .
  当元数据的键是关键字和值是 `true` ，使用紧凑形式标记元数据。
<sup>[[链接](#compact-metadata-notation-for-true-flags)]</sup>

    ```Clojure
    ;; 很好
    (def ^:private a 5)

    ;; 糟糕
    (def ^{:private true} a 5)
    ```

* <a name="private"></a>
  指出代码的私有部分。
<sup>[[链接](#private)]</sup>

    ```Clojure
    ;; 很好
    (defn- private-fun [] ...)

    (def ^:private private-var ...)

    ;; 糟糕
    (defn private-fun [] ...) ; not private at all

    (defn ^:private private-fun [] ...) ; overly verbose

    (def private-var ...) ; not private at all
    ```

* <a name="access-private-var"></a>
  使用 `@#'some.ns/var` 形式，访问私有变量（如：为了测试）。
<sup>[[链接](#access-private-var)]</sup>

* <a name="attach-metadata-carefully"></a>
  Be careful regarding what exactly do you attach metadata to.
  小心你添加元数据的对象。
<sup>[[链接](#attach-metadata-carefully)]</sup>

    ```Clojure
    ;; 我们添加元数据到变量 `a`中
    (def ^:private a {})
    (meta a) ;=> nil
    (meta #'a) ;=> {:private true}

    ;; 我们添加元数据到空的hash-map中
    (def a ^:private {})
    (meta a) ;=> {:private true}
    (meta #'a) ;=> nil
    ```



## <a name="naming"></a>命名

> 编程中真正的难点只有两个：验证缓存的有效性；命名。
> -- Phil Karlton

* <a name="ns-naming-schemas"></a>
  命名空间建议使用以下两种方式：
<sup>[[链接](#ns-naming-schemas)]</sup>

    * `项目名称.模块名称`
    * `组织名称.项目名称.模块名称`

* <a name="lisp-case-ns"></a>
  对于命名空间中较长的元素，使用 `lisp-case` 格式，如（`bruce.project-euler`）。
<sup>[[链接](#lisp-case-ns)]</sup>

* <a name="lisp-case"></a>
  使用 `lisp-case` 格式来命名函数和变量。
<sup>[[链接](#lisp-case)]</sup>

    ```Clojure
    ;; 很好
    (def some-var ...)
    (defn some-fun ...)

    ;; 糟糕
    (def someVar ...)
    (defn somefun ...)
    (def some_fun ...)
    ```

* <a name="CamelCase-for-protocols-records-structs-and-types"></a>
  使用 `CamelCase` 来命名接口（protocol）、记录（record）、结构和类型（struct & type）。对于HTTP、RFC、XML等缩写，仍保留其大写格式。
<sup>[[链接](#CamelCase-for-protocols-records-structs-and-types)]</sup>

* <a name="pred-with-question-mark"></a>
  对于返回布尔值的函数名称，使用问号结尾，（如: even?）。
<sup>[[链接](#pred-with-question-mark)]</sup>

   ```Clojure
    ;; 很好
    (defn palindrome? ...)

    ;; 糟糕
    (defn palindrome-p ...) ; Common Lisp style
    (defn is-palindrome ...) ; Java style
   ```

* <a name="changing-state-fns-with-exclamation-mark"></a>
  The names of functions/macros that are not safe in STM transactions
  should end with an exclamation mark (e.g. `reset!`).
  当方法或宏不能在STM中安全使用时，须以感叹号结尾，（如：reset!）。
<sup>[[链接](#changing-state-fns-with-exclamation-mark)]</sup>

* <a name="arrow-instead-of-to"></a>
  命名类型转换函数时使用 `->` ，而非 `to` 。
<sup>[[链接](#arrow-instead-of-to)]</sup>

    ```Clojure
    ;; 很好
    (defn f->c ...)

    ;; 不够好
    (defn f-to-c ...)
    ```

* <a name="earmuffs-for-dynamic-vars"></a>
  对于可供重绑定的变量（即动态变量），使用星号括起，（如：*earmuffs*）。
<sup>[[链接](#earmuffs-for-dynamic-vars)]</sup>

    ```Clojure
    ;; good
    (def ^:dynamic *a* 10)

    ;; bad
    (def ^:dynamic a 10)
    ```

* <a name="dont-flag-constants"></a>
  无需对常量名进行特殊的标识，因为所有的变量都应该是常量，除非有特别说明。
<sup>[[链接](#dont-flag-constants)]</sup>

* <a name="underscore-for-unused-bindings"></a>
  对于解构过程中或参数列表中忽略的元素，使用 `_` 来表示。
<sup>[[链接](#underscore-for-unused-bindings)]</sup>

  ```Clojure
    ;; 很好
    (let [[a b _ c] [1 2 3 4]]
      (println a b c))

    (dotimes [_ 3]
      (println "Hello!"))

    ;; 糟糕
    (let [[a b c d] [1 2 3 4]]
      (println a b d))

    (dotimes [i 3]
      (println "Hello!"))
  ```
    
* <a name="idiomatic-names"></a>
  参考 `clojure.core` 中的命名规范，如 `pred` 、`coll` 。
<sup>[[链接](#idiomatic-names)]</sup>

    * 函数：
        * `f`，`g`，`h` - 参数内容是一个函数
        * `n` - 整数，通常是一个表示大小的值
        * `index`, `i` - 整数索引
        * `x`, `y` - 数值
        * `xs` - 序列
        * `m` - 映射
        * `s` - 字符串
        * `re` - 正则表达式
        * `coll` - 集合
        * `pred` - 谓词闭包
        * `& more` - 可变参数
        * `xf` - xform, 一个转换器
    * 宏：
        * `expr` - 表达式
        * `body` - 宏的主体
        * `binding` - 一个向量，包含宏的绑定  


## <a name="collections">集合

> 用100种函数去操作同一种数据结构，要好过用10种函数操作10种数据结构。<br/>
> -- Alan J. Perlis

* <a name="avoid-lists"></a>
  避免使用列表（list）来存储数据（除非它真的就是你想要的）。
<sup>[[链接](#avoid-lists)]</sup>

* <a name="keywords-for-hash-keys"></a>
  优先使用关键字（keyword），而非普通的哈希键。
<sup>[[链接](#keywords-for-hash-keys)]</sup>

    ```Clojure
    ;; 很好
    {:name "Bruce" :age 30}

    ;; 糟糕
    {"name" "Bruce" "age" 30}
    ```

* <a name="literal-col-syntax"></a>
  编写集合时，优先使用字面的语法形式，而非构造函数。但是，在定义唯一值集合（set）时，只有当元素都是常量时才可使用字面语法，否则应使用构造函数。
<sup>[[链接](#literal-col-syntax)]</sup>

   ```Clojure
    ;; 很好
    [1 2 3]
    #{1 2 3}
    (hash-set (func1) (func2)) ; 元素在运行时确定

    ;; 糟糕
    (vector 1 2 3)
    (hash-set 1 2 3)
    #{(func1) (func2)} ; 若(func1)和(func2)的值相等，则会抛出运行时异常。
   ```

* <a name="avoid-index-based-coll-access"></a>
  避免使用数值索引来访问集合元素。
<sup>[[链接](#avoid-index-based-coll-access)]</sup>

* <a name="keywords-as-fn-to-get-map-values"></a>
  优先使用关键字来获取哈希表（map）中的值。
<sup>[[链接](#keywords-as-fn-to-get-map-values)]</sup>

    ```Clojure
    (def m {:name "Bruce" :age 30})

    ;; 很好
    (:name m)

    ;; 太过啰嗦
    (get m :name)

    ;; 糟糕 - 可能抛出空指针异常
    (m :name)
    ```

* <a name="colls-as-fns"></a>
  集合可以被用作函数。
<sup>[[链接](#colls-as-fns)]</sup>

    ```Clojure
    ;; 很好
    (filter #{\a \e \o \i \u} "this is a test")

    ;; 糟糕 - 不够美观
    ```

* <a name="keywords-as-fns"></a>
  关键字可以被用作函数。
<sup>[[链接](#keywords-as-fns)]</sup>

    ```Clojure
    ((juxt :a :b) {:a "ala" :b "bala"})
    ```

* <a name="avoid-transient-colls"></a>
只有在非常强调性能的情况下才可使用瞬时集合（transient collection）。
<sup>[[链接](#avoid-transient-colls)]</sup>

* <a name="avoid-java-colls"></a>
  避免使用Java集合。
<sup>[[链接](#avoid-java-colls)]</sup>

* <a name="avoid-java-arrays"></a>
  避免使用Java数组，除非遇到需要和Java类进行交互，或需要高性能地处理基本类型时才可使用。
<sup>[[链接](#avoid-java-arrays)]</sup>

    
## <a name="mutation"></a>可变

### 引用（Refs）

* <a name="refs-io-macro"></a>
  建议所有的IO操作都使用 `io!` 宏进行包装，以免不小心在事务中调用了这些代码。
<sup>[[链接](#refs-io-macro)]</sup>

* <a name="refs-avoid-ref-set"></a>
  避免使用 `ref-set` 。
<sup>[[链接](#refs-avoid-ref-set)]</sup>

    ```Clojure
    (def r (ref 0))

    ;; 很好
    (dosync (alter r + 5))

    ;; 糟糕
    (dosync (ref-set r 5))
    ```
    
* <a name="refs-small-transactions"></a>
  控制事务的大小，(即事务所执行的工作越少越好)。
as small as possible.
<sup>[[链接](#refs-small-transactions)]</sup>

* <a name="refs-avoid-short-long-transactions-with-same-ref"></a>
  避免出现短期事务和长期事务访问同一个引用（Ref）的情形。
<sup>[[链接](#refs-avoid-short-long-transactions-with-same-ref)]</sup>

    

### 代理（Agents）

* <a name="agents-send"></a>
  `send` 仅使用于计算密集型、不会因IO等因素阻塞的线程。
<sup>[[链接](#agents-send)]</sup>

* <a name="agents-send-off"></a>
   `send-off` 则用于会阻塞、休眠的线程。
<sup>[[链接](#agents-send-off)]</sup>

### 原子（Atoms）

* <a name="atoms-no-update-within-transactions"></a>
  避免在事务中更新原子。
<sup>[[链接](#atoms-no-update-within-transactions)]</sup>

* <a name="atoms-prefer-swap-over-reset"></a>
  尽量使用 `swap!` ，而不是 `reset!`。
<sup>[[链接](#atoms-prefer-swap-over-reset)]</sup>

    ```Clojure
    (def a (atom 0))

    ;; 很好
    (swap! a + 5)

    ;; 不够好
    (reset! a 5)
    ```
    
    
## <a name="strings"></a>字符串

* <a name="prefer-clojure-string-over-interop"></a>
  优先使用 `clojure.string` 中提供的字符串操作函数，而不是Java中提供的或是自己编写的函数。
<sup>[[链接](#prefer-clojure-string-over-interop)]</sup>

    ```Clojure
    ;; 很好
    (clojure.string/upper-case "bruce")

    ;; 糟糕
    (.toUpperCase "bruce")
    ```

## <a name="exceptions"></a>异常

* <a name="reuse-existing-exception-types"></a>
  复用已有的异常类型，符合语言习惯的 Clojure 代码，当真的抛出异常时，会抛出标准类型的异常
  (如： `java.lang.IllegalArgumentException`,
  `java.lang.UnsupportedOperationException`,
  `java.lang.IllegalStateException`, `java.io.IOException`).
<sup>[[链接](#reuse-existing-exception-types)]</sup>

* <a name="prefer-with-open-over-finally"></a>
  优先使用 `with-open` ，而非 `finally`。
<sup>[[链接](#prefer-with-open-over-finally)]</sup>


## <a name="macros"></a>宏

* <a name="dont-write-macro-if-fn-will-do"></a>
  如果可以用函数实现相同功能，不要编写一个宏。
<sup>[[链接](#dont-write-macro-if-fn-will-do)]</sup>

* <a name="write-macro-usage-before-writing-the-macro"></a>
  首先编写一个宏的用例，尔后再编写宏本身。
<sup>[[链接](#write-macro-usage-before-writing-the-macro)]</sup>

* <a name="break-complicated-macros"></a>
  尽可能将一个复杂的宏拆解为多个小型的函数。
<sup>[[链接](#break-complicated-macros)]</sup>

* <a name="macros-as-syntactic-sugar"></a>
  宏只应用于简化语法，其核心应该是一个普通的函数。
<sup>[[链接](#macros-as-syntactic-sugar)]</sup>

* <a name="syntax-quoted-forms"></a>
  使用语法转义（syntax-quote，即反引号），而非手动构造list。
<sup>[[链接](#syntax-quoted-forms)]</sup>


## <a name="comments"></a>注释

> 好的代码本身就是文档。因此在添加注释之前，先想想自己该如何改进代码，让它更容易理解。做到这一点后，再通过注释让代码更清晰。<br/>
> -- Steve McConnell

* <a name="self-documenting-code"></a>
  学会编写容易理解的代码，然后忽略下文的内容。真的！
<sup>[[链接](#self-documenting-code)]</sup>

* <a name="four-semicolons-for-heading-comments"></a>
  对于标题型的注释，使用至少四个分号起始。
<sup>[[链接](#four-semicolons-for-heading-comments)]</sup>

* <a name="three-semicolons-for-top-level-comments"></a>
  对于顶层注释，使用三个分号起始。
<sup>[[链接](#three-semicolons-for-top-level-comments)]</sup>

* <a name="two-semicolons-for-code-fragment"></a>
  为某段代码添加注释时，使用两个分号起始，且应与该段代码对齐。
<sup>[[链接](#two-semicolons-for-code-fragment)]</sup>

* <a name="one-semicolon-for-margin-comments"></a>
  对于行尾注释，使用一个分号起始即可。
<sup>[[链接](#one-semicolon-for-margin-comments)]</sup>

* <a name="semicolon-space"></a>
  分号后面要有一个空格。
<sup>[[链接](#semicolon-space)]</sup>
 
    ```Clojure
    ;;;; Frob Grovel

    ;;; 这段代码有以下前提:
    ;;;   1. Foo.
    ;;;   2. Bar.
    ;;;   3. Baz.

    (defn fnord [zarquon]
      ;; If zob, then veeblefitz.
      (quux zot
            mumble             ; Zibblefrotz.
            frotz))
    ```
   
 * <a name="english-syntax"></a>
  对于完整的句子的注释，句首字母应该大写，句与句之间用一个空格分隔。
  [one space](http://en.wikipedia.org/wiki/Sentence_spacing).
<sup>[[链接](#english-syntax)]</sup>

* <a name="no-superfluous-comments"></a>
  避免冗余的注释。
<sup>[[链接](#no-superfluous-comments)]</sup>

    ```Clojure
    ;; 糟糕
    (inc counter) ; increments counter by one
    ```
   
* <a name="comment-upkeep"></a>
  注释要和代码同步更新。过期的注释还不如没有注释。
at all.
<sup>[[链接](#comment-upkeep)]</sup>

* <a name="dash-underscore-reader-macro"></a>
  有时，使用 `#_`  宏要优于普通的注释
<sup>[[链接](#dash-underscore-reader-macro)]</sup>


    ```Clojure
    ;; 很好
    (+ foo #_(bar x) delta)

    ;; 糟糕
    (+ foo
       ;; (bar x)
       delta)
    ```

> 好的代码和好的笑话一样，不需要额外的解释。 <br/>
> -- Russ Olsen

* <a name="refactor-dont-comment"></a>
  避免使用注释去描述一段写得很糟糕的代码。重构它，让它更为可读。（做或者不做，没有尝试这一说。--Yoda）
<sup>[[链接](#refactor-dont-comment)]</sup>

### <a name="comment-annotations"></a>注释中的标识

* <a name="annotate-above"></a>
  标识应该写在对应代码的上一行。
<sup>[[链接](#annotate-above)]</sup>

* <a name="annotate-keywords"></a>
  标识后面是一个冒号和一个空格，以及一段描述文字。
<sup>[[链接](#annotate-keywords)]</sup>

* <a name="indent-annotations"></a>
  如果标识的描述文字超过一行，则第二行需要进行缩进。
<sup>[[链接](#indent-annotations)]</sup>

* <a name="sing-and-date-annotations"></a>
  将自己姓名的首字母以及当前日期附加到标识描述文字中
<sup>[[链接](#sing-and-date-annotations)]</sup>

    ```Clojure
    (defn some-fun
      []
      ;; FIXME: 这段代码在v1.2.3之后偶尔会崩溃。
      ;;        这可能和升级BarBazUtil有关。（xz 13-1-31）
      (baz))
    ```

* <a name="rare-eol-annotations"></a>
  对于功能非常明显，实在无需添加注释的情况，可以在行尾添加一个标识。
<sup>[[链接](#rare-eol-annotations)]</sup>

    ```Clojure
    (defn bar
      []
      (sleep 100)) ; OPTIMIZE
    ```

* <a name="todo"></a>
  使用 `TODO` 来表示需要后期添加的功能或特性。
<sup>[[链接](#todo)]</sup>

* <a name="fixme"></a>
   使用 `FIXME` 来表示需要修复的问题。
<sup>[[链接](#fixme)]</sup>

* <a name="optimize"></a>
  使用 `OPTIMIZE` 来表示会引起性能问题的代码，并需要修复。
<sup>[[链接](#optimize)]</sup>

* <a name="hack"></a>
  使用 `HACK` 来表示这段代码并不正规，需要在后期进行重构。
<sup>[[链接](#hack)]</sup>

* <a name="review"></a>
  使用 `REVIEW` 来表示需要进一步审查这段代码，如：`REVIEW: 你确定客户会正确地操作X吗？`
<sup>[[链接](#review)]</sup>

* <a name="document-annotations"></a>
  可以使用其它你认为合适的标识关键字，但记得一定要在项目的 `README` 文件中描述这些自定义的标识。
<sup>[[链接](#document-annotations)]</sup>

    
## <a name="existential"></a>惯用法

* <a name="be-functional"></a>
  使用函数式风格进行编程，避免改变变量的值。
<sup>[[链接](#be-functional)]</sup>

* <a name="be-consistent"></a>
  保持编码风格。
<sup>[[链接](#be-consistent)]</sup>

* <a name="common-sense"></a>
  用正常人的思维来思考。
<sup>[[链接](#common-sense)]</sup> 
    
    
 ## <a name="tooling"></a>工具

这里有一些由Clojure的社区创建的工具，可能会帮助你
在你努力写出地道的Clojure代码。

* [Slamhound](https://github.com/technomancy/slamhound)是一种能自动从你的现有的代码生成合适的 `ns` 声明。
* [kibit](https://github.com/jonase/kibit) 是一个用于Clojure的静态代码分析器。
  [core.logic](https://github.com/clojure/core.logic) 为代码搜索可能存在一个更惯用模式函数或宏。   
    


# 贡献

本文中的所有内容都还没有最后定型，我很希望能够和所有对Clojure代码规范感兴趣的同仁一起编写此文，从而形成一份对社区有益的文档。

你可以随时创建讨论话题，或发送合并申请。我在这里提前表示感谢。

You can also support the style guide with financial
contributions via [gittip](https://www.gittip.com/bbatsov).

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)
    
    
# 证书

![创作共用许可](http://i.creativecommons.org/l/by/3.0/88x31.png)
这项工作是根据
[知识共享署名3.0 本地化许可协议 ](http://creativecommons.org/licenses/by/3.0/deed.en_US)许可。

# 宣传

一份由社区驱动的代码规范如果得不到社区本身的支持和认同，那它就毫无意义了。微博转发这份指南，分享给你的朋友或同事。我们得到的每个注解、建议或意见都可以让这份指南变得更好一点。而我们想要拥有的是最好的指南，不是吗？

Cheers,<br/>
[Bozhidar](https://twitter.com/bbatsov)