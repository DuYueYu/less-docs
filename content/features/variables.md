---
title: 变量/Variables
---

>用一个单独区域存储一个常用值
> Control commonly used values in a single location.

### Overview

It's not uncommon to see the same value repeated dozens _if not hundreds of times_ across your stylesheets:
在你的式样表中没有几百也有几十次写了相同的属性值：

```css
a,
.link {
  color: #428bca;
}
.widget {
  color: #fff;
  background: #428bca;
}
```

Variables make your code easier to maintain by giving you a way to control those values from a single location:
变量提供了一种在单独区域中控制这些属性值的方式，使你的代码更容易维护：

```less
// 变量：
@link-color:        #428bca; // sea blue
@link-color-hover:  darken(@link-color, 10%);

// 使用：
a, .link {
  color: @link-color;
}
a:hover {
  color: @link-color-hover;
}
.widget {
  color: #fff;
  background: @link-color;
}
```

### Variable Interpolation / 变量修改

上面的例子主要使用变量去控制CSS属性值，但变量同样可以用在别的地方，比如选择器名、属性名、URL、和`@import`语句。
The examples above focused on using variables to control _values in CSS rules_, but they can also be used in other places as well, such as selector names, property names, URLs and `@import` statements.


#### Selectors / 值作为选择器

_v1.4.0_

```less
// 变量
@my-selector: banner;

// 使用
.@{my-selector} {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```
编译为:

```css
.banner {
  font-weight: bold;
  line-height: 40px;
  margin: 0 auto;
}
```

#### URLs

```less
// 变量
@images: "../img";

// 使用
body {
  background: url("@{images}/white-sand.png");
}
```
编译为:

```css
body {
  background: url("../img/white-sand.png");
}
```

#### Import Statements / Import 语句

_v1.4.0_

语法: `@import "@{themes}/tidal-wave.less";`

Note that before v2.0.0, only variables which have been declared in the root or current scope were considered and that only the current file and calling files were considered when looking for a variable.
注意，在v2.0.0之前，less只考虑在全局域或当前域中声明的变量，在查找变量时只考虑当前文件和调用文件。

例子:

```less
// Variables
@themes: "../../src/themes";

// Usage
@import "@{themes}/tidal-wave.less";
```

#### Properties / 值作为属性名

_v1.6.0_

```less
@property: color;

.widget {
  @{property}: #0ee;
  background-@{property}: #999;
}
```

编译为:

```css
.widget {
  color: #0ee;
  background-color: #999;
}
```

### Variable Variables / 可变变量

In Less, you can define a variable's name using another variable.
在Less中，你可以定义一个使用其他变量值的变量
这里应该是参考了编译语言的对象引用，像是JAVA中一个变量存储了对象引用，使用的时候就需要加点符号表明‘不是使用这个变量本身，而是使用变量引用的对象’；
这里多加一个@表示使用引用；
这个用在简单的变量上应该不合适，应该还有别的用法吧？

```less
@primary:  green;
@secondary: blue;

.section {
  @color: primary;
  
  .element {
    color: @@color;
  }
}
```

编译为:

```less
.section .element {
  color: green;
}
```

<span class="anchor-target" id="variables-feature-lazy-loading"></span>
<!-- ^ please keep old anchor to not break zillion outer links -->
### Lazy Evaluation

> Variables do not have to be declared before being used.
>变量

Valid Less snippet:

```less
.lazy-eval {
  width: @var;
}

@var: @a;
@a: 9%;
```
this is valid Less too:

```less
.lazy-eval {
  width: @var;
  @a: 9%;
}

@var: @a;
@a: 100%;
```
both compile into:

```css
.lazy-eval {
  width: 9%;
}
```

When defining a variable twice, the last definition of the variable is used, searching from the current scope upwards. This is similar to css itself where the last property inside a definition is used to determine the value.

For instance:

```less
@var: 0;
.class {
  @var: 1;
  .brass {
    @var: 2;
    three: @var;
    @var: 3;
  }
  one: @var;
}
```
Compiles to:

```css
.class {
  one: 1;
}
.class .brass {
  three: 3;
}
```

Essentially, each scope has a "final" value, similar to properties in the browser, like this example using custom properties:
```css
.header {
  --color: white;
  color: var(--color);  // the color is black
  --color: black;
}
```
This means that, unlike other CSS pre-processing languages, Less variables behave very much like CSS's.


### Properties as Variables **(NEW!)**

_v3.0.0_

You can easily treat properties like variables using the `$prop` syntax. Sometimes this can
make your code a little lighter.

```less
.widget {
  color: #efefef;
  background-color: $color;
}
```

Compiles to:

```css
.widget {
  color: #efefef;
  background-color: #efefef;
}
```

Note that, like variables, Less will choose the last property within the current/parent scope
as being the "final" value.

```less
.block {
  color: red; 
  .inner {
    background-color: $color; 
  }
  color: blue;  
} 
```

Compiles to:
```css
.block {
  color: red; 
  color: blue;  
} 
.block .inner {
  background-color: blue; 
}
```

### Default Variables

We sometimes get requests for default variables - an ability to set a variable only if it is not already set. This feature is not required because you can easily override a variable by putting the definition afterwards.

For instance:

```less
// library
@base-color: green;
@dark-color: darken(@base-color, 10%);

// use of library
@import "library.less";
@base-color: red;
```

This works fine because of [Lazy Loading](#variables-feature-lazy-loading) - `@base-color` is overridden and `@dark-color` is a dark red.
