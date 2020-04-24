* 安装

  ```shell
  npm isntall --save echarts
  ```

* 使用

  指定挂载的DOM元素和配置

  ```javascript
  echarts.init(document.getElementById('main')).setOption(option);
  ```

* 配置

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

    系列列表

    ```javascript
    series: [{
        name: '销量',  // 系列名称
        type: 'bar',  // 系列图表类型
        data: [5, 20, 36, 10, 10, 20]  // 系列中的数据内容
    }]
    ```

    其中`type`决定图标类型

  

  * 参考

    [ECharts](https://echarts.apache.org/zh/option.html)

