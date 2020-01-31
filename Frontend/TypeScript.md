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

# 基础类型

## Boolean

```ts
let isDone: boolean = false;
```

## Number

```ts
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;
let octal: number = 0o744;
```

## String

```ts
let color: string = "blue";
color = 'red';
let fullName: string = `Bob Bobbington`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ fullName }.
```

## Array

```ts
let list: number[] = [1, 2, 3];
```

或使用泛型数组`Array<elemType>`

```ts
let list: Array<number> = [1, 2, 3];
```

## Tuple

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

## Enum

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

枚举本质是由数组实现的, 因此可通过索引获取值

```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

console.log(colorName); // Displays 'Green' as its value is 2 above
```

## Any

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

## Void

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

## Null and Undefined

```ts
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

默认`null`和`undefined`是其他所有类型的子类, 因此可以赋值给任意对象

当`--strictNullChecks`被设置后, `null`和`undefined`只能赋值给`any`和自己的类型 ( 除了`undefined`也可以赋值给`void`.

## Never

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

## Object

表示所有的类型, 除了`number`, `string`, `boolean`, `bigint`, `symbol`, `null`, 或 `undefined`.

## Type assertions

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

# 接口

## 介绍

接口规定了类型检查时, 对象应有的**形状**; 类型检查并不严格, 只要形状匹配, 便类型匹配.

总之, 任何类型都可由接口声明.

## 接口声明

字面值接口

```ts
function printLabel(labeledObj: { label: string }) {
    console.log(labeledObj.label);
}
```

关键字声明

```ts
interface LabeledValue {
    label: string;
}

function printLabel(labeledObj: LabeledValue) {
    console.log(labeledObj.label);
}
```

## 属性声明

声明对象所需的属性, 包括函数

* 必需属性

  类型检查时, 必须存在的属性

  ```ts
  interface LabeledValue {
      label: string;
  }
  ```

  > `label`属性必须存在

* 可选属性

  类型检查时, 可选的属性

  ```ts
  interface SquareConfig {
      color?: string;
      width?: number;
  }
  function createSquare(config: SquareConfig): {color: string; area: number} {
  	...
  }
  let mySquare = createSquare({color: "black"});
  ```

  > 在属性名后加`?`声明

* 只读属性

  只读属性被创建后, 不能被修改

  ```ts
  interface Point {
      readonly x: number;
      readonly y: number;
  }
  let p1: Point = { x: 10, y: 20 };
  p1.x = 5; // error!
  ```

  > 在属性名前加`readonly`声明; 只读属性仅在初始化时可赋值.

  > `readonly` vs `const`
  >
  > `const`声明变量只读, `readonly`声明属性只读.

* 方法

  只给出函数原型, 无实现即可

  ```ts
  interface Car{
      run(num:number):string;
  }
  ```

## 函数类型

通过接口, 描述函数, 而非声明方法属性.

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

> 与函数原型基本一致, 除了无函数名.

使用时, 参数名可以不一致, 类型可以不显式声明, 由编译器自行推断

```ts
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

## 可索引类型

数组和对象都是可索引的, 如`a[10]`, `ageMap['daniel']`

* 数组接口

  ```ts
  interface StringArray {
      [index: number]: string;
  }
  
  let myArray: StringArray;
  myArray = ["Bob", "Fred"];
  
  let myStr: string = myArray[0];
  ```

  > `index`为`number` 表示索引是数字, 所以接口表示数组类型

  数组可以有其他属性

  ```ts
  interface Cars {
      [index: number]: String;
      length: number;    
  }  
  ```

* 可索引的对象

  这种对象可以动态增加属性

  ```ts
  interface Cars {
      [index: string]: String;  
  }  
  let cars:Cars={};
  cars['a']="23";
  cars.a='233';
  ```

  > `index`为`string`, 表示对象

  可含有其他属性, 但是属性类型必须与索引的属性类型一致

  ```ts
  interface NumberDictionary {
      [index: string]: number;
      length: number;    // ok, length is a number
      name: string;      // error, the type of 'name' is not a subtype of the indexer
  }
  ```

  > 因为通过索引也可访问`name`, 如`a['name']`, 若类型不一致, 那它到底为何类型? 因此冲突了.
  >
  > 有个解决办法, 使用联合类型:
  >
  > ```ts
  > interface NumberOrStringDictionary {
  >     [index: string]: number | string;
  >     length: number;    // ok, length is a number
  >     name: string;      // ok, name is a string
  > }
  > ```
  >
  > > 即属性可`number`, 可`string`

* 混合

  混合使用数组或对象索引.

  这里有个转化: 数组也是对象, 索引在对象中以`string`存储, 索引时, 会将`number`转化为`string`来访问.

  因此, 数组索引的元素类型必须匹配对象索引的属性类型

  ```ts
  class Animal {
      name: string;
  }
  class Dog extends Animal {
      breed: string;
  }
  
  interface NotOkay {
      [x: number]: Dog;
      [x: string]: Animal;
  }
  ```

  > `Dog`满足`Animal`的类型形状

## 继承











## 特殊情况

### 额外属性

### 静态属性

# 类







# 其他

## 类型兼容

TypeScript is a structural type system. When we compare two different types, regardless of where they came from, if the types of all members are compatible, then we say the types  themselves are compatible.

However, when comparing types that have `private` and `protected` members, we treat these types differently. For two types to be considered compatible, if one of them has a `private` member, then the other must have a `private` member that originated in the same declaration. The same applies to `protected` members.

Let’s look at an example to better see how this plays out in practice:

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // Error: 'Animal' and 'Employee' are not compatible
```



# 参考

* [TypeScript Docs](https://www.typescriptlang.org/docs/home.html)
* [TypeScript Playground](https://www.typescriptlang.org/play/index.html#) TypeScript To JavaScript 的在线编译器

* [typescript-tutorial](https://github.com/xcatliu/typescript-tutorial) ts教程