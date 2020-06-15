[TOC]

# 使用

安装：

```bash
npm i element-ui -S
```

> `-S`表示安装为`dependencies`依赖

全局使用：在入口文件中添加

```javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);
```

> `Vue.use`用于安装插件, 该步目的是为了简化element ui组件的使用, 如避免了在vue中声明`components`组件和引入组件(`import`)的步骤.

# 基础(Basic)

- **介绍**：`el-row`元素表示一行，`el-col`表示一列，一共**24**列
- **列数**：`el-col`的`span`属性设置占用列数
- **间隔**：`el-row`的`gutter`属性设置列间间隔
- **偏移**：`el-col`的`offset`属性设置偏移量
- **对齐**：`el-row`设置`type="fex"后`，`justify`可指定对齐方式。
- **响应式**：`el-col`可设置`xs`,`sm`,`md`,`lg`,`xl`

# 表单(Form)

## Select

- **介绍**：提供下拉菜单供选择。`el-select`元素代表下拉菜单，`el-option`元素代表其中一项。

- **基本使用**：`el-select`上使用`v-model`；`el-option`的`label`属性设置标签，`value`属性设置选中后得到的值。

  > 使用`v-for`指令时务必设置`key`

- **禁用select**：`el-select`添加`disabled`

- **可清除**：添加清除图标，`el-select`添加`clearable`属性

- **多选择**：`el-select`添加`multiple`允许选择多个；默认选择的选型显示成标签模式，添加`collapse-tags`进入压缩模式。

- **自定义选项**：略

- **选项组**：。。。

- 。。。

- **创建新选项**：`el-select`添加`allow-create`属性，并且必须设置`filterable`属性。

  > 一般设置`multiple`允许多选；设置`default-first-option`允许回车选择第一个

## Upload

文件上传控件，需放入子元素，才能触发文件选择。通过`list-type`属性，可设置已上传文件的列表样式。如果不满意，可自行禁止列表`show-file-list`，然后自定义子元素。详细见文档的例子User avatar upload。

- 常用属性

  - action（必须）：请求url

  - name：上传文件的参数名，默认`file`

  - auto-upload：是否自动上传，默认`true`

    ------

  - multiple：是否允许一次上传多个文件，即一次选择多个文件。默认`false`

  - drag：是否允许拖拽上传，默认`false`

  - limit：最大上传文件个数。

    - on-exceed：超过limit时的回调函数。

    ------

  - show-file-list：是否显示已上传文件列表，默认`true`

  - list-type：文件列表类型。

    - text（默认）：![1559207926245](.Element%20UI/1559207926245.png)
    - picture：![1559207980004](.Element%20UI/1559207980004.png)
    - picture-card：![1559208082581](.Element%20UI/1559208082581.png)

  - file-list：默认已上传文件，默认`[]`

    ------

  - headers：请求头字段

  - data：请求的额外数据

  - with-credentials：是否携带凭证（cookies），用于跨域。

- 一堆回调函数：略

- Slot

  - trigger：触发文件对话框的内容。貌似没有指定时，触发内容为默认slot
  - tip：提示内容

- 方法：比如`submit`，手动上传时使用；等等；

## Form

### 基础

- **介绍**

  * `el-form`表示表单，`el-form-item`表示表单项（包裹`input`元素）。
  * 能够提供收集、**验证**（主要使用原因）和提交数据的功能。

- **基本使用**

  * `el-form`的`model`设置表单数据，
  * `label-width`：设置标签宽度；
  * `el-form-item`的`label`属性设置标签名

- **inline form**

  `el-form-item`默认为`block`，`el-form`的`inline`属性将之设置为`inline-block`

- **标签位置**

  `el-form`的`label-position`设置标签位置，值`left/right/top`

- **验证**

  `el-form`的`rules`属性设置所有规则，`el-form-item`的`prop`属性设置某个规则对应的键值。

### 校验

例子:

```javascript
rules : {
   name: [
      { required: true, message: '请输入活动名称', trigger: 'blur' },
      { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
   ],
   region: [
      { required: true, message: '请选择活动区域', trigger: 'change' }
   ],
   date1: [
      { type: 'date', required: true, message: '请选择日期', trigger: 'change' }
   ],
   date2: [
      { type: 'date', required: true, message: '请选择时间', trigger: 'change' }
   ],
   type: [
      { type: 'array', required: true, message: '请至少选择一个活动性质', trigger: 'change' }
   ],
   resource: [
      { required: true, message: '请选择活动资源', trigger: 'change' }
   ],
   desc: [
      { required: true, message: '请填写活动形式', trigger: 'blur' }
   ],
   age: [
      {
         validator: (rule, value, callback) => {
            if (!value) {
               return callback(new Error('年龄不能为空'));
            }
            setTimeout(() => {
               if (!Number.isInteger(value)) {
                  callback(new Error('请输入数字值'));
               } else {
                  if (value < 18) {
                     callback(new Error('必须年满18岁'));
                  } else {
                     callback();
                  }
               }
            }, 1000);
         },
         trigger: 'blur'
      }
   ]
}
```

常用的校验规则声明字段:

