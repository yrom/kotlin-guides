---
layout: page
title: Interop guide
site_nav_category: interop
is_site_nav_category: true
site_nav_category_order: 300
---

一套用于Java 和 Kotlin 编写公共 API 的规范，旨在两者混编互调时可遵循同一种习惯写法。

_<a href="changelog.html">上次更新于: {{ site.changes.last.date | date: "%Y-%m-%d" }}</a>_


# Java (Kotlin 调用)

## 不用Kotlin的“硬性关键字”

不要用 Kotlin 所声明的[硬性关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#hard-keywords)来做方法或字段的名称，因为从 Kotlin 调用这些时要使用反引号（`）。但[软性关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#soft-keywords)，[修饰符关键字](https://kotlinlang.org/docs/reference/keyword-reference.html#modifier-keywords)，以及[特殊标识符](https://kotlinlang.org/docs/reference/keyword-reference.html#special-identifiers)除外。

如， Mockito 的 `when` 方法在 Kotlin 中使用时需加上反引号：

```kotlin
val callable = Mockito.mock(Callable::class.java)
Mockito.`when`(callable.call()).thenReturn(/* … */)
```


## 参数中的 Lambda 后置

符合[SAM(单抽象方法)转换](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions)规则的参数类型应该放在最后。

举个例子， RxJava 2 中 `Flowable.create()` 方法签名被定义为：

```java
public static <T> Flowable<T> create(
    FlowableOnSubscribe<T> source,
    BackpressureStrategy mode) { /* … */ }
```

因为 `FlowableOnSubscribe` 符合 SAM 转换的规则，在 Kotlin 中调用此方法的函数看起来会是：

```kotlin
Flowable.create({ /* … */ }, BackpressureStrategy.LATEST)
```

如果方法签名的两个参数倒过来，即符合了后置 Lambda 的语法(trailing-lambda)：

```kotlin
Flowable.create(BackpressureStrategy.LATEST) { /* … */ }
```


## 属性(Property)的前缀

为了让某个方法在 Kotlin 中以属性(Property)形式使用， 应该严格遵循 "bean" 样式的前缀。

获取属性的方法须用 'get' 做前缀，对于返回值为`boolean`类型的方法则可以用 'is'。

```java
public final class User {
  public String getName() { /* … */ }
  public boolean isActive() { /* … */ }
}
```
```kotlin
val name = user.name // 调用 user.getName()
val active = user.active // 调用 user.isActive()
```

而设置属性值的方法则须用 'set' 前缀。

```java
public final class User {
  public String getName() { /* … */ }
  public void setName(String name) { /* … */ }
}
```
```kotlin
user.name = "Bob" // 调用 user.setName(String)
```

如果你想让方法被当作属性，不要使用不标准的前缀，如 'has'/'set' 或者非 'get' 前缀。
不标准前缀的方法仍然是可以被当作函数使用的，但具体需要视方法的行为而定。


## 操作符重载

要特别注意 Kotlin 那些特殊调用点语法中的方法的名称（比如[操作符重载](https://kotlinlang.org/docs/reference/operator-overloading.html)），应确保这些方法名称与缩短的语法一起使用都有效果。

```java
public final class IntBox {
  private final int value;
  public IntBox(int value) {
    this.value = value;
  }
  public IntBox plus(IntBox other) {
    return new IntBox(value + other.value);
  }
}
```
```kotlin
val one = IntBox(1)
val two = IntBox(2)
val three = one + two // 调用 one.plus(two)
```


## 可空 (Nullability）注解

公开 API 中的每一个非基本数据类型的参数、返回值和字段的类型都需要标上可空注解。没有标上注解的类型都会被视为可空可不空的[“平台类型”](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)

JSR 305 包中的注解可以用来设置合理的默认值，但目前不鼓励使用。 They require an opt-in flag to be honored by the compiler and conflict with Java 9's module system.


# Kotlin (Java 调用)

## 文件名

一个文件中有顶级函数或者属性时，*都要*用标记 `@file:JvmName("Foo")` 来给它们提供一个不错的名称。

默认情况下，文件`Foo.kt`中顶级成员会在以`FooKt`的名称为结尾的类中，这个类一点也不吸引人，而且暴露了语言的实现细节。

可以考虑加上`@file:JvmMultifileClass`注解，把多个文件里的顶级成员合并到一个类中。

## Lambda 参数

[函数类型](https://kotlinlang.org/docs/reference/lambdas.html#function-types)如要在Java中调用，应避免返回 `Unit` 类型。因为如果这样做的话，Java 中必须要明确使用返回语句 `return Unit.INSTANCE;`，这不符合 Java 语言习惯。

```kotlin
fun sayHi(callback: (String) -> Unit) = /* … */
```
```kotlin
// Kotlin 中调用:
greeter.sayHi { Log.d("Greeting", "Hello, $it!") }
```
```java
// Java 中调用:
greeter.sayHi(name -> {
    Log.d("Greeting", "Hello, " + name + "!");
    return Unit.INSTANCE;
});
```

同时这个语法也不允许提供一个方便其它类型实现的有语义名称的类型。

在 Kotlin 中为 Lambda 类型定义一个有明确名称的单抽象方法（SAM）的接口可以避免 Java 中出现这个问题，但会导致 Kotlin 中不能用 Lambda 语法。

```kotlin
interface GreeterCallback {
    fun greetName(name: String): Unit
}

fun sayHi(callback: GreeterCallback) = /* … */
```
```kotlin
// Kotlin 调用:
greeter.sayHi(object : GreeterCallback {
    override fun greetName(name: String) {
        Log.d("Greeting", "Hello, $name!")
    }
})
```
```java
// Java 调用:
greeter.sayHi(name -> Log.d("Greeting", "Hello, " + name + "!"))
```

在 Java 中定义一个有名称的 SAM 接口，允许使用 Kotlin 中稍微低一些版本的 Lambda 语法，但这个语法必须明确指定接口类型。

```java
// 定义在 Java 中:
interface GreeterCallback {
    void greetName(String name);
}
```
```kotlin
fun sayHi(greeter: GreeterCallback) = /* … */
```
```kotlin
// Kotlin 调用:
greeter.sayHi(GreeterCallback { Log.d("Greeting", "Hello, $it!") })
```
```java
// Java 调用:
greeter.sayHi(name -> Log.d("Greeter", "Hello, " + name + "!"));
```

目前为止，还没有办法能够定义一个参数类型让 Java 和 Kotlin 都可以用 Lambda 表示，使得两个语言都遵循一种习惯用法。当前的建议还是偏向用“函数类型”，尽管当返回类型是 `Unit` 时会降低在 Java 中的使用体验。

_注意: 这条建议未来可能会有所改变，详见链接[KT-7770](https://youtrack.jetbrains.com/issue/KT-7770) 和 [KT-21018](https://youtrack.jetbrains.com/issue/KT-21018)。_


## 避免在泛型中使用 `Nothing` 

泛型参数为 `Nothing` 的类型会以原始类型(raw types)的形式暴露给 Java， 然而在 Java 中非常少使用原始类型，所以应当避免这种用法。

## 文档中的异常

函数抛出的检查异常（checked exception）必须在文档中用`@Throws`声明。运行时异常（Runtime exception）则必须声明在 KDoc 中。

特别注意那些被函数所委托的API，因为它们可能抛出 Kotlin 中默认允许“传播”的检查异常。


## 保护性拷贝

当从公共 API 返回共享的或者无主的只读集合时，将它们包在一个不可被修改的容器中或者执行保护性的拷贝（defensive copy）。尽管 Kotlin 会强制使用它们的只读属性，但在 Java 中并没有这种强制性。一个长时间存活的集合，在返回时没有经过容器再次包装或者进行保护性的拷贝，而是返回它的直接引用，会导致它的不可变性被篡改。

## 伴生函数

伴生对象（`companion object`）中的公开函数必须标记上 `@JvmStatic` ，使得它得以静态方法的形式暴露出来。

如果没有用这个注解的话，这些函数会以实例方法的形式存在于静态字段 `Companion` 的实例对象中。

_不正确: 没有注解_

```kotlin
class KotlinClass {
    companion object {
        fun doWork() {
            /* … */
        }
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        KotlinClass.Companion.doWork();
    }
}
```

_正确: 标记 `@JvmStatic` _


```kotlin
class KotlinClass {
    companion object {
        @JvmStatic fun doWork() {
            /* … */
        }
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        KotlinClass.doWork();
    }
}
```


## 伴生常量

公开的非常量属性，在伴生对象中却是[有效常量](TODO)的必须标记 `@JvmField` 使得它能以静态字段形式暴露出来。

没有这个注解，这些属性会以奇怪名称的'getter'的形式可见于静态字段 `Companion` 的实例对象中。用注解 `@JvmStatic` 替换 `@JvmField` 可以将这些奇怪名称的 'getter' 改为静态方法，但仍然还是不正确的。


_不正确: 没有注解_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.Companion.getBIG_INTEGER_ONE());
    }
}
```

_不正确: 用了注解 `@JvmStatic`_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        @JvmStatic val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.getBIG_INTEGER_ONE());
    }
}
```

_正确: 用注解 `@JvmField`_

```kotlin
class KotlinClass {
    companion object {
        const val INTEGER_ONE = 1
        @JvmField val BIG_INTEGER_ONE = BigInteger.ONE
    }
}
```
```java
public final class JavaClass {
    public static void main(String... args) {
        System.out.println(KotlinClass.INTEGER_ONE);
        System.out.println(KotlinClass.BIG_INTEGER_ONE);
    }
}
```

## 语言习惯性命名
 
Kotlin 比起 Java 有着不同的调用约定，这会改变你对函数的命名。使用 `@JvmName` 可以设计出符合两个语言各自约定俗成，或者能与它们的标准库相匹配的命名。

这通常会发生在扩展函数或者扩展属性中，因为接收器类型的位置是不一样的。

```kotlin
sealed class Optional<T : Any>
data class Some<T : Any>(val value: T): Optional<T>()
object None : Optional<Nothing>()

@JvmName("ofNullable")
fun <T> T?.asOptional() = if (this == null) None else Some(this)
```
```kotlin
// FROM KOTLIN:
fun main(vararg args: String) {
    val nullableString: String? = "foo"
    val optionalString = nullableString.asOptional()
}
```
```java
// FROM JAVA:
public static void main(String... args) {
    String nullableString = "Foo";
    Optional<String> optionalString =
          Optionals.ofNullable(nullableString);
}
```

## 重载函数的参数默认值

有默认值参数的函数必须用 `@JvmOverloads` 标记，否则调用这个函数时任何的默认值都无效。

当使用 `@JvmOverloads` 时，检查生成的方法以确保它们切实有效。如果无效，则按照下面列举的方式重构直到达到要求：

 1. 更改参数顺序，那些有默认值的优先放到后面。
 2. 把默认值放到手写的重载函数中。

_不正确: 没用 `@JvmOverloads`_

```kotlin
class Greeting {
    fun sayHello(prefix: String = "Mr.", name: String) {
        println("Hello, $prefix $name")
    }
}
```
```java
public class JavaClass {
    public static void main(String... args) {
        Greeting greeting = new Greeting();
        greeting.sayHello("Mr.", "Bob");
    }
}
```

_正确: 使用 `@JvmOverloads`._

```kotlin
class Greeting {
    @JvmOverloads
    fun sayHello(prefix: String = "Mr.", name: String) {
        println("Hello, $prefix $name")
    }
}
```
```java
public class JavaClass {
    public static void main(String... args) {
        Greeting greeting = new Greeting();
        greeting.sayHello("Bob");
    }
}
```
