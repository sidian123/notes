[TOC]

# 一 介绍

Node.js是一个开源的、跨平台的JavaScript运行时环境，使JavaScript可以运行在浏览器端之外；Node.js是事件驱动的架构，支持异步IO，单线程；Node.js实现了CommonJS的模块加载功能（当然也支持ES6模块语法）。

但是开发者常用Node.js制作进行前端开发的工具（如Gurnt、Gulp和Webpack），进行前端开发。

# 二 模块
**模块**（module）是一个可复用的功能模块，一个模块对应一个Js文件，模块可导出模块内的变量、函数、对象给其他模块使用。

一个**package**（包）可含有一个或多个module，但只暴露某个模块（通过`package.json`中的`main`属性）。`package.json`记录该包的所有属性，如包依赖、作者等。

>把package看成是module也行，毕竟只暴露一个文件的导出，在被引入时表现得像本地模块一样，
## module
模块通过`module.exports`导出对象（`exports`），通过`require`获得对象(`exports`)，下面直接看模块的实现原理。

### module wrapper
假设一个模块：

```javascript
const PI = 3.14159265359;

exports.area = radius => (radius ** 2) * PI;
exports.circumference = radius => 2 * radius * PI;
```
其他模块中要引入该模块时，会用一个匿名函数包裹该模块：
```javascript
(function (exports, require, module, __filename, __dirname) {
    module.exports = exports = {};

    // Your module code ...
	const PI = 3.14159265359;

	exports.area = radius => (radius ** 2) * PI;
	exports.circumference = radius => 2 * radius * PI;
});
```
当`require`一个模块时，会执行该wapper函数，得到模块导出的对象`module.exports`。

可以看出`exports`与`module.exports`指向同一个对象，但最终导出的是`module.exports`对象。因此如果直接`exports=obj`赋值对象，由于是引用传值的，并不会修改`module.exports`导致该对象没有被导出，需要使用`module.exports=obj`。

wrapper函数的参数`module`不是全局的，每个模块都有一个`module`，它除了包含模块导出对象`exports`外，还包含id、parent、children等等。

剩下的`__filename`表示当前模块文件路径；`__dirname`表示模块文件所处的目录路径；`require`是用来导入其他模块的函数。

### require
使用方式如：
```javascript
const fs = require('fs');

fs.readFile('./file.txt', 'utf-8', (err, data) => {
  if(err) { throw err; }
  console.log('data: ', data);
});
```
`require()`函数返回的是被导入模块导出的对象`exports`。函数参数传入模块的文件名（无后缀），它会安装下面的方式查找模块：
1. **内置模块**：查找Node.js内置模块（如fs）
2. **社区共享的modules**：查找`node_modules`文件夹下的**package**的文件夹名。即传入的是package名，它会返回该package出口文件（`main`属性指定）的导出对象。
	
	>如果知道该package中某个文件具体的名字，可直接引入，如`packageName/fileName.js`
4. **本地模块**：如果`require`参数名以`./`,`/`或`../`开始，它会在当前目录查找本地模块。


>把package看成是module也行，毕竟只暴露一个文件的导出，在被引入时表现得像本地模块一样

### exports
没啥好讲的，2.1.1小节讲完了。

### import ?

`exports`的内容能够被`import`吗? 经测试, 能! 

为什么? 这好像是webpack提供的模块加载功能, 允许多种加载方式.

## package
一个package包含多个module，使用package.json记录该package关键的信息。

package.json一些字段如下：
* `main`：指定该package的入口文件，当`require(packageName)`时，会导出package入口文件的`exports`。
* `scripts`：定义脚本命令，通过`npm run`来执行。
* `dependencies`：该package依赖的其他包
* `devDependencies`：开发时需要的依赖包。

package安装有全局安装(`npm install --global`)和局部安装（默认）两种方法。
* **全局安装**：package会被安装到`/usr/lib/node_modules`或`/usr/local/lib/node_modules`目录下。package如果有可执行脚本的话，会被添加环境变量下，因此可以直接执行它（如`npm`）。package下的模块不能直接`require`全局安装的模块，需要使用`npm link`创建符号链接。
* **局部安装**（默认）：package会被安装到当前package的`node_modules`目录下。如果有可执行文件，会被放入`node_modules/.bin`目录下，需要使用`npx`执行，它会自动在对应文件夹中寻找。

