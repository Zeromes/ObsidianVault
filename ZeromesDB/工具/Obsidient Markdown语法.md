# 标题

## 语法

``` markdown
# 标题1

## 标题2

###### 标题6
```
支持6个标题层级。

# 引用

## 语法

``` markdown
> 这是一段引用
```
## 效果

> 这是一段引用

# 列表

## 语法

``` markdown
## 无序列表
*   Red
*   Green
*   Blue

## 有序列表
1.  Red
2. 	Green
3.	Blue
```
按Tab可以调整缩进

## 效果

### 无序列表
* Red
	* Green
* Blue

### 有序列表
1. Red
	1. Green
2. Blue

# 代办列表

## 语法

``` markdown
- [ ] a task list item
- [ ] list syntax required
- [ ] normal **formatting**, @mentions, #1234 refs
- [ ] incomplete
- [x] completed
```

## 效果

- [ ] a task list item
- [ ] list syntax required
- [ ] normal **formatting**, @mentions, #1234 refs
- [ ] incomplete
- [x] completed

# 代码块

## 语法

````markdown
```js
function test() {
  console.log("notice the blank line before this function?");
}
```
````

## 效果

```js
function test() {
  console.log("notice the blank line before this function?");
}
```

# 数学公式

## 语法

支持LaTeX数学公式语法，语法详情见网页：[通用 LaTeX 数学公式语法手册 - UinIO.com 电子技术博客](http://www.uinio.com/Math/LaTex/)
markdown中，使用双$号包裹数学公式的代码内容。
示例：
```markdown
$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
```

## 效果

$$
\mathbf{V}_1 \times \mathbf{V}_2 =  \begin{vmatrix}
\mathbf{i} & \mathbf{j} & \mathbf{k} \\
\frac{\partial X}{\partial u} &  \frac{\partial Y}{\partial u} & 0 \\
\frac{\partial X}{\partial v} &  \frac{\partial Y}{\partial v} & 0 \\
\end{vmatrix}
$$
# 列表

## 语法

``` markdown
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```
在分隔栏中加入引号，可控制该栏的对其方式

## 效果

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |

# 链接

## 语法


``` markdown
一般链接：
[Zeromes的博客](http://www.zeromes.cn/)

图片链接（任何链接都可以在前加感叹号来使链接内容之间显示在文本中）：
![火车](./attachments/火车.png)

双向链接指向文章：
[[Typora Markdown语法]]

双向链接指向具体标题：
[[Typora Markdown语法#Links]]

双向链接指向具体段落：
[[Typora Markdown语法#^4276e8]]

双向链接指向自定义段落编号：
[[Typora Markdown语法#^linkDelimitation]]

双向链接自定义显示名：
[[Typora Markdown语法#^linkDelimitation|链接划分]]
```

## 效果

一般链接：
[Zeromes的博客](http://www.zeromes.cn/)

图片链接：
![火车](./attachments/火车.png)

双向链接指向文章：
[[Typora Markdown语法]]

双向链接指向具体标题：
[[Typora Markdown语法#Links]]

双向链接指向具体段落：
[[Typora Markdown语法#^4276e8]]

双向链接指向自定义段落编号：
[[Typora Markdown语法#^linkDelimitation]]

双向链接自定义显示名：
[[Typora Markdown语法#^linkDelimitation|链接划分]]

# 字体修饰

## 语法

``` markdown
*斜体*
**加粗**
这是`内嵌代码`。
~~删除线~~
<u>下划线</u>
```

## 效果

*斜体*
**加粗**
这是`内嵌代码`。
~~删除线~~
<u>下划线</u>

