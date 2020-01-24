# 介绍

* TypeScript是JavaScript的超集

  > 即正常的JavaScript语法可在TypeScript中用, 反之不行.

# Get Started

* 安装

  ```shell
  npm install -g typescript
  ```

* 编译

  ```shell
  tsc greeter.ts
  ```

* 类型注解

  ```typescript
  function greeter(person: string) {
      return "Hello, " + person;
  }
  ```

  标注于变量, 参数, 字段上的类型信息, 主要让IDE获取更多的信息, 更好的进行类型检查, 代码补全等.
  
* 接口

  是对象内部结构的约定; 

  一个类型要兼容一个接口, 仅需拥有接口的内部结构即可; 

  实现一个接口可直接定义接口一致的结构即可, 无需显式声明`implements`

  ```ts
  interface Person {
      firstName: string;
      lastName: string;
  }
  
  function greeter(person: Person) {
      return "Hello, " + person.firstName + " " + person.lastName;
  }
  
  let user = { firstName: "Jane", lastName: "User" };
  
  document.body.textContent = greeter(user);
  ```

* 类

  ```ts
  class Student {
      fullName: string;
      constructor(public firstName: string, public middleInitial: string, public lastName: string) {
          this.fullName = firstName + " " + middleInitial + " " + lastName;
      }
  }
  ```

  > 构造函数中`public`声明的参数将自动生成对应字段.

# 语法

## 基础类型

### Boolean

```ts
let isDone: boolean = false;
```

### Number

```ts
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

### String

```ts
let color: string = "blue";
color = 'red';
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.
```

### Array

```ts
let list: number[] = [1, 2, 3];
```

或使用泛型数组`Array<elemType>`

```ts
let list: Array<number> = [1, 2, 3];
```

### Tuple

也是一种数据, 但是数组元素类型可以不同

```ts
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ["hello", 10]; // OK
// Initialize it incorrectly
x = [10, "hello"]; // Error
console.log(x[0].substring(1)); // OK
console.log(x[1].substring(1)); // Error, 'number' does not have 'substring'
x[3] = "world"; // Error, Property '3' does not exist on type '[string, number]'.
console.log(x[5].toString()); // Error, Property '5' does not exist on type '[string, number]'.
```

### Enum

TypeScript中新增的类型

```ts
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

每个元素对应一个数值, 第一个元素默认为零, 之后以此递增. 但可修改:

```ts
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```

```ts
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

枚举本质是由数据实现的, 因此可通过索引获取值

```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

console.log(colorName); // Displays 'Green' as its value is 2 above
```

### Any

为了兼容三方无类型信息的对象, 提供了`any`类型, 表示任何对象, <u>避免编译时类型检查</u>, 如

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

与`Object`相比, `any`完全退出了类型检查, 而`Object`还处于继承的关系中, 如:

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

`any`也可以附带部分信息

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

### Void

用于函数, 表示无返回值

```ts
function warnUser(): void {
    console.log("This is my warning message");
}
```

用于声明变量类型, 则该变量只能赋值`undefined`和`null`( `--strictNullChecks`未指定的情况下) 

```ts
let unusable: void = undefined;
unusable = null; // OK if `--strictNullChecks` is not given
```

### Null and Undefined

```ts
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

默认`null`和`undefined`是其他所有类型的子类, 因此可以赋值给任意对象

当`--strictNullChecks`被设置后, `null`和`undefined`只能赋值给`any`和自己的类型 ( 除了`undefined`也可以赋值给`void`.

### Never

用于声明变量, 它的值永远得不到,如

```ts
// Function returning never must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is never
function fail() {
    return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

> 三个函数的返回值永远得不到.

`never`是所有类型的子类, 可以赋值给所有类型. 但`never`除了自己, 不能被任何类型的变量赋值.

### Object

表示所有的类型, 除了`number`, `string`, `boolean`, `bigint`, `symbol`, `null`, 或 `undefined`.

### Type assertions

与类型转化cast类型, 但不对运行时造成影响, 仅作用于编译器.

有两种语法形式

* Angle-Bracket语法

  ```ts
  let someValue: any = "this is a string";
  
  let strLength: number = (<string>someValue).length;
  ```

* As语法

  ```ts
  let someValue: any = "this is a string";
  
  let strLength: number = (someValue as string).length;
  ```

> TypeScript于JSX一起使用时, 仅As语法可用.

## 变量声明

* `var` ( 弃用 )

  声明的变量

  1. 没有[block scope](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/block)（块作用域）,只有function, module, namespace, or global scope ；
  2. **声明**有[hoisting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting)现象，但**初始化**没有该现象；
  3. 可以多次声明变量。

* `let`

  有块作用域; 无`hoisting`现象; 仅能声明一次; 嵌套作用域中存在*shadowing*

* `const`

  与`let`一致, 但是变量不能多次被赋值

* 解构

  * 数组解构
  * 元组解构
  * 对象解构
    * 属性重命名
    * 默认值

* 方法声明

  * 声明有*hoisting*现象, 无块作用域, 但该作用域会限制函数声明*hosting*到该块作用域的顶端

* Spread语法

> 参考
>
> * [JavaScript]
> * [Variable Declarations in TypeScript](https://www.typescriptlang.org/docs/handbook/variable-declarations.html)

## 接口



# 其他

`union`类型

------

```ts
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

`?`表示可选

# 参考

* [TypeScript Docs](https://www.typescriptlang.org/docs/home.html)
* [TypeScript Playground](https://www.typescriptlang.org/play/index.html#) TypeScript To JavaScript 的在线编译器

