# 表达式语言

> 由 `scripts/build-refs.mjs` 从 `infra-docs` 生成，请勿手工编辑；改归类请改 `data/topics.json`。

SpEL 表达式解析、求值上下文、运算符与集合投影。

覆盖文档：23 页（含示例 21 页）。原文位于仓库 `infra-docs/`，下方每个页面标注了路径。

## 页面索引

- SpEL 表达式 —— `core/expressions`
- **Bean 定义中的表达式** —— `core/expressions/beandef`
- **评估** —— `core/expressions/evaluation`
- **示例中使用的类** —— `core/expressions/example-classes`
- 语言参考 —— `core/expressions/language-ref`
- **数组构造** —— `core/expressions/language-ref/array-construction`
- **Bean 引用** —— `core/expressions/language-ref/bean-references`
- **集合投影** —— `core/expressions/language-ref/collection-projection`
- **集合选择** —— `core/expressions/language-ref/collection-selection`
- **构造函数** —— `core/expressions/language-ref/constructors`
- **函数** —— `core/expressions/language-ref/functions`
- **内联列表** —— `core/expressions/language-ref/inline-lists`
- **内联 Map** —— `core/expressions/language-ref/inline-maps`
- **字面量表达式** —— `core/expressions/language-ref/literal`
- **方法** —— `core/expressions/language-ref/methods`
- **Elvis 运算符** —— `core/expressions/language-ref/operator-elvis`
- **安全导航运算符** —— `core/expressions/language-ref/operator-safe-navigation`
- **三元运算符 (If-Then-Else)** —— `core/expressions/language-ref/operator-ternary`
- **运算符** —— `core/expressions/language-ref/operators`
- **属性、数组、列表、Map 和索引器** —— `core/expressions/language-ref/properties-arrays`
- **表达式模板化** —— `core/expressions/language-ref/templating`
- **类型** —— `core/expressions/language-ref/types`
- **变量** —— `core/expressions/language-ref/variables`

## SpEL 表达式

> 原文：[`core/expressions`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions.adoc)

### 概述

我们直接使用了 Infra EL SpEL 是一种强大的表达语言，支持在运行时查询和操作对象图。该语言的语法类似于 Jakarta Expression Language， 但提供了额外的功能，尤其是方法调用和基本的字符串模板功能。 虽然还有其他 Java 表达式语言（例如 OGNL、MVEL 和 JBoss EL），Infra 表达式语言的目标是为 Infra 社区提供一个可在 Infra 产品组合中统一使用、并得到良好支持的表达式语言。 其语言特性由 Infra 产品组合中各项目的需求驱动，其中也包括在 Infra Tools for Eclipse 中提供代码补全支持的工具需求。尽管如此，SpEL 仍基于一个与技术无关的 API， 这使得在需要时可以集成其他表达式语言实现。 虽然 SpEL 作为 Infra 产品组合中的表达式求值基础，但它并不直接依赖 Infra，也可以独立使用。 

## Bean 定义中的表达式

> 原文：[`core/expressions/beandef`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/beandef.adoc)

### 概述

你可以将 SpEL 表达式与配置元数据一起使用，以定义 bean 实例。在这两种情况下， 定义表达式的语法形式为 #{ <expression string> }。 应用程序上下文中的所有 bean 都可以作为预定义变量使用，变量名为其常见的 bean 名称。 这包括标准上下文 bean，如 environment（类型为 infra.core.env.Environment） 以及用于访问运行时环境的 systemProperties 和 systemEnvironment（类型为 Map<String, Object>）。 要指定默认值，可以将 @Value 注解放在字段、方法以及方法或构造函数参数上（或 XML 等效项）。 下例设置字段的默认值：

```java
public class FieldValueTestBean {

  @Value("#{ systemProperties['user.region'] }")
  private String defaultLocale;

  public void setDefaultLocale(String defaultLocale) {
    this.defaultLocale = defaultLocale;
  }

  public String getDefaultLocale() {
    return this.defaultLocale;
  }
}
```

## 评估

