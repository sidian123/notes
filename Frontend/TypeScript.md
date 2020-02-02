# 引言

## 介绍

* TypeScript是JavaScript的超集

  > 即正常的JavaScript语法可在TypeScript中用, 反之不行.

## Get Started

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

# 类型

## 基础类型

## 类型推断

## 类型兼容

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

# 函数

## 介绍

带类型的函数声明, 主要涉及参数类型, 返回值类型的声明, 如

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

函数名不能当作类型定义变量, 需要使用接口表示函数的类型, 如:

```ts
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

> 这是表示函数的接口的字面值形式, 注意`=>`后接返回类型

当然, 类型自动推断, 能减少显式的类型声明, 如

```ts
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return  x + y; };

// The parameters 'x' and 'y' have the type number
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```

> 从函数推断变量类型, 或从变量推断函数类型, 都可.

## 参数

### 必需参数

```ts
function hello(word:string):string{
    return `hello ${word}`;
}
hello("world");
```

> 尽管必须, 但可以传入`null`, `undefined`

### 可选参数

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

注意, 可选参数必须位于必需参数之后

### 默认参数

函数调用在参数未被传入值或传入`undefined`后, 将使用参数的默认值.

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right
```

---

默认参数类似可选参数, 与可选参数的类型一致, 如

```ts
function buildName(firstName: string, lastName?: string) {
    // ...
}
```

```ts
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```

类型都是`(firstName: string, lastName?: string) => string`

---

与可选参数不同, 默认参数可位于必需参数之前, 若用默认值, 传入`undefined`即可

```ts
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

### Rest参数

函数调用时, 可传入多个参数, 函数内部提供数组来访问这些参数, 如

```ts
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

// employeeName will be "Joseph Samuel Lucas MacKinzie"
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

> 以`...`为Rest参数前缀.

Rest参数可传入0到多个, 必须位于最后一个参数.

## this

### 绑定this

非strict模式下, 函数的`this`指向全局变量, 对象方法的`this`指向对象本省, 但对象的方法赋值给其他对象, `this`值又会改变. 总之, `this`会随环境改变而改变

> 详细见[JavaScript]

可通过`apply()`,`call()`显式绑定`this`. 闭包也行, 如

```ts
let a={
    name:"a",
    hello(){
        return ()=>{
            console.log(`Hi, I'm ${this.name}`)
        }
    }
}
let b=a.hello();
b();
```

> Arrow函数表达式中, `this`指向对象本身, 直接返回函数, 将捕获`this`值

### this参数

上述被返回的函数中, `this`的类型为`any`, 可显式提供`this`参数, 指定其类型, 如

```ts
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```

### 回调函数中this

一个接口如下

```ts
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

回调函数`onclick()`要求`this`类型为`void`, 即用不到`this`

那么调用时需提供`this`参数

```ts
class Handler {
    info: string;
    onClickGood(this: void, e: Event) {
        // can't use `this` here because it's of type void!
        console.log('clicked!');
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickGood);
```

## 重载

JavaScript中有比较Low的重载实现, 而TS仅对它提供了类型支持, 如

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

`pickCard()`有两种重载形式, 第三个`pickCard()`不是重载的部分, 仅提供实现, 且要求: 参数和返回值类型必须包含所有重载形式的类型.

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

声明对象所需的属性, 包括函数. 

> 不能声明静态属性/方法, 它是属于类的, 而非实例对象的. Java中, 接口属性是静态的常量, 不要搞混了.

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

  只读属性被创建后, 不能被修改. 也是*必需属性*

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

## 混合类型

由于JavaScript的动态和灵活性, 可以将上述几种类型组合成一个混合类型.

如, 组合函数和对象:

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = (function (start: number) { }) as Counter;
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

## 继承

### 继承接口

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = {} as Square;
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

### 继承类

接口可继承类, 仅获取类的所有属性声明, 方法原型 ( 含`public`,`private`,`protected`的, 应该不含静态的 ) .

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class Button2{
    private state:any;
    select(){}
}

let control:SelectableControl=new Button();
let control2:SelectableControl=new Button2();//state属性的来源不同, 因此类型不兼容
```

> 关于类型兼容, 见下

## 特殊情况

### 额外属性

#### 介绍

一般来说, 对象赋值给变量时, 只要满足变量类型的必要属性, 方法存在即可 ( 此处说法不严谨 ). 即使对象有多余属性也算类型匹配.

但是有个特殊情况, 字面值对象直接赋值给变量或参数时, 不允许对象有多余的属性. 如

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });//error: colour多余
```

> 这里, 接口中两个属性都是可选的, 那么字面值对象的`colour`属性是多余的, 不允许

#### 原因

因为大多数情况下, 多余的属性意味着拼错单词了, 因此TS在这种情况下禁止这种用法了.

#### 解决办法

绕过该类检查有三种方式:

1. 类型断言

   ```ts
   let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
   ```

2. 间接赋值

   ```ts
   let squareOptions = { colour: "red", width: 100 };
   let mySquare = createSquare(squareOptions);
   ```

