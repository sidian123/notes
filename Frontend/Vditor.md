* 安装

  ```shell
  npm install vditor --save
  ```

* 引入

  ```javascript
  import Vditor from 'vditor'
  import "~vditor/src/assets/scss/classic" // 或者使用 dark
  
  const vditor = new Vditor(id, {options...})
  vditor.focus()
  ```

* 使用

  ```html
  <template>
      <div>
        <div id="editor"></div>
      </div>
  </template>
  <script>
      import Vditor from "vditor"
      import "vditor/src/assets/scss/classic.scss"
      export default {
          name: "Test",
          mounted(){
              const vditor = new Vditor("editor");
              vditor.focus()
          }
      }
  </script>
  ```

  > 不用使用`textarea`元素作为Vditor的容器, 即`id`的元素最好为`div`

  * 使用形式

    ```javascript
    const vditor = new Vditor(id, {options...})
    ```

    























> 参考[一款浏览器端的 Markdown 编辑器](https://hacpai.com/article/1549638745630)