> 原文：[`core/expressions/evaluation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/evaluation.adoc)

### 概述

本节介绍 SpEL 接口及其表达式语言的编程式使用。 完整的语言参考可以在 语言参考 中找到。 以下代码演示了如何使用 SpEL API 评估字面字符串表达式 Hello World。

```java
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'"); // <1>
String message = (String) exp.getValue();
```

##### 类型转换

默认情况下，SpEL 使用 Infra core 中可用的转换服务 (infra.core.conversion.ConversionService)。此转换服务附带了许多用于常见转换的内置转换器， 但也完全可扩展，以便你可以在类型之间添加自定义转换。此外，它还具有泛型感知能力。 这意味着，当你在表达式中使用泛型类型时，SpEL 会尝试转换以保持其遇到的任何对象的类型正确性。 NOTE: SimpleEvaluationContext 默认不支持反射类型转换，可以通过构建器的 withConverter(...) 增加转换器。 参见 <<expressions-evaluation-context-security,安全注意事项>>。 这在实践中意味着什么？假设使用 setValue() 进行赋值来设置 List 属性。 该属性的类型实际上是 List<Boolean>。SpEL 认识到列

```java
class Simple {
  public List<Boolean> booleanList = new ArrayList<>();
}

Simple simple = new Simple();
simple.booleanList.add(true);

EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// "false" 作为 String 传入。SpEL 和转换服务
// 将识别出它需要是 Boolean 并相应地进行转换。
parser.parseExpression("booleanList[0]").setValue(context, simple, "false");

// b 是 false
Boolean b = simple.booleanList.get(0);
```

#### 解析器配置

可以使用解析器配置对象 (infra.expression.spel.SpelParserConfiguration) 配置 SpEL 表达式解析器。 优先使用 SpelParserConfiguration.builder()，只设置需要覆盖的选项；全部使用构建器默认值时， 调用 SpelParserConfiguration.withDefaults()。配置对象控制某些表达式组件的行为。 例如，如果你对数组或集合进行索引，并且指定索引处的元素为 null， SpEL 可以自动创建该元素。这在使用由属性引用链组成的表达式时非常有用。 如果你对数组或列表进行索引并指定超出数组或列表当前大小的索引，SpEL 可以自动增加数组或列表以容纳该索引。 为了在指定索引处添加元素，SpEL 将尝试在设置指定值之前使用元素类型的默认构造函数创建元素。 如果元素类型没有默认构造函数，则将 null 添加

#### SpEL 编译

Infra 提供了一个基本的 SpEL 表达式编译器。表达式通常是被解释的，这在评估期间提供了很大的动态灵活性， 但不能提供最佳性能。对于偶尔的表达式使用，这很好，但是，当被其他组件（如 Infra Integration）使用时， 性能可能非常重要，并且并不真正需要动态性。 SpEL 编译器旨在解决这一需求。在评估期间，编译器生成一个体现运行时表达式行为的 Java 类， 并使用该类来实现更快的表达式评估。由于表达式周围缺乏类型化，编译器在执行编译时使用在表达式的解释评估期间收集的信息。 例如，它不能仅从表达式中知道属性引用的类型，但在第一次解释评估期间，它会找出它是什么。 当然，如果各种表达式元素的类型随时间变化，基于此类派生信息的编译稍后可能会导致问题。 因此，编译最适合那些类型信息在重复评估中不会改变的表达式。 考虑以下基本表达式。

##### 编译器配置

编译器默认未开启，但你可以通过两种不同的方式开启它。 你可以通过使用解析器配置过程（前面已讨论） 开启它，或者当 SpEL 用法嵌入在另一个组件内部时使用 Infra 属性开启它。 本节讨论这两个选项。 编译器可以在三种模式之一下运行，这些模式捕获在 infra.expression.spel.SpelCompilerMode 枚举中。 模式如下。 IMMEDIATE 模式的存在是因为 MIXED 模式可能会导致具有副作用的表达式出现问题。 如果编译后的表达式在部分成功后崩溃，它可能已经做了一些影响系统状态的事情。 如果发生了这种情况，调用者可能不希望它在解释模式下静默重新运行，因为部分表达式可能会运行两次。 选择模式后，使用 SpelParserConfiguration 配置解析器。下例展示了如何做到这一点。

## 示例中使用的类

> 原文：[`core/expressions/example-classes`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/example-classes.adoc)

### 概述

本节列出了本章示例中使用的类。

#### Inventor

```java
package samples.spel.inventor;

import java.util.Date;
import java.util.GregorianCalendar;

public class Inventor {

  private String name;
  private String nationality;
  private String[] inventions;
  private Date birthdate;
  private PlaceOfBirth placeOfBirth;

  public Inventor(String name, String nationality) {
    GregorianCalendar c= new GregorianCalendar();
    this.name = name;
    this.nationality = nationality;
    this.birthdate = c.getTime();
  }

  public Inventor(String name, Date birthdate, String nationality) {
    this.name = name;
    this.nationality = nationality;
    this.birthdate = birthdate;
  }

  public Inventor() {
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getNationality() {
    return nationality;
  }

  public void setNationality(String nationality) {
    this.nationality = nationality;
  }

  public Date getBirthdate() {
    return birthdate;
  }

  public void setBirthdate(Date birthdate) {
    this.birthdate = birthdate;
  }

  public PlaceOfBirth getPlaceOfBirth() {
    return placeOfBirth;
// ... 省略，完整示例见原文
```

#### PlaceOfBirth

```java
package samples.spel.inventor;

public class PlaceOfBirth {

  private String city;
  private String country;

  public PlaceOfBirth(String city) {
    this.city=city;
  }

  public PlaceOfBirth(String city, String country) {
    this(city);
    this.country = country;
  }

  public String getCity() {
    return city;
  }

  public void setCity(String s) {
    this.city = s;
  }

  public String getCountry() {
    return country;
  }

  public void setCountry(String country) {
    this.country = country;
  }
}
```

#### Society

## 语言参考

> 原文：[`core/expressions/language-ref`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref.adoc)

### 概述

本节描述 Infra 表达式语言的工作方式。

## 数组构造

> 原文：[`core/expressions/language-ref/array-construction`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/array-construction.adoc)

### 概述

你可以使用熟悉的 Java 语法来构建数组，可选地提供一个初始化器以便在构造时填充数组。 下例展示了如何做到这一点：

```java
	int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

	// 带初始化器的数组
	int[] numbers2 = (int[]) parser.parseExpression("new int[] {1, 2, 3}").getValue(context);

	// 多维数组
	int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

## Bean 引用

> 原文：[`core/expressions/language-ref/bean-references`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/bean-references.adoc)

### 概述

如果评估上下文已配置 bean 解析器，你可以使用 @ 符号从表达式中查找 bean。 下例展示了如何做到这一点：

```java
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// 这最终将在评估期间调用 MyBeanResolver 上的 resolve(context,"something")
Object bean = parser.parseExpression("@something").getValue(context);
```

## 集合投影

> 原文：[`core/expressions/language-ref/collection-projection`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/collection-projection.adoc)

### 概述

投影（Projection）允许集合驱动子表达式的评估，结果是一个新的集合。 投影的语法是 .![projectionExpression]。 例如，假设我们有一个发明家列表，但想要他们出生的城市列表。 实际上，我们想要对发明家列表中的每个条目评估 placeOfBirth.city。 下例使用投影来做到这一点：

```java
	// 评估结果为 ["Smiljan", "Idvor"]
List placesOfBirth = parser.parseExpression("members.![placeOfBirth.city]")
    .getValue(societyContext, List.class);
```

## 集合选择

> 原文：[`core/expressions/language-ref/collection-selection`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/collection-selection.adoc)

### 概述

选择（Selection）是一项强大的表达式语言功能，它允许你通过从源集合的条目中进行选择，将其转换为另一个集合。 选择使用的语法为 .?[selectionExpression]。它过滤集合并返回一个包含原始元素子集的新集合。 例如，选择让我们很容易获得塞尔维亚发明家的列表，如下例所示：

```java
List<Inventor> list = (List<Inventor>) parser.parseExpression(
    "members.?[nationality == 'Serbian']").getValue(societyContext);
```

## 构造函数

> 原文：[`core/expressions/language-ref/constructors`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/constructors.adoc)

### 概述

你可以使用 new 运算符调用构造函数。除了位于 java.lang 包中的类型 （Integer, Float, String 等）之外，你应该对所有类型使用完全限定的类名。 下例展示了如何使用 new 运算符调用构造函数：

```java
Inventor einstein = p.parseExpression(
    "new spel.inventor.Inventor('Albert Einstein', 'German')").getValue(Inventor.class);

// 在 List 的 add() 方法中创建新的 Inventor 实例
p.parseExpression(
    "Members.add(new spel.inventor.Inventor('Albert Einstein', 'German'))").getValue(societyContext);
```

## 函数

> 原文：[`core/expressions/language-ref/functions`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/functions.adoc)

### 概述

你可以通过注册用户定义的函数来扩展 SpEL，这些函数可以使用 #functionName(...) 语法在表达式中调用。 函数可以通过 setVariable() 方法注册为 EvaluationContext 实现中的变量。 StandardEvaluationContext 还定义了 registerFunction(...) 方法， 提供了一种方便的方式来将函数注册为 java.lang.reflect.Method 或 java.lang.invoke.MethodHandle。 由于函数与评估上下文中的 变量 共享一个公共命名空间， 因此必须注意确保函数名称和变量名称不重叠。 下例展示了如何注册一个用户定义的函数，以便使用 java.lang.reflect.Method 通过反射调用：

```java
	Method method = ...;

	EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
	context.setVariable("myFunction", method);
```

## 内联列表

> 原文：[`core/expressions/language-ref/inline-lists`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/inline-lists.adoc)

### 概述

你可以使用 {} 符号直接在表达式中表示列表。

```java
	// 评估结果为包含这四个数字的 Java 列表
	List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

	List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

## 内联 Map

> 原文：[`core/expressions/language-ref/inline-maps`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/inline-maps.adoc)

### 概述

你还可以使用 {key:value} 符号直接在表达式中表示 Map。 下例展示了如何做到这一点：

```java
	// 评估结果为包含这两个条目的 Java Map
	Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

	Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

## 字面量表达式

> 原文：[`core/expressions/language-ref/literal`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/literal.adoc)

### 概述

SpEL 支持以下类型的字面量表达式。 字符串 :: 数字 :: 布尔值 :: Null :: 由于 Infra 表达式语言的设计和实现，字面量数字在内部始终存储为正数。 例如，-2 在内部存储为正 2，然后在评估表达式时取反（通过计算 0 - 2 的值）。 这意味着不可能在 Java 中表示等于该类型数字最小值的负字面量数字。 例如，Java 中 int 的最小支持值是 Integer.MIN_VALUE，其值为 -2147483648。 但是，如果你在 SpEL 表达式中包含 -2147483648，则会抛出异常，通知你无法将值 2147483648 解析为 int （因为它超过了 Integer.MAX_VALUE 的值，即 2147483647）。 如果你需要在 SpEL 表达式中使用特定类型数字的最小值，你可以引用相应包装类型的 MIN_VALUE 常量 （例如 Integer.

```java
	ExpressionParser parser = new SpelExpressionParser();

	// 评估结果为 "Hello World"
	String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

	// 评估结果为 "Tony's Pizza"
	String pizzaParlor = (String) parser.parseExpression("'Tony''s Pizza'").getValue();

	double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

	// 评估结果为 2147483647
	int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

	boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

	Object nullValue = parser.parseExpression("null").getValue();
```

## 方法

> 原文：[`core/expressions/language-ref/methods`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/methods.adoc)

### 概述

你可以使用典型的 Java 编程语法调用方法。你也可以在字面量上调用方法。 也支持可变参数。下例展示了如何调用方法：

```java
// 字符串字面量，评估结果为 "bc"
String bc = parser.parseExpression("'abc'.substring(1, 3)").getValue(String.class);

// 评估结果为 true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
    societyContext, Boolean.class);
```

## Elvis 运算符

> 原文：[`core/expressions/language-ref/operator-elvis`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/operator-elvis.adoc)

### 概述

Elvis 运算符是三元运算符语法的缩写，用于 Groovy 语言中。 使用三元运算符语法时，通常需要重复一个变量两次，如下例所示：

```groovy
	String name = "Elvis Presley";
	String displayName = (name != null ? name : "Unknown");
```

## 安全导航运算符

> 原文：[`core/expressions/language-ref/operator-safe-navigation`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/operator-safe-navigation.adoc)

### 概述

安全导航运算符 (?.) 用于避免 NullPointerException，它来自 Groovy 语言。 通常，当你拥有对象的引用时，你可能需要在访问对象的方法或属性之前验证它是否不为 null。 为了避免这种情况，安全导航运算符针对特定的空安全操作返回 null，而不是抛出异常。 当安全导航运算符针对复合表达式中的特定空安全操作评估为 null 时， 复合表达式的其余部分仍将被评估。 有关详细信息，请参阅 <<expressions-operator-safe-navigation-compound-expressions>>。

#### 安全属性和方法访问

下例展示了如何使用安全导航运算符进行属性访问 (?.)。

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

// 评估结果为 "Smiljan"
String city = parser.parseExpression("placeOfBirth?.city") // <1>
    .getValue(context, tesla, String.class);

tesla.setPlaceOfBirth(null);

// 评估结果为 null - 不会抛出 NullPointerException
city = parser.parseExpression("placeOfBirth?.city") // <2>
    .getValue(context, tesla, String.class);
```

#### 安全索引访问

Infra 表达式语言支持对以下类型的结构进行索引的安全导航。 下例展示了如何使用安全导航运算符对列表进行索引 (?.[])。

```java
	ExpressionParser parser = new SpelExpressionParser();
	IEEE society = new IEEE();
	EvaluationContext context = new StandardEvaluationContext(society);

	// 评估结果为 Inventor("Nikola Tesla")
	Inventor inventor = parser.parseExpression("members?.[0]") // <1>
			.getValue(context, Inventor.class);

	society.members = null;

	// 评估结果为 null - 不会抛出异常
	inventor = parser.parseExpression("members?.[0]") // <2>
			.getValue(context, Inventor.class);
```

#### 安全集合选择和投影

Infra 表达式语言通过以下运算符支持 集合选择 和 集合投影 的安全导航。 下例展示了如何使用安全导航运算符进行集合选择 (?.?)。

#### 复合表达式中的空安全操作

如本节开头所述，当安全导航运算符针对复合表达式中的特定空安全操作评估为 null 时， 复合表达式的其余部分仍将被评估。这意味着必须在整个复合表达式中应用安全导航运算符， 以避免任何不需要的 NullPointerException。 给定表达式 #person?.address.city，如果 #person 为 null，安全导航运算符 (?.) 确保在尝试访问 #person 的 address 属性时不会抛出异常。 但是，由于 #person?.address 评估为 null，因此在尝试访问 null 的 city 属性时会抛出 NullPointerException。 为了解决这个问题，你可以在整个复合表达式中应用空安全导航，如 #person?.address?.city。 如果 #person 或 #person?.address 评估为 null，该表达式将安全地评估为

## 三元运算符 (If-Then-Else)

> 原文：[`core/expressions/language-ref/operator-ternary`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/operator-ternary.adoc)

### 概述

你可以使用三元运算符在表达式内部执行 if-then-else 条件逻辑。 以下清单显示了一个最小示例：

```java
String falseString = parser.parseExpression(
    "false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

## 运算符

> 原文：[`core/expressions/language-ref/operators`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/operators.adoc)

### 概述

Infra 表达式语言支持以下类型的运算符：

#### 关系运算符

使用标准运算符符号支持关系运算符（等于、不等于、小于、小于或等于、大于和大于或等于）。 这些运算符适用于 Number 类型以及实现 Comparable 的类型。 以下清单显示了一些关系运算符的示例：

```java
	// 评估结果为 true
	boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

	// 评估结果为 false
	boolean falseValue = parser.parseExpression("2 < -5.0").getValue(Boolean.class);

	// 评估结果为 true
	boolean trueValue = parser.parseExpression("'black' < 'block'").getValue(Boolean.class);

	// 使用 CustomValue:::compareTo
	boolean trueValue = parser.parseExpression("new CustomValue(1) < new CustomValue(2)").getValue(Boolean.class);
```

#### 逻辑运算符

SpEL 支持以下逻辑 (boolean) 运算符： 所有文本运算符都不区分大小写。 下例展示了如何使用逻辑运算符：

```java
// -- AND --

// 评估结果为 false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// 评估结果为 true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// 评估结果为 true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// 评估结果为 true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// 评估结果为 false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --

String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### 字符串运算符

你可以在字符串上使用以下运算符。 下例展示了 String 运算符的使用：

#### 数学运算符

你可以在数字上使用以下运算符，并强制执行标准运算符优先级。 除法和取模运算符也可以指定为纯文本等效项。 这避免了所使用的符号在嵌入表达式的文档类型中具有特殊含义的问题（例如在 XML 文档中）。文本等效项是： 所有文本运算符都不区分大小写。 自增和自减运算符可以与可写入的变量或属性一起使用前缀（A、--A）或后缀（A、A--）符号。 下例展示了数学运算符的使用：

#### 赋值运算符

要设置属性，请使用赋值运算符 (=)。这通常在调用 setValue 时完成， 但也可以在调用 getValue 时完成。以下清单显示了使用赋值运算符的两种方式：

#### 重载运算符

默认情况下，SpEL 的 Operation 枚举中定义的数学运算（ADD、SUBTRACT、DIVIDE、 MULTIPLY、MODULUS 和 POWER）支持简单类型，如数字。 通过提供 OperatorOverloader 的实现，表达式语言可以支持对其他类型的这些操作。 例如，如果我们想重载 ADD 运算符以允许使用 + 号连接两个列表，我们可以实现一个自定义 OperatorOverloader，如下所示。

## 属性、数组、列表、Map 和索引器

> 原文：[`core/expressions/language-ref/properties-arrays`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/properties-arrays.adoc)

### 概述

Infra 表达式语言支持导航对象图和索引各种结构。 NOTE: 数字索引值是从零开始的，例如在访问 Java 数组的第 n 个元素时。 TIP: 有关如何使用空安全运算符导航对象图和索引各种结构的详细信息， 请参阅 安全导航运算符 部分。

#### 属性导航

你可以使用句点指示嵌套属性值来在对象图中导航属性引用。 Inventor 类的实例 pupin 和 tesla 使用了 示例中使用的类 部分列出的数据进行填充。 为了向下导航对象图并获取 Tesla 的出生年份和 Pupin 的出生城市， 我们使用以下表达式：

```java
// 评估结果为 1856
int year = (Integer) parser.parseExpression("birthdate.year + 1900").getValue(context);

// 评估结果为 "Smiljan"
String city = (String) parser.parseExpression("placeOfBirth.city").getValue(context);
```

#### 索引数组和集合

数组或集合（例如 Set 或 List）的第 n 个元素可以使用方括号符号获取，如下例所示。 如果被索引的集合是 java.util.List，则第 n 个元素将通过 list.get(n) 直接访问。 对于任何其他类型的 Collection，第 n 个元素将通过使用其 Iterator 迭代集合 并返回遇到的第 n 个元素来访问。

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();

// 发明数组

// 评估结果为 "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
    context, tesla, String.class);

// 成员列表

// 评估结果为 "Nikola Tesla"
String name = parser.parseExpression("members[0].name").getValue(
    context, ieee, String.class);

// 列表和数组索引

// 评估结果为 "Wireless communication"
String invention = parser.parseExpression("members[0].inventions[6]").getValue(
    context, ieee, String.class);
```

#### 索引字符串

字符串的第 n 个字符可以通过在方括号内指定索引来获取，如下例所示。 NOTE: 字符串的第 n 个字符将评估为 java.lang.String，而不是 java.lang.Character。

#### 索引 Map

Map 的内容通过在方括号内指定键值来获取。 在下例中，因为 officers Map 的键是字符串，我们可以指定字符串字面量，如 'president'：

#### 索引对象

对象的属性可以通过在方括号内指定属性名称来获取。 这类似于基于键访问 Map 的值。 下例演示了如何索引对象以检索特定属性。

#### 索引自定义结构

Infra 表达式语言支持通过允许开发人员实现 IndexAccessor 并将其注册到 EvaluationContext 来索引自定义结构。如果你想支持依赖于自定义索引访问器的表达式的 编译， 该索引访问器必须实现 CompilableIndexAccessor SPI。 为了支持常见用例，Infra 提供了一个内置的 ReflectiveIndexAccessor， 它是一个灵活的 IndexAccessor，使用反射读取和可选地写入目标对象的索引结构。 索引结构可以通过 public 读取方法（读取时）或 public 写入方法（写入时）访问。 读取方法和写入方法之间的关系基于适用于索引结构典型实现的约定。 NOTE: ReflectiveIndexAccessor 也实现了 CompilableIndexAccessor，以支持 编译 为字节码以进行读取访问。 但是请注意，配置的

## 表达式模板化

> 原文：[`core/expressions/language-ref/templating`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/templating.adoc)

### 概述

表达式模板允许将字面文本与一个或多个评估块混合。 每个评估块都由你可以定义的前缀和后缀字符分隔。 通常的选择是使用 #{ } 作为分隔符，如下例所示：

```java
String randomPhrase = parser.parseExpression(
    "random number is #{T(java.lang.Math).random()}",
    new TemplateParserContext()).getValue(String.class);

// 评估结果为 "random number is 0.7038186818312008"
```

## 类型

> 原文：[`core/expressions/language-ref/types`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/types.adoc)

### 概述

你可以使用特殊的 T 运算符来指定 java.lang.Class（类型）的实例。 静态方法也通过使用此运算符来调用。 StandardEvaluationContext 使用 TypeLocator 来查找类型， 而 StandardTypeLocator（可以替换）是在了解 java.lang 包的情况下构建的。 这意味着对 java.lang 包内类型的 T() 引用不需要完全限定，但所有其他类型引用必须完全限定。 下例展示了如何使用 T 运算符：

```java
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
    "T(java.math.RoundingMode).CEILING < T(java.math.RoundingMode).FLOOR")
    .getValue(Boolean.class);
```

## 变量

> 原文：[`core/expressions/language-ref/variables`](https://github.com/TAKETODAY/today-infrastructure/blob/master/infra-docs/modules/ROOT/pages/core/expressions/language-ref/variables.adoc)

### 概述

你可以使用 #variableName 语法在表达式中引用变量。 变量通过使用 EvaluationContext 实现中的 setVariable() 方法进行设置。 变量名必须以字母（定义如下）、下划线或美元符号开头。 变量名必须由一个或多个以下支持的字符类型组成。 在 EvaluationContext 中设置变量或根上下文对象时，建议变量或根上下文对象的类型为 public。 否则，某些涉及非公共类型的变量或根上下文对象的 SpEL 表达式可能无法评估或编译。 由于变量与评估上下文中的 函数 共享一个公共命名空间， 因此必须注意确保变量名和函数名不重叠。 下例展示了如何使用变量。

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");

EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("name = #newName").getValue(context, tesla);
System.out.println(tesla.getName());  // "Mike Tesla"
```

#### #this 和 #root 变量

#this 变量始终已定义，并引用当前的评估对象（针对该对象解析非限定引用）。 #root 变量始终已定义，并引用根上下文对象。 虽然 #this 可能会随着表达式组件的评估而变化，但 #root 始终引用根。 下例展示了如何结合 集合选择 使用 #this 变量。

```java
// 创建质数整数列表
List<Integer> primes = List.of(2, 3, 5, 7, 11, 13, 17);

// 创建解析器并将变量 'primes' 设置为整数列表
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build();
context.setVariable("primes", primes);

// 从列表中选择所有大于 10 的质数（使用选择 ?{...}）
String expression = "#primes.?[#this > 10]";

// 评估结果为包含 [11, 13, 17] 的列表
List<Integer> primesGreaterThanTen =
  parser.parseExpression(expression).getValue(context, List.class);
```

---

> 本文件已达到 60 个示例的上限，其余内容请在 `infra-docs/` 原文查阅。
