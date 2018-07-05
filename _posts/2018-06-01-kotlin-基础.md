---
title: Kotlin 基础
date: 2018-06-01
categories:
  - Android
tags:
  - kotlin
---



## 语句

### in 关键字

> 判断一个对象是否在某一个区间内（比如array，集合之类的）

### when 表达式

> 取代 java 中的 `switch` 

### is 关键字

> 判断一个对象是否为一个类的实例，与 java 中的 `instanceof` 类似

- 可以取反 `!is`
- 做过类型判断后，对象会被系统自动转换成比对的类型，在判断代码块外部的该对象依然是原始类型的引用

```kotlin
    fun getStrLen(obj: Any) : Int? {
        if (obj is String) {
            // 做过类型判断后，obj 会被系统自动转换为 String 类型
            return obj.length
        }

        // 代码块外部的 obj 依然是 Any 类型的引用
        return null
    }
```

### let 关键字

- 非空执行

### 空值检测

- `println(files?.size)`  这句代码之后在 `files` 不为空时执行

## 函数

- 函数可以直接设置默认参数: `fun say(str: String = "hi")`

### 构造函数

- 分为主构造函数和次构造函数
- 如有 *注解* 需要加上关键字 `constructor`
- 在构造函数中声明的参数默认为 `public`，如果不希望其他类访问到这个变量可以使用 `private` 来修饰
- `init` 块：在主构造函数中不能有任何代码实现，如有此需求要放到 `init` 块中执行
- 如果一个非抽象类没有声明任何(主或次)构造函数，它会有一个生成的不带参数的主构造函数
- 构造函数默认是 public 的
- 如果你不希望你的类 有一个公有构造函数,你需要声明一个带有非默认可见性的空的主构造函数 `class myClass private constructor () {  }`
- 在 JVM 上,如果主构造函数的所有的参数都有默认值，编译器会生成一个额外的无参构造函数,它将使用默认值
- 次级构造函数需要用 `constructor` 声明。次级构造函数不能直接将参数转换为字段

### 变参函数

> 在 Kotlin 中，变长参数用 `vararg` 关键字表示

```java
    // java
    public boolean hasEmpty(String... strArray) { 
        //... 
    }

    // kotlin
    fun hasEmpty(vararg strArray: String?): Boolean { 
        // ...
    }
```

### 扩展函数 

> 给父类添加一个方法，这个方法将可以在它以及它所有的子类中使用

```kotlin
    fun Activity.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
        Toast.makeText(this, message, duration).show()
    }

    // 此后可以直接在 Activity 中调用
    toast("hello kotlin")

```

- <mark><b>注意，扩展方法是静态解析的，并不是真正给类添加了方法</b></mark>

```kotlin
    open class Animal {
    }

    class Dog: Animal() 

    object Main {
        fun Animal.bark() = "animal"
        fun Dog.bark() = "dog"

        fun Animal.printBark(anim: Animal) {
            println(anim.bark())
        }

        @JvmStatic fun main(args: Array<String>) {
            Animal().printBark(Dog()) // "animal"
        }
    }

```

### 嵌套函数

> Kotlin 的一个特性，在函数中在声明函数。与内部类有些类似，内部函数可以直接访问外部函数的局部变量、常量，而外部函数不能访问到内部函数。通常使用在 *会在某些条件下触发递归的方法内* 或者是 *不希望外部其他函数访问到的函数*，在一般情况下是不推荐使用嵌套函数的

```kotlin
    fun sample() {
        val str = "Hello!"

        fun say(count: Int = 10) {
            println(str)
            if (count > 0) {
                say(count - 1)
            }
        }
        say()
    }
```

### 高阶函数

> 当定义一个闭包作为参数的函数，称为高阶函数

### 内联函数

> 它可以大幅提升高阶函数的性能

### 闭包

1. 自执行闭包：在定义闭包的同时直接执行闭包，一般用于初始化上下文环境

    ```kontlin
    { x: Int, y: Int ->
        println("${x + y}")    
    }(1, 3)
    ```
    - 闭包不能有变长参数
    - 闭包虽然有强大的灵活性，可以省略很多临时变量和参数声明。但是滥用的话，可以会写出可读性非常差的代码

## 类

- Kotlin 的所有类在默认情况下都是 `final` 
- Kotlin 中，每个枚举常量都是一个对象
- `is` 判断一个对象是否是某个类的实例，`as` 用来强转
- 如果 smart cast(智能转换) 的对象是一个全局变量，这个变量可能在别的地方被改变赋值，所以必须手动判断与转换它的类型

```kotlin
    open class Animal {
    }
    class Dog: Animal {
        fun bark() = println("animal")
    }

    var animal: Animal? = null

    fun main(args: Array<String>) {

        if (animal is Dog) {
            // 并不满足执行的条件，所以不会执行以下语句
            (animal as Dog).bark()
        }
    }
```

### 单例类 - Singleton

```kotlin
    // 类被调用的时候才去初始化它的对象
    // 推荐写法
    class Single private constructor() {
        companion object {
            fun get(): Single {
                return Holder.instance
            }
        }

        private object Holder {
            val instance = Single()
        }
    }
```

### 动态代理

> Kotlin 原生支持动态代理


## 对象

### companion object

> 伴生对象

## Lambda

- Lambda 表达式最大的特点是可以作为参数传递

### 语法糖

- 当参数只有一个的时候，声明中可以不用显式声明参数，在使用参数时可以用 `it` 来替代那个唯一的参数
- 当有多个用不到的参数时，可以用下划线来替代参数名，但是如果已经使用下划线来省略参数时，是不能使用 `it` 来替代当前参数的
- Lambda 最后一条语句的执行结果表示这个 Lambda 的返回值

## 与 Java 的交互

- 关于 `Class` 的调用，在 M13 之前，Java 中的 `myJava.class` 对应 Kotlin 中的 `JavaClass<myJava>`，而 M13 之后写法已经变为 `myJava::class.java`

- Kotlin 中没有 `static` 关键字，如果在 Java 中想要通过雷鸣调用一个 Kotlin 的方法，就需要给这个方法加入 `@JvmStatic` 注解。否则就必须结合 `Companion` 来调用方法

## 集合

- 在 Kotlin 中，集合类一般不使用构造方法去初始化，而是使用同一的入口方法

### 操作符

> 本质是方法调用

- Kotlin 的操作符跟 RxJava 基本一致

#### 常用操作符

1. 下标操作符
- contains
- elementAt
- firstOrNull
- lastOrNull
- indexOf
- singleOrNull

2. 判断类
- any
- all
- none
- count
- reduce

3. 过滤类
- filter
- filterNot
- filterNotNull
- take

4. 转换类
- map
- mapIndexed
- mapNotNull
- flatMap
- groupBy

5. 排序类
- reversed
- sorted
- sortedBy
- sortedDescending





##  编码风格

- 不要给属性前面加前缀，比如 `m` 或者 下划线等