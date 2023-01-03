# 网页

## html

html 是浏览器需要的一种格式文本

### 标签

- `<h1>`~`<h2>` 标题标签
- `<p>` 段落标签
- `<br />` 换行标签
- `<strong></strong>` 加粗
- `<em></em>` 倾斜
- `<div></div>` 块级元素
- `<span></span>` 行内元素
- `<img src="图像URL" />` 图片标签
- `alt` 替换文本， 和 src 放一起，图片列开显示
- `title` 提示文本，和 src 放一起，显示图片信息
- `width=" "` 设置图片宽度
- `height=" "` 设置图片高度
- `border=" "` 设置图片边框
- `<a href="跳转目标" target="目标窗口的弹出方式">文本或图像</a>` 超链接标签
  target 中`_self`指再当前窗口打开
  `_blank`指再新窗口打开
  锚点链接也可以使用
- `<!-- -->`注释标签 ctrl+/快速打出

- 表格

        <table>
            <tr> <th></th> <th></th> <th></th></tr>
            <tr> <td></td> <td></td> <td></td></tr>
            <tr> <td></td> <td></td> <td></td></tr>
        </table>

- `<form></form>` 表单标签
- `<input type="属性值" name=" " value=" " />` 表单元素
  `text` 文本框
  `password` 密码框
  `radio` 单选按钮
  `checkbox` 复选框

- `<select> </select>`下拉表单元素

- `<textarea>`文本域标签

## css

- 类选择器

        <style> .red{color:red} </style>
        <div class="red">刘德华</div>

- id 选择器

        <style> #red{color:red} </style>
        <div class="red">刘德华</div>
        id选择器智能给一个对象使用

- `font-family`字体设置
- `text-align` 文本对齐 `centre,left,right`
- `text-indent：2em` 文本缩进，这里`2em`表示缩进两个单位大小的距离

- `line-height` 设置行间距

**行内样式表**
可以直接在标签内部引入 css,即在标签内写 style
`<div style="color:red font-size:12px">输入的文本</div>`

**外部样式表**
新建一个`.css`文件，然后再`.html`文件中去引用
_代码_ `<link rel="stylesheet" href="css文件路径">`

## javascrip

`js` 操作中多用单引号

> 行内式

                <input type="button" value="唐伯虎"onclick="alert('秋香')" />

> 内嵌式

                <script>
                        alert('沙漠');
                </script>

> 外部式

                写好一个.js文件
    
                <script src="路径名"></script>

> 输入输出

- `prompt`输入框
- `alert`弹出警示框
- `console.log`控制台输出

                <script>
                prompt('请输入你的年龄');
                alert('计算结果是');
                console.log('我是程序员能看到的');
                </script>

> `var`变量，是一个动态的数据类型

- `length` 可以获取字符串的长度+

> 数据类型的转换

- `string(num)`强制转换 字符->数字
- `parseInt(string)` 数字->字符
