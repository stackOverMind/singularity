---
title: google closure compiler 中的 annotation
date: 2016-11-03 16:25:16
tags: [google closure compiler,前端]
---

因为经常要用到 google closure compiler 中的 annotation,所以索性把https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler 翻译出来。

虽然google closure compiler 用的人一直不多，但对于一些需要高压缩比的js项目来说还是无可取代的。翻译出来，或许能够对别人有所帮助。

`Closure Compiler` 可以利用Js变量的类型信息来增强优化。然而JS中没有类型。因为JS中没有类型的概念，所以你必须使用注释来标注数据的类型。

`Closure Compiler`使用 JSdoc中使用的annotation。这篇文档描述了ClosureCompiler能够认识的annotation

# JSdoc 标签

Closure Compiler（以下称为编译器） 在 JSdoc 标签中寻找有用的类型信息。使用下面的标签可以帮助 Closure Compiler 优化你的代码，检查可能的类型错误和其他问题。

以下仅仅包含能够影响Closure Compiler 行为的JsDoc 标签。

## `@const` `@const {type}`

标记一个变量只读，compiler会把`@const` 变量做inline 处理，从而优化js执行效率

type 声明是可选的。

如果@const 变量赋值超过1次，编译器会产生一条警告信息。如果变量是个对象，编译器不会禁止内部属性的变化。

```js
** @const */ var MY_BEER = 'stout';

/**
 * My namespace's favorite kind of beer.
 * @const {string}
 */
mynamespace.MY_BEER = 'stout';

```

## `@constructor`

标记一个函数是构造函数，对于每一个使用new 作为关键字的函数，编译器需要 `@constructor` annotation 。如果使用EcmaScript的 `class` 构造器或使用 `google.defineClass` @constructor 应该被省略.

```js
/**
 * A rectangle.
 * @constructor
 */
function GM_Rect() {
  ...
}

```

## `@define {Type}` `description`

标记一个变量可能在编译时被编译器覆盖。下面的例子中，你可以向编译器传入参数`--define='ENABLE_DEBUG=false'` 来改变ENABLE_DEBUG 的值。类型可以是 `number`, `string` 或 `boolean` 。只能在全局作用域定义

```js
/** @define {boolean} */
var ENABLE_DEBUG = true;

/** @define {boolean} */
goog.userAgent.ASSUME_IE = false;

```

## `@deprecated` `Description`

警告用户不建议使用这个function，method 或property。使用的话会报一条warning。

```js
/**
 * Determines whether a node is a field.
 * @return {boolean} True if the contents of
 *     the element are editable, but the element
 *     itself is not.
 * @deprecated Use isField().
 */
BN_EditUtil.isTopEditableField = function(node) {
  ...
};

```

## `dict`

@dict 用来创建property数量可变的对象。当一个constructor（这个例子中的`Foo`）被注释为@dict,你只能使用`[]`来访问Foo的属性。这个注释还可以直接用到一个对象的literals 上

```js
/**
 * @constructor
 * @dict
 */
function Foo() {}
var obj1 = new Foo();
obj1['x'] = 123;
obj1.x = 234;  // warning

var obj2 = /** @dict */ { 'x': 321 };
obj2.x = 123;  // warning
```

## `@enum {Type}`

Specifies the type of an enum. An enum is an object whose properties constitute a set of related constants. The @enum tag must be followed by a type expression. If the type of an enum is omitted, number is assumed.

The type label of an enum applies to each property of the enum. For example if an enum has type number, each of its enumerated properties must be a number.

```js
/**
 * Enum for tri-state values.
 * @enum {number}
 */
project.TriState = {
  TRUE: 1,
  FALSE: -1,
  MAYBE: 0
};
```


## @export, @export {SomeType}

当一个属性 @export 标记，并且编译器有 ` --generate_exports` ，一个相应的goog.exportSymbol声明会生成。

```js
/** @export */
foo.MyPublicClass.prototype.myPublicMethod = function() {
  // ...
};
goog.exportSymbol('foo.MyPublicClass.prototype.myPublicMethod',
foo.MyPublicClass.prototype.myPublicMethod);
You can write /** @export {SomeType} */ as a shorthand for /** @export @type {SomeType} */.
```

