## Clojure学习笔记

#### LIsp语法
  clojure的语法源自于Lisp：有很多括号。
  clojure括号的两个目的：调用函数、构造列表
   括号是语言里所有表达式的容器，在编辑代码时可以像积木一样安排这些表达式，每一个表达式都是产生一致值的独立功能小世界。

#### Clojure与Java
  在所有Clojure程序中，默认加载Java的java.lang包中的，这样就可以直接引用String和Integer等。许多clojure数据结构实现Java接口，所以如果java程序库使用实现那些接口的对象，就可以接受Clojure的数据结构作为参数，
  点运算符（.）是java互操作的基础，clojure提供的语法糖衣是静态域和方法用一个前向斜杠访问，比如（Math/abs -3）。PI是一个静态域而不是方法，所以不需要通过调用括号返回值，但是abs是一个方法，所以必须用括号调用。
  要创建类的实例，可以使用new运算符或者点后缀，表示调用类构造程序。
```clojure
 （new Integer "42"）
  ;; Result: 42
 （Integer. "42"）
  ;; Result: 42
```

JAVA虽然具有编写共享可变状态的安全并发程序所需要的一切工具，但在实践中很难正确地写出这样的程序，Clojure的核心数据结构都是不可变的，在需要可变状态时，Clojure提供名为var（变量）、atom（原子）、ref（引用）和agent（代理）的并发数据结构。

#### Hello world
``` clojure
 user> (println "Hello,world")
 Hello,world
 => nil
```
  println函数不同寻常，他是一个有副作用的函数，它向标准输出打印一个字符串，然后返回nil。

#### 查找文档
1.doc
  宏doc可以用来搜索与任何其他函数与宏相关的文档。
``` clojure
 user> (doc +)
```
 函数参数规范中的&符号意为“与任何数量的可选参数”，这样的函数称为可变参数函数。
2.find-doc
  该函数接受一个字符串参数，这个参数可以是正则表达式。该函数按照名称或者相关文档与所提供模式的匹配寻找符合条件的函数或宏文档。
3.apropos
  该函数的工作方式与find-doc类似，但是只打印匹配搜索模式的函数名称。

####语法要点
 - 前缀表示法：函数调用时使用前缀表示法，尤其是使用数学函数（+、/等）的时候。Clojure不使用1+2这样的表示，而是使用（+1 2）。
 - 空格：和Ruby、Java不同，它不需要逗号来分割列表元素，如果使用的话clojure会将它们当成空格而忽略。
 - 注释：Clojure的单行注释用分号表示，要将一行文本变成注释，可以在行首加上一个或多个分号。Clojure还提供了一个宏comment用于多行注释。
``` clojure
  (comment
    (defn this-is-not-working [x y]
    (+ x y)))
  ;=>nil
```
 - 大小写敏感
 - 符号：符号前加单引号，clojure会把这个符号当成数据而不是代码来处理。

#### 数据结构
---
 - 列表：列表是基本集合数据结构。Clojure列表是单链表，这意味着很容易从列表的第一个元素遍历到最后一个元素，但是不可能从最后一个元素反向遍历到第一个元素，只能从列表的前面添加或删除元素。
    用**list函数**可以创建一个列表，**list？**函数可以测试列表类型：
``` clojure
  (list 1 2 3 4 5)
  ;=> (1 2 3 4 5)
  (list? *1)
  ;=> true
```
  **conj函数**是clojure中用于“为集合添加一个元素”的通用函数，它总是以可能的最快方式在一个集合中添加元素。
``` clojure
 (conj (list 1 2 3 4 5) 6)
 ;=>(6 1 2 3 4 5)
 (conj (list 1 2 3)4 5 6)
 ;=>(6 5 4 1 2 3)
 (conj(conj(conj(list 1 2 3) 4) 5) 6)
 ;=>(6 5 4 1 2 3)
```
 也可以把列表当成栈来对待，用**peek**返回表头，**pop**返回表尾：
``` clojure
 (peek (list 1 2 3))
 ;=> 1
 (pop (list 1 2 3))
 ;=> (2 3)
```
 **count函数**得到列表中的元素数量：
``` clojure
 (count(list))
 ;=>0
 (count (list 1 2 3 4))
 ;=>4
```

- 向量：向量与列表不同的地方在于向量用方括号表示，以数字作为索引。向量可以用vector函数创建，也可以用方括号表示法创建：
``` clojure
 (vector 10 20 30 40 50)
 ;=>[10 20 30 40 50]
 (def the-vector [10 20 30 40 50])
 ;=> #'user/the-vector
```
向量用数字方法索引，你可以快速随机访问向量中的元素，获取多元素向量中元素的函数有**get**和**nth**。
``` clojure
 (get the-vector 2)
 ;=> 30
 (nth the-vector 2)
 ;=> 30
 (get the-vector 10)
 ;=> nil
```
nth和get的差别在于，如果没有找到对应的值，nth抛出异常而get返回nil。
**assoc**是最常用修改向量的方法。
``` clojure
 (assoc the-vector 2 25)
 ;=> [10 20 25 40 50]     ;可以修改现有的索引
 (assoc the-vector 5 50) 
 ;=> [10 20 30 40 50 60]  ;可以添加到尾部
 (assoc the-vector 6 70)
 ;=>exception             ;不能超过尾部
```
**conj**函数也适用于向量，新元素出现在序列的最后，那是向量中最快速的插入位置：
``` clojure
 (conj [1 2 3 4 5] 6)
 ;=> [1 2 3 4 5 6]
```
**peek**和**pop**也适用于向量，peek查看空集合总是返回nil，pop弹出空集合元素总是抛出异常。

- 映射：一个映射就是一个键值对序列。键可以是任何类型的对象，用对应的键可以在映射中查到一个值。映射用花括号表示。也能用**hash-map**函数构建：
``` clojure
 (def the-map {:a 1 :b 2 :c 3})
 ;=> #'user/the-map
 (hash-map :a 1 :b 2 :c 3)
 ;=> {:a 1, :c 3, :b 2}
```
值的查找方式如下：
``` clojure
 (the-map :b)
 ;=> 2
```
clojure映射也是一个函数，关键字(:a和:b)也是函数，它们接受一个关联集合，在集合中查找自身：
``` clojure
  (:b the-map)
  ;=> 2
```
和所有clojure的数据结构一样，映射也是不可变的，有多个函数可以修改映射，最常用的是**assoc**和**dissoc**。
``` clojure
(def updated-map (assoc the-map :d 4))
;=> #'user/updated-map
updated-map
;=> {:d 4, :a 1, :b 2, :c 3}
(dissoc updated-map :a)
;=> {:b 2, :c 3, :d 4}
```
