* 获取时间戳

  ```python
  import time
  time.time()
  ```

* 删除文件

  ```python
  import os
  os.remove('aaa.csv')
  ```


# Excel

* 安装依赖

  ```shell
  pip install pandas
  ```

* 读取文件 & 配置参数

  默认读取第一个sheet, 以第一行为header.

  ```python
  import pandas as pd
  
  df = pd.read_excel (r'filename.xlsx')
  print (df)
  ```

  > 返回的`DataFrame`

* 遍历

  ```python
  for index, row in df.iterrows():
      print(index, row)
  ```

  > 这里的`index`的0指向excel中的第二行.

  `row`可以`dict`的方式访问每一列的值, 如

  ```python
  for index, row in df.iterrows():
      print(index, row['delay'],row['distance'],row['origin'])
  ```

* 参考

  * [read_excel](https://pandas.pydata.org/docs/reference/api/pandas.read_excel.html#pandas.read_excel)
  * [DataFrame](https://pandas.pydata.org/docs/reference/frame.html)
  * [How To Loop Through Pandas Rows? or How To Iterate Over Pandas Rows?](https://cmdlinetips.com/2018/12/how-to-loop-through-pandas-rows-or-how-to-iterate-over-pandas-rows/)