Code that uses the @export annotation must either:
使用@export的代码必须 引入 closure/base.js 或在用户自己codebase中定义 ` goog.exportSymbol`

**注：编译器会做很多优化，比如替换属性名，使用@export 就是为了属性名不被替换，在外部能够访问到**

## @nocollapse

Denotes a property that should not be collapsed by the compiler into a variable. The primary use for @nocollapse is to allow exporting of mutable properties. Note that non-collapsed properties can still be renamed by the compiler. If you annotate a property that is an object with @nocollapse, all its properties will also remain uncollapsed.

```js
/**
 * A namespace.
 * @const
 */
var foo = {};

/**
 * @nocollapse
 */
foo.bar = 42;

window['foobar'] = foo.bar;
```

## @extends {Type}

Marks a class or interface as inheriting from another class. A class marked with @extends must also be marked with either @constructor or @interface.

Note: @extends does not cause a class to inherit from another class. The annotation simply tells the compiler that it can treat one class as a subclass of another during type-checking.
For an example implementation of inheritance, see the Closure Library function goog.inherits().

```js
/**
 * Immutable empty node list.
 * @constructor
 * @extends {goog.ds.BasicNodeList}
 */
goog.ds.EmptyNodeList = function() {
  ...
};
```

## @externs

Declares an externs file.

Files annotated with @externs in the @fileoverview block are treated as externs files even if they appear compilation units regular srcs.

For example:

```js
/**
 * @fileoverview This is an externs file.
 * @externs
 */
var document;

```

## @fileoverview Description

Makes the comment block provide file level information including suppressions.

For example:

```js
/**
 * @fileoverview Utilities for doing things 
 */
```

## @final

Indicates that this class is not allowed to be extended. For methods, indicates that no subclass is allowed to override that method.

```js
/**
 * A class that cannot be extended.
 * @final
 * @constructor
 */
sloth.MyFinalClass = function() { ... }

/**
 * A method that cannot be overridden.
 * @final
 */
 
sloth.MyFinalClass.prototype.method = function() { ... };
```

## @implements {Type}

Used with @constructor to indicate that a class implements an interface.

The compiler produces a warning if you tag a constructor with @implements and then fail to implement all of the methods and properties defined by the interface.

```js
/**
 * A shape.
 * @interface
 */
function Shape() {};
Shape.prototype.draw = function() {};

/**
 * @constructor
 * @implements {Shape}
 */
function Square() {};
Square.prototype.draw = function() {
  ...
};
```

## @implicitCast

This annotation can only appear in externs property declarations. The property has a declared type, but you can assign any type to it without a warning. When accessing the property, you get back a value of the declared type. For example, element.innerHTML can be assigned any type, but will always return a string.

```js
/**
 * @type {string}
 * @implicitCast
 */
Element.prototype.innerHTML;
```

## @inheritDoc

Indicates that a method or property of a subclass intentionally hides a method or property of the superclass, and has exactly the same documentation. Note that the @inheritDoc tag implies the @override tag, @override is preferred.

```js
/** @inheritDoc */
project.SubClass.prototype.toString = function() {
  ...
};
```


## @interface @record

Marks a function as an interface. An interface specifies the required members of a type. Any class that implements an interface must implement all of the methods and properties defined on the interface's prototype. See @implements.

The compiler verifies that interfaces are not instantiated. If the new keyword is used with an interface function, the compiler produces a warning.

For the differences between @record and @interface, see Structural Interfaces

```js
/**
 * A shape.
 * @interface
 */
function Shape() {};
Shape.prototype.draw = function() {};

/**
 * A polygon.
 * @interface
 * @extends {Shape}
 */
function Polygon() {};

Polygon.prototype.getSides = function() {};
```


## @lends {objectName}

Indicates that the keys of an object literal should be treated as properties of some other object. This annotation should only appear on object literals.

