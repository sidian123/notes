[TOC]

# 一 介绍
Iconfont是阿里打造的图标平台，拥有大量的图片，提供将图片转化为**字体**供前端人员使用的功能，十分方便。
>国外也有类似的图标字体库，如[Font Awesome][1]

# 二 使用方法
那如何在vue的单文件组件（`.vue`）中使用呢？下面介绍多种方法，各有优缺点。在使用之前，先添加图标到**项目**中。

## 方法一
1. 进入项目，生成在线链接，如：`//at.alicdn.com/t/font_1199749_deo2fjl10md.css`
2. 在vue组件的`style`元素中，使用`@import`引入在线css
	```css
	@import "//at.alicdn.com/t/font_1199749_deo2fjl10md.css";
	```
3. `template`元素中使用方式如下：
    ```html
    <i class="iconfont icon-xxx"></i>
    ```

**缺点**：这种方式用起来不太方便，原因有三：

1. 每次项目中收藏了新的图标后，都需要生产新的css链接，需要改`@impot "url"`
2. 在多处组件的`style`元素中引入，会引入多次，造成css泛滥。
3. 如果在顶层组件（如App.vue）中的`style`元素中引入，会造成样式优先级问题，不能方便修改样式。

## 方法二
解决思路是：下载到本地，在`script`元素中导入，能够解决上述所有问题。步骤如下：
1. 首先下载项目，解压到`assets`目录下，重命名`iconfont`，如图所示：
![在这里插入图片描述](.Vue%E4%B8%8EIconfont/2019052011461690.png)
2. 在需要的组件的`script`元素中引入：
	```javascript
	import "@/assets/icon/iconfont/iconfont.css"
	```
3. 然后使用

**优点**：
* 每次添加新的图标后，下载项目覆盖原来文件夹，能够解决方法一的问题1；
* `script`元素中引入，同时也进入到了webpack的依赖图，不会造成重复引入，解决方法一的问题2、3

**缺点**：

1. 每次项目中新添图标时，都要下载并覆盖，麻烦。
2. 为了使用图标，还要单独引入一次css文件，费劲。

## 方法三（推荐）
我们一般会抽取公共的、有用的代码，如我会放在`src/Utils/Utils.vue`中，其他地方引入时能够一次性引入Javascript和css代码，很是方便。并且多次引入`Utils.vue`，打包时最终只会引入一次。

在这里，我们采用方式一的方式，获取项目在线链接，在`style`元素中引入。下面给出一个含有公共JS、CSS和图标的`.vue`文件：
```html
<template>

</template>

<script>
    export default {
        /**
         * 得到URL上的参数值
         * @param parameterName 参数名
         * @returns {*} 返回参数值(string)或null
         * @constructor
         */
        getUrlParameter(parameterName) {
            var result = null,
                tmp = [];
            location.search	//search字段返回查询url的查询参数部分，如?paramter=value&parameter2=value2
                .substr(1)	//去掉前面的问号（?）
                .split("&")	//以&为界限，分离出含有参数键值对的数组
                .forEach(function (item) {  //对每个数组元素操作，item为当前元素
                    tmp = item.split("=");	//以=为分解符，将参数键值对分离
                    if (tmp[0] === parameterName) result = decodeURIComponent(tmp[1]);//对参数值解码
                });
            return result;
        }
    }
</script>


<style lang="scss">
    @import url("//at.alicdn.com/t/font_1199749_t172xat03rn.css");
    .btn{
        padding:.375rem .75rem;
        border-radius:5px;
        border:none;

        &:hover,&:focus{
            outline: none;
            box-shadow: 0 0 3px 0 black;
        }

        &-primary{
            background-color: #007bff;
            color: white;
        }
    }
</style>
```
完美！！解决上述所有问题！！
# 参考
[帮助-使用](https://www.iconfont.cn/help/detail?spm=a313x.7781069.1998910419.d0091c141&helptype=code)

[1]:http://fontawesome.dashgame.com/
[2]:https://www.iconfont.cn/home/index?spm=a313x.7781069.1998910419.2