3. 允许任意属性存在

   ```ts
   interface SquareConfig {
       color?: string;
       width?: number;
       [propName: string]: any;
   }
   ```

### 构造函数

一般接口中不能声明静态方法的, 但有种例外, 可以声明构造函数原型, 如

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}
```

这种接口是不能被继承的, 只能用于指定变量类型, 如

```ts
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick(): void;
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

> `DigitalClock`, `AnalogClock`满足接口`ClockConstructor`的形状, 因此可以赋值给`createClock()`的参数`ctor`

# 类

## 介绍

ES6语法支持类, TS也支持, 并且更为完善.

### 声明

一个简单Demo:

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

* `constructor()`是构造函数
* 字段方法默认公有
* `this`调用实例属性&方法, `super`调用父类属性&方法
* 支持多态 ( due to prototype chain )
* 类也是一种类型, 可以出现在接口出现的地方

### 修饰符

* 访问权限

  * `public` (默认) 允许类外访问

    ```ts
    class Animal {
        public name: string;
        public constructor(theName: string) { this.name = theName; }
        public move(distanceInMeters: number) {
            console.log(`${this.name} moved ${distanceInMeters}m.`);
        }
    }
    ```

  * `private` 仅类中被访问

    ```ts
    class Animal {
        private name: string;
        constructor(theName: string) { this.name = theName; }
    }
    
    new Animal("Cat").name; // Error: 'name' is private;
    ```

    TS3.8支持私有字段的另一种写法

    ```ts
    class Animal {
        #name: string;
        constructor(theName: string) { this.#name = theName; }
    }
    
    new Animal("Cat").#name; // Property '#name' is not accessible outside class 'Animal' because it has a private identifier.
    ```

  * `protected` 类且子类中可被访问

    ```ts
    class Person {
        protected name: string;
        constructor(name: string) { this.name = name; }
    }
    
    class Employee extends Person {
        private department: string;
    
        constructor(name: string, department: string) {
            super(name);
            this.department = department;
        }
    
        public getElevatorPitch() {
            return `Hello, my name is ${this.name} and I work in ${this.department}.`;
        }
    }
    
    let howard = new Employee("Howard", "Sales");
    console.log(howard.getElevatorPitch());
    console.log(howard.name); // error
    ```

* `readonly` 属性只读, 只能在字段声明或构造函数中初始化

  ```ts
  class Octopus {
      readonly name: string;
      readonly numberOfLegs: number = 8;
      constructor (theName: string) {
          this.name = theName;
      }
  }
  let dad = new Octopus("Man with the 8 strong legs");
  dad.name = "Man with the 3-piece suit"; // error! name is readonly.
  ```

### 属性

例子:

```ts
class Person{
    name:string;//一般属性声明方式
    static num:number;//静态属性声明方式
    //参数属性声明方式
    constructor(public age:number,readonly sex:string){

    }
}
```

> 参数属性声明=声明属性+初始化, 仅在参数前添加修饰符即可

### getter&setter

```ts
class Person{
    private _age:number;
    get age(){
        return this._age;
    }
    set age(value:number){
        this._age=value;
    }
}
let me:Person=new Person();
me.age=23;
```

getter&setter是一组方法, 声明一个**伪属性**, 对该属性访问时会调用不同的方法. 

注意, 不允许存在一个同名的属性.

## 继承

* 与Java类似, 属于单继承, 只能继承一个类, 但可以实现多个接口; 
* 存在多态; 
* 子类若有构造函数, 函数必须先调用父类构造函数.

* 简单例子:

    ```ts
    class Animal {
        name: string;
        constructor(theName: string) { this.name = theName; }
        move(distanceInMeters: number = 0) {
            console.log(`${this.name} moved ${distanceInMeters}m.`);
        }
    }

    class Snake extends Animal {
        constructor(name: string) { super(name); }
        move(distanceInMeters = 5) {
            console.log("Slithering...");
            super.move(distanceInMeters);
        }
    }

    class Horse extends Animal {
        constructor(name: string) { super(name); }
        move(distanceInMeters = 45) {
            console.log("Galloping...");
            super.move(distanceInMeters);
        }
    }

    let sam = new Snake("Sammy the Python");
    let tom: Animal = new Horse("Tommy the Palomino");

    sam.move();
    tom.move(34);
    ```

* 注意--类的类型匹配规则

  不同类见, `public`属性/方法只要形状一致即可,  对于`private`, `protect`属性/方法, 除了形状外, 原型链上的来源也要一致.

  > 例子见接下来的*类型兼容*

## 抽象类

抽象类不能实例化, 只能被继承, 可以有抽象方法(也可无), 方法实现.

```ts
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log("roaming the earth...");
    }
}
```

## 其他

### 获取类的类型

类如`Person`, 代表的是实例化对象的类型. 类自身的类型可由`typeof Person`给出. 如

```ts
class Greeter {
    static standardGreeting = "Hello, there";
	...
}
//获取类的类型, 直接修改类的静态属性值.
let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";
```

# 其他

## 类型

### 类型推断

### 类型兼容

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