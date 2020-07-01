# 基础

## 安装

```shell
npm isntall --save echarts
```

## 使用

指定挂载的DOM元素和配置

```javascript
echarts.init(document.getElementById('main')) // 挂载元素
    .setOption(option); // 设置描述对象
```

## 配置

* `titile`

  标题

  ```
  title: {
      text: '第一个 ECharts 实例'
  }
  ```

* `tooltip`

  提示信息

  ```
  tooltip: {},
  ```

* `legend`

  图例组件，如柱状图类型

* `xAxis`

  X轴的项

* `yAxis`

  Y轴的项

* `series`

  表示一系列支持图表展示的数据. 所以`series`包含一下关键信息

  * 图表类型
  * 一系列数据
  * 其他映射数据到图表的参数

  ```javascript
  series: [{
      name: '销量',  // 系列名称
      type: 'bar',  // 系列图表类型
      data: [5, 20, 36, 10, 10, 20]  // 系列中的数据内容
  }]
  ```

  其中`type`决定图标类型

## Demo

```html
  <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>ECharts</title>
        <!-- including ECharts file -->
        <script src="echarts.js"></script>
    </head>
    <body>
        <!-- prepare a DOM container with width and height -->
        <div id="main" style="width: 600px;height:400px;"></div>
        <script type="text/javascript">
            // based on prepared DOM, initialize echarts instance
            var myChart = echarts.init(document.getElementById('main'));
  
            // specify chart configuration item and data
            var option = {
                title: {
                    text: 'ECharts entry example'
                },
                tooltip: {},
                legend: {
                    data:['Sales']
                },
                xAxis: {
                    data: ["shirt","cardign","chiffon shirt","pants","heels","socks"]
                },
                yAxis: {},
                series: [{
                    name: 'Sales',
                    type: 'bar',
                    data: [5, 20, 36, 10, 10, 20]
                }]
            };
        
            // use configuration item and data specified to show chart
            myChart.setOption(option);
        </script>
    </body>
    </html>
```

# 进阶

## 基本概念





# 参考

[ECharts](https://echarts.apache.org/zh/option.html)



