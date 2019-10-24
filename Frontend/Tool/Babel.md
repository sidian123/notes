目的: 将es6语法转化为es5语法, 避免兼容性问题, 尤其是可恶的ie浏览器

步骤如下

1. 安装

   ```shell
   npm install --save-dev @babel/core @babel/cli @babel/preset-env
   npm install --save @babel/polyfill
   ```

   > * `@babel/core`含核心功能, 如转换ES5+语法的转换器; 
   > * `@babel/cli`为使用Babel功能的一个CLI工具; 
   > * `@babel/preset-env`一个默认的配置, 指示什么情况下使用什么语法转换器, 默认全部使用. 需在配置文件中指定该设置.
   > * `@babel/polyfill`模拟ES5+所需的环境, 如某些浏览器不支持一些函数, ` Promise`等.

2. 配置文件`.babelrc`

   ```json
   {
       "presets": ["@babel/preset-env"]
   }
   ```

   > 没有特定为浏览器指定规则, 因此所有语法都将转化为es5, 没有优化可言

3. 运行

   ```shell
   npx babel src --out-dir target
   ```

   > 将`src`下的Js文件编译到`target`目录下

4. 配置命令, 在`package.json`的中添加

   ```json
   "scripts": {
       "build":" npx babel src --out-dir target"
   }
   ```

   之后, 可以通过`npm run build`来运行


> 参考
>
> [Babel Usage Guide]( https://babeljs.io/docs/en/usage ): 一篇足以

