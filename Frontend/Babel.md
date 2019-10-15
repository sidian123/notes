目的: 将es6语法转化为es5语法, 避免兼容性问题, 尤其是可恶的ie浏览器

步骤如下

1. 安装

   ```shell
   npm install --save-dev @babel/core @babel/cli @babel/preset-env
   npm install --save @babel/polyfill
   
   npm install --save-dev @babel/preset-env
   ```

2. 配置文件`.babelrc`

   ```json
   {
       "presets": ["@babel/preset-env"]
   }
   ```

   > 没有特定为浏览器指定规则, 因此所有语法都将转化为es5, 没有优化可言

3. 运行

   ```shell
   npx babel src --out-dir lib
   ```

   > 将`src`下的Js文件编译到`lib`目录下

4. 配置命令, 在`package.json`的中添加

   ```json
   "scripts": {
       "build":" npx babel src --out-dir lib"
   }
   ```

   之后, 可以通过`npm run build`来运行

   

   