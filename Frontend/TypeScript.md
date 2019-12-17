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

* `boolean`

  ```ts
  let isDone: boolean = false;
  ```

* `number`

  ```ts
  let decimal: number = 6;
  let hex: number = 0xf00d;
  let binary: number = 0b1010;
  let octal: number = 0o744;
  ```

* 





# 参考

[TypeScript](https://www.typescriptlang.org/docs/home.html)