* **字段类型**

  Indicates the `type` of validator to use. Recognised type values are:

  * `string` (default): Must be of type `string`. `This is the default type.`
  * `number`: Must be of type `number`.
  * `boolean`: Must be of type `boolean`.
  * `method`: Must be of type `function`.
  * `regexp`: Must be an instance of `RegExp` or a string that does not generate an exception when creating a new `RegExp`.
  * `integer`: Must be of type `number` and an integer.
  * `float`: Must be of type `number` and a floating point number.
  * `array`: Must be an array as determined by `Array.isArray`.
  * `object`: Must be of type `object` and not `Array.isArray`.
  * `enum`: Value must exist in the `enum`.
  * `date`: Value must be valid as determined by `Date`
  * `url`: Must be of type `url`.
  * `hex`: Must be of type `hex`.
  * `email`: Must be of type `email`.
  * `any`: Can be any type.

* **是否必须**

  The `required` rule property indicates that the field must exist on the source object being validated.

* **匹配模式**

  The `pattern` rule property indicates a regular expression that the value must match to pass validation.

* 值域范围

  A range is defined using the `min` and `max` properties. For `string` and `array` types comparison is performed against the `length`, for `number` types the number must not be less than `min` nor greater than `max`.

* 长度

  To validate an exact length of a field specify the `len` property. For `string` and `array` types comparison is performed on the `length` property, for the `number` type this property indicates an exact match for the `number`, ie, it may only be strictly equal to `len`.

  > If the `len` property is combined with the `min` and `max` range properties, `len` takes precedence.

* 内容不可空白

  `whitespace:true` , 仅适用于`string`类型

* 转化

  在校验前转化字段值, 且覆盖原有值

  ```javascript
    name: {
      type: "string",
      required: true, pattern: /^[a-z]+$/,
      transform(value) {
        return value.trim();
      }
    }
  ```

* **提示消息**

  校验后不符合时的提示消息

  ```javascript
  {name:{type: "string", required: true, message: "Name is required"}}
  ```

* **校验器**

  提供自定义校验, 更为灵活, 分为同步和异步校验器.

  下面给出同步校验的例子

  ```javascript
  field: {
      validator(rule, value, callback) {
          return value === 'test';
      },
      message: 'Value is not equal to "test".',
  }
  ```

  下面给出Element UI中的使用

  ```javascript
  ...
  
  var validatePass2 = (rule, value, callback) => {
      if (value === '') {
          callback(new Error('请再次输入密码'));
      } else if (value !== this.ruleForm.pass) {
          callback(new Error('两次输入密码不一致!'));
      } else {
          callback();
      }
  };
  
  ...
  rules: {
      pass: [
          { validator: validatePass, trigger: 'blur' }
      ],
      checkPass: [
          { validator: validatePass2, trigger: 'blur' }
      ],
      age: [
         { validator: checkAge, trigger: 'blur' }
      ]
  }
  
  ...
  ```

  > 注意, `callback`必须调用, 否则会抛出异常.

* **触发器**

  什么时候校验的触发器声明.

  已知两种:

  * `blur` 字段失去聚焦时
  * `change` 字段改变时

  例子

  ```javascript
  { required: true, message: '请输入邮箱地址', trigger: 'blur' },
  { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
  ```

# 通知(Notice)

## Alert(警告)

在**页面中**展示重要的**提示**信息.

可以有标题, 辅助性文字, 是否可关闭, 是否显示图标, 主题颜色等属性.

## Notification(通知)

**悬浮**出现在页面**角落**，显示**全局**的**通知提醒**消息。

Element会为`Vue.prototype`添加了全局方法`$notify`, 通过该方法, 并传入一个选项对象, 即可产生通知消息.

可以设置标题, 说明文字, 显示时间, 是否可关闭等属性.

## Message(消息提示)

常用于**主动**操作后的**反馈提示**。与 Notification 的区别是后者更多用于系统级通知的被动提醒。

> 个人看来, 就是显示位置不同而已

与`Notification`基本一致, 同样在vue实例中添加了`$message`方法.

可以设置消息文字, 主题, 显示时间, 是否可关闭等属性.

## MessageBox

**模拟系统**的消息提示框而实现的一套**模态对话框**组件，用于**消息提示**(`$alert`)、**确认消息**(`$confirm`)和**提交内容**(`$prompt`)。这些方法底层都基于`$msgbox`.

> 如果要展示更复杂的内容, 则使用对话框Dialog

支持`Promise`; 默认不区分取消和关闭, `promise`的`reject`回调和`callback`回调的参数都为`cancel`, 可修改, 此时分别为`cancel`和`close`.

# 其他(Others)

## Dialog(对话框)

适合制作更复杂的对话框, 用于告知用户并承载相关操作.

对话框与Notice不一样, 需要在html模板中定义并隐藏, 之后控制它是否显示与否.

可设置标题`title`, 是否显示`visible`, 内容`default slot`, 操作区内容`footer slot`等等.

## Tooltip(文本提示)

常用于展示鼠标 hover 时的提示信息。

`el-tooltip`元素包裹在要提示的元素上, `content`提供提示信息, `placement`设置显示位置.

> 其中`placement`值形式为`方向[-对齐位置]`; 方向可选`top`、`left`、`right`、`bottom`; 对齐位置可选`start`, `end`