默认下，局部安装时，安装的依赖会被记录在`package.json`文件的`dependencies`字段中，添加选项`--save-dev`可将依赖记入在`devDependencies`字段中。

>之所以可以直接执行Node.js脚本，是因为在linux环境中，解析型脚本在首部添加了`#!/usr/bin/env Interpreter`，它会在执行时使用指定解析器。

# 三 使用
## 安装Node.js
可从[Long Term Support (LTS) version of Node](https://github.com/nodejs/LTS#lts-schedule1)中查看Node目前长期支持版本（LTS），最好选择`v10.15.3`版本的。在[download page][312]中下载Node.js；对于Linux，使用包管理器安装，参考[package manager][313]

我使用的是WSL的Ubuntu，安装命令如下：
```bash
# Using Ubuntu
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
sudo apt-get install -y nodejs
```
安装时，它会自己全局安装npm包，并且该包含有可执行脚本，因此可以使用npm命令

> 在deepin中, 命令安装不成功, 而是通过下载压缩包并配置环境变量安装成功的...

[312]:https://nodejs.org/en/download/
[313]:https://nodejs.org/en/download/package-manager/

## npm init && node
尽管`node`命令可以直接执行js脚本（如`node index.js`），但最好还是使用`npm init`初始化产生`package.json`，它记录包依赖关系，可以在发布时无需附带其他package，使用时再安装。
```bash
npm init # 之后一直回车，使用默认选项
# 或者
npm init -y
```
>使用过程中，对`node_module`文件夹或`package.json`的修改都会产生`package-lock.json`文件，也记录这依赖关系，但它是为了保证重新安装依赖时版本树的一致性而存在的，因为`package.json`使用了语义版本规则。重新安装包（`npm install`）时，会安装`package-lock.json`指定的版本安装。晕=_=
>当`package-lock.json`不存在时，重新安装依赖包，会根据语义版本规则安装：即主版本号一致，尽量安装最高的版本。

## npm install
安装package，分全局和局部安装，如：
```bash
# 全局依赖
$ npm install uglify-js --global
# 局部依赖
$ npm install uglify-js
# 根据配置文件安装所有局部依赖(包括开发依赖)
$ npm install
# 安装局部依赖为开发依赖
$ npm install uglify-js --save-dev
# 安装指定版本package
$ npm install underscore@1.9.0
```
一般依赖包（node_modules）不会放入git仓库中，取出来后，使用`npm install`会自动安装所有的依赖（包括开发依赖包），但依赖的开发依赖包不会被安装（即开发依赖包无传递性）

npm安装package分全局或局部安装，全局安装的package如果有可执行文件（如npm），它会被添加到环境变量中，因此可以直接执行（如`npm install underscore`）；局部安装且有可执行文件（位于`node_modules/.bin/`目录），此时可用npx来执行，它会自动寻找，如`npx cowsay "Hello"`

>安装的package的版本受`package-lock.json`影响

--------

依赖

* 非开发依赖
  * `-S`, `--save`
* 开发依赖: 仅开发时需要的依赖
  * `-D`, `--save-dev`

> 好像必须加选项, `package.json`才会改变

> 有些package安装时会自动编译, 如果环境中没有对应的工具, 将报错.

## npm list
实现已安装依赖形参的关系树
```bash
#全局
$ npm list --global
#全局，但指定深度
$ npm list -g --depth=0
#局部依赖树
$ npm list
```

## npm uninstall
卸载package
```bash
$ npm uninstall underscore
```
> 貌似删除dev依赖, 仍需要加`--save-dev`

## npm update

* 介绍

  更新包，它会同时修改package.json和package-lock.json文件，单独修改package.json是无效的。

* 版本

  ```
  大版本号.小版本号.次版本号
  ```

* 例子

  仅更新次版本号

  ```shell
  npm update underscore
  ```

  指定版本更新, 可以更新小版本号, 非大版本号

  ```shell
  npm update lodash@3.10.* --save
  ```

  大版本号呢? 不知道

## npm search
在npm仓库中搜索package
```bash
$ npm search mkdir
```
## npm run
运行package.json中scripts字段的脚本（shell脚本），如
```json
{
  // ...
  "scripts": {
    "build": "node build.js"
  }
}
```
此时可运行
```bash
$ npm run build
# equivalent to execution
$ node build.js
```
显示所有可运行命令
```bash
$ npm run
```
运行过程：开启新shell，并暂时将`node_modules/.bin`添加到环境变量中，然后运行脚本。

默认值、hooks、缩写，略！

>参考：[https://www.tutorialdocs.com/article/npm-scripts-tutorial.html](https://www.tutorialdocs.com/article/npm-scripts-tutorial.html)

## npm link
一般情况下，全局package不能直接引入，需要npm link创建符号链接。或者直接指定绝对路径。

略

## npx

用于执行命令, 查找顺序如下:

1. 看`PATH`变量下是否存在
2. 在`node_modules/.bin`寻找
3. 在缓存中找
4. 在线下载被执行该命令.

例子:

```bash
npx cowsay
```
如果该命令不存在, 则每次运行时都需下载

> 小命令无所谓, 大命令建议安装后再运行.

## npm

上面的命令都是通过npm执行的，这里介绍的是它的帮助命令，如：
```bash
# 查看npm的使用
npm help
# 查看install命令的使用
npm help install
```

## 缓存

安装过的package会被缓存起来。

## 帮助

```shell
npm help <command>
```

# 其他

## cnpm

* 介绍

  cnpm是npm在国内的镜像版, 即除了使用国内镜像仓库外, 与npm无本质区别. 

  国内镜像速度很快

* 安装

  ```shell
  $ npm install -g cnpm --registry=https://registry.npm.taobao.org
  ```

  或者直接使用npm, 但配置国内镜像, 这里给出别名的配置

  ```shell
  alias cnpm="npm --registry=https://registry.npm.taobao.org \
  --cache=$HOME/.npm/.cache/cnpm \
  --disturl=https://npm.taobao.org/dist \
  --userconfig=$HOME/.cnpmrc"
  ```

* 使用

  npm怎么使用, cnpm就怎么使用, 除了不支持`publish`命令

> 参考http://npm.taobao.org/

## 编译node-sass失败

`install`或`update` `node-sass`时, 会编译该package, 如果运行环境中缺少对应的工具, 将报错.

我在Windows中遇到了 **Microsoft Build Tools** 版本太低的问题, 可考虑以下步骤解决

* 从[Microsoft Build Tools](https://www.microsoft.com/en-us/download/details.aspx?id=48159)中下载构架工具并安装

* 在npm中全局配置使用该工具

  ```shell
  npm config set msvs_version 2015 --global
  ```

  > 避免每次`install`, 需加后缀, 如`npm install [package name] --msvs_version=2015`

> 参考 https://github.com/nodejs/node-gyp/issues/629 

# 参考
* [Getting started with Node.js modules: require, exports, imports and beyond](https://adrianmejia.com/blog/2016/08/12/getting-started-with-node-js-modules-require-exports-imports-npm-and-beyond/)
* [A Beginner’s Guide to npm — the Node Package Manager](https://www.sitepoint.com/beginners-guide-node-package-manager/)
* [Node.js wiki](https://en.wikipedia.org/wiki/Node.js)
* [npm-package.json](https://docs.npmjs.com/files/package.json)
* [Npm Scripts Tutorial In 10 Minutes](https://www.tutorialdocs.com/article/npm-scripts-tutorial.html)
* [Module Wrapper Function](https://www.youtube.com/watch?v=WlWdbtJoCLQ)
* [Difference between “module.exports” and “exports” in the CommonJs Module System](https://stackoverflow.com/questions/16383795/difference-between-module-exports-and-exports-in-the-commonjs-module-system)
* [Difference between a module and a package in Node?](https://stackoverflow.com/questions/20008442/difference-between-a-module-and-a-package-in-node)
* [NodeJS require a global module/package](https://stackoverflow.com/questions/15636367/nodejs-require-a-global-module-package)
* [The npx Node Package Runner](https://flaviocopes.com/npx/)
* [How to use or execute a package installed using npm](https://flaviocopes.com/how-to-use-npm-package/)
* [What's the difference between dependencies, devDependencies and peerDependencies in npm package.json file?](https://stackoverflow.com/questions/18875674/whats-the-difference-between-dependencies-devdependencies-and-peerdependencies)
* [NPM5, What is the difference of package-lock.json with package.json?](https://stackoverflow.com/questions/48456236/npm5-what-is-the-difference-of-package-lock-json-with-package-json)

[https://github.com/nodejs/LTS#lts-schedule1]: 