Notice that the name in braces is not a type name like in other annotations. It's an object name. It names the object to which the properties are lent. For example, @type {Foo} means "an instance of Foo", but @lends {Foo} means "the constructor Foo".

The JSDoc Toolkit docs have more information on this annotation.

```js
goog.object.extend(
    Button.prototype,
    /** @lends {Button.prototype} */ ({
      isButton: function() { return true; }
    }));
```

## @license, @preserve Description

Tells the compiler to insert the associated comment before the compiled code for the marked file. This annotation allows important notices (such as legal licenses or copyright text) to survive compilation unchanged. Line breaks are preserved.

```js
/**
 * @preserve Copyright 2009 SomeThirdParty.
 * Here is the full license text and copyright
 * notice for this file. Note that the notice can span several
 * lines and is only terminated by the closing star and slash:
 */
```



## @nosideeffects @modifies {this|arguments}

@nosideeffects indicates that a call to the declared function has no side effects. This annotation allows the compiler to remove calls to the function if the return value is not used. This is not a signal that the function is "pure", it may still read mutable global state.

@modifies {this} signals that only direct properties of the provided this are modified. Calls to these functions maybe be removed if the "this" object is known to be otherwise unused.

@modifies {arguments} signals that only direct properties of the provided arguments are modified. Calls to these functions maybe be removed if the parameters provided to the function are otherwise known to be otherwise unused.

Theses annotation is only allowed in extern files.

```js
/** @nosideeffects */
function noSideEffectsFn1() {
  ...
};

/** @nosideeffects */
var noSideEffectsFn2 = function() {
  ...
};

/** @nosideeffects */
a.prototype.noSideEffectsFn3 = function() {
  ...
};
@override

/**
 * @return {string} Human-readable representation of
 *     project.SubClass.
 * @override
 */
project.SubClass.prototype.toString = function() {
  ...
};
```

Indicates that a method or property of a subclass intentionally hides a method or property of the superclass. If no other annotations are included, the method or property automatically inherits annotations from its superclass.

## @package

Marks a member or property as package private. Only code in the same directory can access names marked @package. In particular, code in parent and child directories cannot access names marked @package.

Public constructors can have @package properties to restrict the methods that callers outside the directory can use. On the other hand, @package constructors can have public properties to prevent callers outside the directory from directly instantiating a type.

```js
/**
 * Returns the window object the foreign document resides in.
 *
 * @return {Object} The window object of the peer.
 * @package
 */
goog.net.xpc.CrossPageChannel.prototype.getPeerWindowObject = function() {
  // ...
};
```

## @param {Type} varname Description

Used with method, function and constructor definitions to specify the types of function arguments.

The @param tag must be followed by a type expression.

Alternatively, you can annotate the types of the parameters inline (see function foo in the example).
```js
/**
 * Queries a Baz for items.
 * @param {number} groupNum Subgroup id to query.
 * @param {string|number|null} term An itemName,
 *     or itemId, or null to search everything.
 */
goog.Baz.prototype.query = function(groupNum, term) {
  ...
};

function foo(/** number */ a, /** number */ b) {
  return a - b + 1;
}
```

## @private

Marks a member as private. Only code in the same file can access global variables and functions marked @private. Constructors marked @private can only be instantiated by code in the same file and by their static and instance members.

The public static properties of constructors marked @private may also be accessed anywhere, and the instanceof operator can always access @private members.

```js
/**
 * Handlers that are listening to this logger.
 * @private {Array<Function>}
 */
this.handlers_ = [];

```

## @protected

Indicates that a member or property is protected. A property marked @protected is accessible to:

all code in the same file
static methods and instance methods of any subclass of the class on which the property is defined.

```js

/**
 * Sets the component's root element to the given element.
 * Considered protected and final.
 * @param {Element} element Root element for the component.
 * @protected
 */
goog.ui.Component.prototype.setElementInternal = function(element) {
  // ...
};
```

## @public

Indicates that a member or property is public. A property marked @public is accessible to all code in the any file. This is the implicit default and rarely used. This is not used to indicate that name should be preserved in obfuscating builds see @export.

