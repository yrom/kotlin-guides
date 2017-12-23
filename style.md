---
layout: page
title: Style guide
site_nav_category_order: 200
is_site_nav_category: true
site_nav_category: style
---

本文承载了 Google 对于 Android 上关于 Kotlin 编程语言的编码准则的完整定义。一份 Kotlin 源文件，当且仅当它遵守本文的规则书写时，可被称为符合了 Google Android Style。

像别的编程语言风格指南一样，所涉及的不仅仅是美化代码格式的问题，还包括其它类型的约定或者编码标准。然而，本文主要关注的还是我们应普遍遵循的硬性规定，并尽量避免给出不明确的强制性建议（无论是人或者工具）。

_<a href="changelog.html">上次更新于: {{ site.changes.last.date | date: "%Y-%m-%d" }}</a>_

# 源文件

所有源文件必须用 UTF-8 编码。

## 命名

如果一个源文件只包含一个顶级类，这个文件名必须反应出类名且区分大小写的名称，并加上 `.kt` 扩展名。

如果包含多个顶级声明，请选择一个能够描述文件内容的名称，并用[PascalCase](https://en.wikipedia.org/wiki/PascalCase)格式(注：首字母大写，每个单词首字母大写，其余小写)表示，加上 `.kt` 扩展名。

```kotlin
// Foo.kt
class Foo { }

// Bar.kt
class Bar { }
fun Runnable.toBar(): Bar = // …

// Map.kt
fun <T, O> Set<T>.map(func: (T) -> O): List<O> = // …
fun <T, O> List<T>.map(func: (T) -> O): List<O> = // …
```

## 特殊字符

### 空白字符

除了行终止符外，**ASCII 水平空格字符 (0x20)** 是唯一允许出现在源文件的空白字符。这意为着：

 1. 字符串和字符字面量中所有其它空白字符都会被转义；
 2. **不要**用制表符(TAB)来缩进。

### 特殊转义字符

[特殊转义序列](https://kotlinlang.org/docs/reference/basic-types.html#characters) (`\b`, `\n`, `\r`, `\t`, `\'`, `\"`, `\\`, 和 `\$`) 中的字符可以直接用，而不需要用相应的 Unicode (如`\u000a`) 来转义

### 非 ASCII 字符

对于非 ASCII 字符，用实际的 Unicode 字符（如`∞`）或者相同意义的 Unicode 转义 （如`\u221e`）都可以。根据**较容易被阅读和理解**的原则来选择其中一种。Unicode 转义不建议用于可打印字符，并且强烈反对在字符串字面量和注释之外使用。

| **示例**                           | **说明**                                        |
|------------------------------------|------------------------------------------------|
| `val unitAbbrev = "μs"`            | 完美: 即便没有注释也一目了然                       |
| `val unitAbbrev = "\u03bcs" // μs` | 凑合: 可打印字符没有道理要用 Unicode 转义表示       |
| `val unitAbbrev = "\u03bcs"`       | 不好: 读者完全看不懂这是个啥                       |
| `return "\ufeff" + content`        | 还行: 转义不可打印字符，并且要加上必要注释           |


## 结构

一个 `.kt` 文件依次由下列内容组成:

1. Copyright / License 头 (可选)
2. 文件级注解
3. 包定义语句
4. Import 语句
5. 顶级声明（Top-level）

上述内容均用空行分隔。

### Copyright / License

如果有版权 `copyright` 或者授权 `license` 声明，必须放在顶部开始位置，用多行注释表示

```kotlin
/*
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
 ```

不要用 [KDoc 样式](https://kotlinlang.org/docs/reference/kotlin-doc.html) 或者单行注释.

```kotlin
/**
 * Copyright 2017 Google, Inc.
 *
 * ...
 */
```
```kotlin
// Copyright 2017 Google, Inc.
//
// ...
```

### 文件级注解

[使用目标点](https://kotlinlang.org/docs/reference/annotations.html#annotation-use-site-targets)为 'file' 的注解需要放到头部注释和包声明之间。

### 包声明语句

包声明不被行字数限制且不允许换行。

### Import 语句

导入（import）类、函数和属性的语句，按类型分组并按 ASCII 码表序分别排列在一起。

**不允许**使用通配符(*)。

跟包声明一样不受行字数限制，不换行。

### 顶级声明

一个 `.kt` 文件可以在顶级声明一个或者多个类型、函数、属性以及类型别名。

文件的内容应该聚焦在一个主题上，比如单个的公开（public）的类，或者一组在多个接收器类型上执行同样的操作的扩展函数。
不相关的声明要分开到各自的文件里，并应尽量的减少单个文件里的公开（public）声明。

对于文件内容的数量和排列顺序没有明确的限制。

阅读源代码通常是从上往下的，这意味着，在一般情况下，顺序上应该能反应出声明越靠上越能让他人对其理解得更为深入。
不同文件可能会按其内容选择不同的顺序书写。同样的，一个文件可能会包含100个属性，还有10个函数，甚至会有一个类。

重要的是，每个类应该有其**一定逻辑上的顺序**，维护者被问到时是可以解释其合理性的。例如，新增的函数不应只是习惯性的放到文件末尾，因为那样不过是迁就于“按日期时间顺序添加”的顺序，而不是一个合理的顺序。

### 类成员的顺序

类成员顺序跟顶级声明遵循同样的规则。

# 格式

## 大括号({})

`when` 的分支语句，以及 `if` 的语句没有 `else if`/`else` 分支且单行表示的，可以省略大括号。

```kotlin
if (string.isEmpty()) return

when (value) {
    0 -> return
    // …
}
```

任何 `if`、 `for`、 `when`的分支、`do` 以及 `while` 的语句，即使它们后面内容是空的或只有一行，都必须用大括号。

```kotlin
if (string.isEmpty())
    return  // WRONG!

if (string.isEmpty()) {
    return  // Okay
}
```

### 非空代码块（block）

非空代码块的大括号遵循 [Kernighan & Ritchie](https://en.wikipedia.org/wiki/K%26R) 风格 ("埃及括号 [Egyptian brackets](https://blog.codinghorror.com/content/images/uploads/2012/07/6a0120a85dcdae970b016768a17a2a970b-800wi.jpg)")：

* 左大括号前不换行
* 左大括号后换行
* 右大括号前换行
* 右大括号后换行，_仅_ 适用于以括号结束的语句或者函数、构造器、和类的结束。例如，大括号后跟着 `else` 或者逗号是不需要换行的。

```kotlin
return Runnable {
    while (condition()) {
        foo()
    }
}

return object : MyClass() {
    override fun foo() {
        if (condition()) {
            try {
                something()
            } catch (e: ProblemException) {
                recover()
            }
        } else if (otherCondition()) {
            somethingElse()
        } else {
            lastThing()
        }
    }
}
```

针对[枚举类](#枚举类)的特别情况在后文有提到。

### 空代码块（block）

空代码块需遵循 K&R 风格.

```kotlin
try {
    doSomething()
} catch (e: Exception) {} // 错了!
```
```kotlin
try {
    doSomething()
} catch (e: Exception) {
} // 可以
```

### 表达式

`if`/`else` 条件语句用作表达式时可以在只有一行内容的时候省略大括号。

```kotlin
val value = if (string.isEmpty()) 0 else 1  // Okay
```
```kotlin
val value = if (string.isEmpty())  // WRONG!
                0
            else
                1
```
```kotlin
val value = if (string.isEmpty()) { // Okay
    0
} else {
    1
}
```


## 缩进

每次书写一个新的代码块或者块状构造时，缩进增加四个空格。代码块结束时，缩进回到前一个层级。
缩进层级可以应用于整个代码块中的代码和注释。

## 一行一句

每个语句都应以换行结尾，无需用分号。

## 换行

一行代码限制在100个字符以内。除了以下面列举的几种情况外，任何超过限制的行都必须按下面解释进行换行。

例外：
* 不可以换行的（如KDoc中的一个很长的URL）
* package 和 import 的语句
* 在注释中的命令行（可能被直接粘贴进shell中执行）

### 换行位置

换行的根本准则是：优先在较高的句法层面（syntactic level）上换行。

 1. _非赋值_ 运算符处换行时，在符号前换行。
    * 这也适用于"类运算符"的符号：
      * 点分隔符 (`.`)
      * 两个冒号的成员引用 (`::`)
 2. _赋值_ 运算符处换行时，在符号后换行。
 3. 在方法或者构造器的左括号 (`(`) 之后换行
 4. 在跟在标记的逗号 (`,`) 之后换行
 5. 在跟在 lambda 的参数后面的箭头 (`->`) 之后换行

注意：换行的主要目的是让代码清晰，而不是过分追求更少的代码行数。

### 连续缩进

当换行后，每一行（每个连续行），相对原行至少要有 +8 个字符的缩进。

当有多个连续的换行后，缩进可能根据需要会增加到 +8 以上。
通常来说，当且仅当两个连续的缩进行在语法上为并行的元素开头，使用相同的缩进层级。

### 函数

当函数的签名一行容不下，在每个参数声明处都换行。这种格式下定义的参数应使用一个缩进（+4)。
闭合括号(`)`) 要和返回类型放在同一行，且不用额外的缩进。

```kotlin
fun <T> Iterable<T>.joinToString(
    separator: CharSequence = ", ",
    prefix: CharSequence = "",
    postfix: CharSequence = ""
): String {
    // …
}
```

#### 表达式函数

当一个函数只包含了一个表达式语句时，可以将其用[表达式函数](https://kotlinlang.org/docs/reference/functions.html#single-expression-functions)来表示。

```kotlin
override fun toString(): String {
    return "Hey"
}
```
```kotlin
override fun toString(): String = "Hey"
```

表达式函数不应该被分成两行，如果一个表达式函数过长，需要被分行书写，这时应该用普通的函数体，`return`声明，符合换行规则的普通表达式来替代。

### 属性

当一个属性的初始化代码一行放不下时，可以在等号`=`之后换行，并使用连续缩进。

```kotlin
private val defaultCharset: Charset? =
        EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

属性的`set`和`get`函数应该放到它们各自的行，并用一个普通缩进（+4）。它们跟别的普通函数遵循一样的格式。

```kotlin
var directory: File? = null
    set(value) {
        // …
    }
```

只读属性若在一行内可以用更简短的语法表示。

```kotlin
val defaultExtension: String get() = "kt"
```

## 空行

### 纵向

空行出现在：

 1. 在类的连续成员之间：属性、构造器、函数、嵌套类等等
     * **例外**: 两个连续的（没有其它代码）属性之间空行不是必须的。空行可以用于将属性按逻辑分组，还有将属性和其幕后属性（如果存在）作关联用。
     * **例外**: 枚举常量之间的空白行下面有提到。
2. 在语句之间，按需将代码组织成逻辑上的小节。
3. _可选项_：在函数的第一个语句之前，在类的第一个成员之前，或者类的最后一个成员之后（不鼓励也不反对）。
4. 按本文其它章节的要求（如[“结构”](#结构)）

允许多个连续的空行，但不鼓励也不强求。

### 横向

除了语言和其它样式规范的要求之外，不包括字面值、注释和KDoc，单个 ASCII 的空格仅可出现在下列位置：

1. 隔开保留词（如`if`、`for`及`catch`）和跟在其后的开括号（`(`）。
    ```kotlin
    // WRONG!
    for(i in 0..1) {
    }
    ```
    ```kotlin
    // Okay
    for (i in 0..1) {
    }
    ```
    
2. 隔开保留词（如`else`、`catch`）和跟在其后的右大括号（`}`）。
    ```kotlin
    // WRONG!
    }else {
    }
    ```
    ```kotlin
    // Okay
    } else {
    }
    ```
    
3. 在开大括号（`{`）之前。
    ```kotlin
    // WRONG!
    if (list.isEmpty()){
    }
    ```
    ```kotlin
    // Okay
    if (list.isEmpty()) {
    }
    ```
    
4. 在二元运算符两边。
    ```kotlin
    // WRONG!
    val two = 1+1
    ```
    ```kotlin
    // Okay
    val two = 1 + 1
    ```
    
    这个规则也适应下面的“类运算符”符号：

    * lambda表达式的箭头（`->`）
    
        ```kotlin
        // WRONG!
        ints.map { value->value.toString() }
        ```
        ```kotlin
        // Okay
        ints.map { value -> value.toString() }
        ```

    但不适用于：

    * 成员引用的双冒号（`::`）

        ```kotlin
        // WRONG!
        val toString = Any :: toString
        ```
        ```kotlin
        // Okay
        val toString = Any::toString
        ```

    * 点号分隔符（`.`）
    
        ```kotlin
        // WRONG
        it . toString()
        ```
        ```kotlin
        // Okay
        it.toString()
        ```
    *  区间运算符 (`..`).

        ```kotlin
        // WRONG
        for (i in 1 .. 4) print(i)
        ```
        ```kotlin
        // Okay
        for (i in 1..4) print(i)
        ```
    
5. 冒号（`:`）之前，仅限于类声明中指定基类或接口的冒号，或者在[泛型约束](https://kotlinlang.org/docs/reference/generics.html#generic-constraints)的where从句中的冒号。

    ```kotlin
    // WRONG!
    class Foo: Runnable
    ```
    ```kotlin
    // Okay
    class Foo : Runnable
    ```
    
    ```kotlin
    // WRONG
    fun <T: Comparable> max(a: T, b: T)
    ```
    ```kotlin
    // Okay
    fun <T : Comparable> max(a: T, b: T)
    ```

    ```kotlin
    // WRONG
    fun <T> max(a: T, b: T) where T: Comparable<T>
    ```
    ```kotlin
    // Okay
    fun <T> max(a: T, b: T) where T : Comparable<T>
     ```

6. 逗号（`,`）和冒号（`:`）之后。

    ```kotlin
    // WRONG!
    val oneAndTwo = listOf(1,2)
    ```
    ```kotlin
    // Okay
    val oneAndTwo = listOf(1, 2)
    ```

    ```kotlin
    // WRONG!
    class Foo :Runnable
    ```
    ```kotlin
    // Okay
    class Foo : Runnable
    ```
  
7. 在行尾注释双斜杠（`//`）的两边。这里允许有多个空格，但不强制

    ```kotlin
    // WRONG!
    var debugging = false//disabled by default
    ```
    ```kotlin
    // Okay
    var debugging = false // disabled by default
    ```

本规则不负责解释行首/尾所要求或禁止的额外空格，它仅用于行内。

## 特定的结构体

### 枚举类

无函数、无文档的枚举类，可以选择用一行书写。

```kotlin
enum class Answer { YES, NO, MAYBE }
```

枚举中的常量若是在不同的行，它们之间不需要空行，但定义了代码主体的除外。

```kotlin
enum class Answer {
    YES,
    NO,

    MAYBE {
        override fun toString() = """¯\_(ツ)_/¯"""
    }
}
```

枚举类也属于类，别的对于类的格式要求也同样适用于它。

### 注解

成员和类的注解，直接在被标注的结构之前分行书写。

```kotlin
@Retention(SOURCE)
@Target(FUNCTION, PROPERTY_SETTER, FIELD)
annotation class Global
```

多个无参注解可以一并放到一行。

```kotlin
@JvmField @Volatile
var disposable: Disposable? = null
```

只有一个无参注解，可以把它和声明放在同一行。

```kotlin
@Volatile var disposable: Disposable? = null

@Test fun selectAll() {
    // …
}
```

### 隐式返回/属性类型

若一个表达式函数或者属性的初始化代码，是标量值（如一个String），或者它们的返回值类型可以从后面的代码中推断出来，
那么类型可以省略。

```kotlin
override fun toString(): String = "Hey"
// becomes
override fun toString() = "Hey"
```

```kotlin
private val ICON: Icon = IconLoader.getIcon("/icons/kotlin.png")
// becomes
private val ICON = IconLoader.getIcon("/icons/kotlin.png")
```

但在编写库时，公开的 API 应保留其显式类型声明。

# 命名

标识符只能使用 ASCII 字母和数字，以及下面少数情况中的下划线。亦即每个合法的标识符都可用正则表达式`\w+`匹配。

有特殊前缀和后缀的，如示例中所见到的`name_`、`mName`、`s_name` 和 `kName`，只在[幕后属性](#幕后属性)中使用。

## 包名

包名应该全部为小写，连续单词直接拼在一起（不用下划线）。

```kotlin
// Okay
package com.example.deepspace
// WRONG!
package com.example.deepSpace
// WRONG!
package com.example.deep_space
```

## 类名

类名用 [PascalCase](https://en.wikipedia.org/wiki/PascalCase)格式书写，且通常是名词及名词短语，如 `Character`、`ImmutableList`。
接口名也可以用名词及名词短语（如`List`），也可以用形容词及形容词短语（如`Readable`）。

测试类用要测试的类的名字作开头，并以 `Test` 结尾，如`HashTest`、`HashIntegrationTest`。

## 函数名

函数名使用驼峰法命名，通常使用动词及动词短语，如`sendMessage`、`stop`。

测试函数名称中允许用下划线分隔其逻辑元素。

```kotlin
@Test fun pop_emptyStack() {
    // …
}
```

## 常量名

常量名用蛇形大小写（UPPER_SNAKE_CASE）格式：所有字母大写、单词之间用下划线隔开。但究竟什么是常量？

常量是那些没有自定义`getter`的`val`属性，其内容完全不可变，其函数没有可察觉的[副作用](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29)（译者注：不会对函数外的任何东西作修改）。这包括不可变类型、不可变类型的不可变集合，以及标记为`cost`的标量和字符串。如果一个实例的任何可被观察的状态是可改变的，它就不是一个常量。仅仅在意图上不对实例作改动是不够的。

```kotlin
const val NUMBER = 5
val NAMES = listOf("Alice", "Bob")
val AGES = mapOf("Alice" to 35, "Bob" to 32)
val COMMA_JOINER = Joiner.on(',') // Joiner 是不可变的
val EMPTY_ARRAY = arrayOf<SomeMutableType>()
```

这些名称通常是名词或名词短语。

常量只能定义在在一个对象（`object`）内或者作为一个顶级声明。否则，即使满足上述对常量的要求，但定义在类（`class`）中的必须使用非常量的名称。

标量值类型（scalar values）的常量，必须用[`const`修饰符](http://kotlinlang.org/docs/reference/properties.html#compile-time-constants)。

## 非常量命名

非常量的名字可采用驼峰写法。适用于实例的属性、局部属性和参数名。

```kotlin
val variable = "var"
val nonConstScalar = "non-const"
val mutableCollection: MutableSet<String> = HashSet()
val mutableElements = listOf(mutableInstance)
val mutableValues = mapOf("Alice" to mutableInstance, "Bob" to mutableInstance2)
val logger = Logger.getLogger(MyClass::class.java.name)
val nonEmptyArray = arrayOf("these", "can", "change")
```

这些名字通常是名词或名词短语。


### 幕后属性

当需要用到[幕后属性](https://kotlinlang.org/docs/reference/properties.html#backing-properties)时，它的名称，除了有个下划线当前缀外，与其实际属性应该完全匹配。

```kotlin
private var _table: Map<String, Int>? = null

val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap()
        }
        return _table ?: throw AssertionError()
    }
```


## 类型变量的名称

类型变量（注：用于泛型）按下面两种风格之一命名：

 1. 一个大写字母，可视情况跟个数字（如：`E`，`T`，`X`，`T2`）
 2. 用类型的名字加大写字母`T`（如：RequestT，FooBarT）

## 关于驼峰写法

有时候，会有不止一种合理的方式将英文短语转成驼峰风格，如缩略词或者类似于 "IPv6"、"iOS"等特别的结构。为了提高可预见性，可遵循下列方案。

从名字的原始形式开始：

 1. 将短语转为纯 ASCII 表示，并删除所有撇号。举个例子，"Müller's algorithm" 可转成 "Muellers algorithm"。

 2. 将结果再分成单词，用空格和剩下的标点符号（一般是连接字符）分割。

    * _建议_: 如果一个单词已经采用了驼峰写法，则按其组成部分分开（如 "AdWords" 转换成 "ad words"）。注意，像"iOS"这样的单词 _本身_ 并不是真正的驼峰写法，它违背了所有惯例，因此这个建议对它不适用。

 3. 将所有字符都小写（包括缩略词），然后只将下面单词的首字符大写：
 
    * Pascal写法中单词，或者

    * 驼峰写法中的单词（第一个除外）

4. 最后把所有的词组合成一个识别符。

需要注意的是，原始词组的外表几乎完全被忽略了。

| **原始形式**            | **Correct**                             | **Incorrect**       |
|------------------------|-----------------------------------------|---------------------|
| "XML Http Request"     | `XmlHttpRequest`                        | `XMLHTTPRequest`    |
| "new customer ID"      | `newCustomerId`                         | `newCustomerID`     |
| "inner stopwatch"      | `innerStopwatch`                        | `innerStopWatch`    |
| "supports IPv6 on iOS" | `supportsIpv6OnIos`                     | `supportsIPv6OnIOS` |
| "YouTube importer"     | `YouTubeImporter`<br>`YoutubeImporter`* |                     |

**注意**：有些词组的连字符在英语中是可有可无的，例如，"nonempty"和"non-empty"都是可以的，所以方法名`checkNonempty`和`checkNonEmpty`同样都可以。

# 文档

## 格式

如下是基本的 KDoc 文档:

```kotlin
/**
 * KDoc 的多行文本可以书写到这里，
 * 普通的换行…
 */
fun method(arg: String) {
    // …
}
```

或者像这样的单行:

```kotlin
/** 一个特别短的 KDoc。 */
```

基本样式在任何情况下都是可以用的。单行样式可以用来替换一行就能整个放下的 KDoc（包括注释标记），需要注意这仅适用于没有块标签（如`@return`）的 KDoc。

### 段落

空行，即只有对齐用的前置星号(`*`)的行，可以出现在段落间，也可以出现在块标签组（如果有的话）之前

### 块标签

所有标准的“块标签”都要按`@constructor`、`@receiver`、`@param`、`@property`、`@return`、`@throws`、`@see`的顺序排列，而且标签的描述不能留白。当块标签一行放不下时，换行后的连续行从`@`的位置缩进8个空格。

## 摘要片段

每个 KDoc 块都要以一个简短的摘要片段开头。这个片段非常重要：它是文档文本的唯一可以出现在某些上下文（例如类和方法索引）中的部分。

一个片段是名词短语或动词短语，而非一个完整的句子。它不以“`某某是一个……`” 或者“`这个方法返回……`”开头，也不必是一个完整的祈使句，如“`Save the record.`”。虽然，片段用大写字母开头又有标点符号（指用英文书写的片段），看起来像是一个完整的句子。


## 用法

至少在每个 `public` 的类型，以及每个`public` 或 `protected` 的成员上都要有 KDoc，下面的例子除外。

### 例外：自明（self-explanatory）函数

KDoc 对于“简单、明了”的函数（如`getFoo`）和属性（如`foo`）不是必需的，实在没什么值得说的时候就用“Returns the foo”吧。

举这个例子来说明其实不是很恰当，因为这样可能会省略掉某些读者正需要了解的相关信息。例如，函数名为`getCanonicalName`，或者属性名为`canonicalName`，如果某些读者可能不知道术语"canonical name"的意思的话，就不要省略它的文档（尽管术语解释起来可能只是一句`/** Returns the canonical name. */`）。

### 例外：覆写（override）

那些覆写自父类的方法并不是都会有 KDoc。

