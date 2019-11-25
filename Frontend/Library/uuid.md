* 介绍

  * 支持1,3,4,5版本的UUID
  
* 安装

  ```shell
  npm install uuid --save
  ```

* 引入

  `uuid/[v1|v3|v4|v5] `, 可选择不同版本, 如
  
  ```shell
import uuidv4 from "uuid/v4"
  ```
  

* 使用

  每个版本UUID使用的算法都不一样, 个人喜欢用`v4`版本的无参调用形式

  ```javascript
  uuidv4();
  ```

> 参考[uuid](https://www.npmjs.com/package/uuid)