```js
/**
 * @public
 */
goog.ui.Component.prototype.setElementInternal = function(element) {
  // ...
};
```

## @return {Type} Description

Specifies the return types of method and function definitions. The @return tag must be followed by a type expression.

Alternatively, you can annotate the return type inline (see function foo in the example).

If a function that is not in externs has no return value, you can omit the @return tag, and the compiler will assume that the function returns undefined.

```js
/**
 * Returns the ID of the last item.
 * @return {string} The hex ID.
 */
goog.Baz.prototype.getLastId = function() {
  ...
  return id;
};

function /** number */ foo(x) { return x - 1; }
```

## @struct

@struct is used to create objects with a fixed number of properties. When a constructor (Foo in the example) is annotated with @struct, you can only use the dot notation to access the properties of Foo objects, not the bracket notation. Also, you cannot add a property to a Foo instance after it's constructed. The annotation can also be used directly on object literals.

```js
/**
 * @constructor
 * @struct
 */
function Foo(x) {
  this.x = x;
}

var obj1 = new Foo(123);
var someVar = obj1.x;  // OK
obj1.x = "qwerty";  // OK
obj1['x'] = "asdf";  // warning
obj1.y = 5;  // warning

var obj2 = /** @struct */ { x: 321 };
obj2['x'] = 123;  // warning
```

## @suppress {warningGroup1,warningGroup2}

Suppresses warnings. Warning categories are separated by | or ,. For a list of warning names, take a look at Warnings.

For example:

```js
/**
 * @suppress {deprecated}
 */
function f() {
  deprecatedVersionOfF();
}
@template T

See Generic Types.

/**
 * @param {T} t
 * @constructor
 * @template T
 */
Container = function(t) { ... };
```

## @this {Type}

Specifies the type of the object to which the keyword this refers within a function. The @this tag must be followed by a type expression.

To prevent compiler warnings, you must use a @this annotation whenever this appears in a function that is neither a prototype method nor a function marked as a @constructor.

```js
    /**
     * Returns the roster widget element.
     * @this {Widget}
     * @return {Element}
     */
    function() {
      return this.getComponent().getElement();
    });
```

## @throws {Type}

Used to document the exceptions thrown by a function. The type checker does not currently use this information. It is only used to figure out if a function declared in an externs file has side effects.

```js
/**
 * @throws {DOMException}
 */
DOMApplicationCache.prototype.swapCache = function() { ... };
```

## @type {Type}

Identifies the type of a variable, property, or expression. The @type tag must be followed by a type expression. You can also write the type annotation inline and omit @type, as in the second example.

```js

/**
 * The message hex ID.
 * @type {string}
 */
var hexId = hexId;
var /** string */ name = 'Jamie';
```

## @typedef {Type}

Declares an alias for a more complex type. Currently, typedefs can only be defined at the top level, not inside functions. We have fixed this limitation in the new type inference.

```js
/** @typedef {(string|number)} */
goog.NumberLike;

/** @param {goog.NumberLike} x A number or a string. */
goog.readNumber = function(x) {
  ...
}
```

## @unrestricted

Indicates that a class is neither a @struct type, nor a @dict type. This is the default so it is generally not necessary to write it explicitly, unless you are using goog.defineClass, or the class keyword, which both produce classes which are @structs by default.

```js
/**
 * @constructor
 * @unrestricted
 */
function Foo(x) {
  this.x = x;
}

var obj1 = new Foo(123);
var someVar = obj1.x;  // OK
obj1.x = "qwerty";  // OK
obj1['x'] = "asdf";  // OK
obj1.y = 5;  // OK
```
Special purpose annotations

## @noalias

Used in an externs file to indicate to the compiler that the variable or function should not be aliased as part of the "alias externals" pass of the compiler, is not enabled by default and only available through the Java API.

For example:

```js
/** @noalias */
function Range() {}
```

## @nocompile

For example

```js
/** @nocompile */

```
Used at the top of a file to tell the compiler to parse this file but not compile it. Code that is not meant for compilation and should be omitted from compilation tests (such as bootstrap code) uses this annotation. Most code should use other means such as a @define to change behavior.