### Windows右键新建菜单中添加md文件

1. 创建文件a.txt，添加如下内容：

   Windows Registry Editor Version 5.00
   [HKEY_CLASSES_ROOT\.md\ShellNew]
   "NullFile"=""
   "FileName"="template.md"

2. 文件改为a.reg

3. 双击运行

### 常用快捷键

| 快捷键 | 作用     | 快捷键 | 作用       |
| ------ | -------- | ------ | ---------- |
| Ctrl+1 | 一阶标题 | Ctrl+B | 字体加粗   |
| Ctrl+3 | 三阶标题 | Ctrl+I | 字体倾斜   |
| Ctrl+3 | 六阶标题 | Ctrl+U | 下划线     |
| Ctrl+T | 创建表格 | Ctrl+K | 创建超链接 |
| Ctrl+F | 搜索     | Ctrl+H | 搜索并替换 |

### 块元素

#### 目录

`[toc]`

[TOC]

#### 标题级别

\#一级标题 快捷键为 Ctrl + 1

\##二级标题 快捷键为 Ctrl + 2

......

\###六级标题 快捷键为 Ctrl + 6

#### 引用

\> + 空格 + 引用文字

#### 代码块

\`代码\`，效果：`效果`

\``` + 回车，创建代码块

#### 脚注

在需要添加上标的文字后面加上`[^脚注标记]`，`[^]:`为脚注注解

文字[^脚注]

[^脚注]:这是脚注

#### 下标上标高亮

需要在文件->偏好设置中打开下标、上标、和高亮选项。

`文字^上标^`：文字^上标^,

`文字~下标~`：文字~下标~

`==高亮==`：==高亮==

#### 分割线

`***`或者`---`

---

***

#### 斜体加粗

`*斜体*`*斜体*

`**加粗**`**加粗**

`***斜体加粗***`***斜体加粗***

#### 下划线

通过`<u>下划线的内容</u>` 或者 快捷键`Ctrl + U`可实现下划线

<u>下划线的内容

#### 删除线

`~~删除线~~`：~~删除线~~

#### 字体、字号与颜色

`<font color=#00ffff size=3 face="黑体">黑体字</font>`

<font color=#00ffff size=3 face="黑体">黑体字</font>

#### emoji表情

emoji表情使用:EMOJICODE:的格式，详细列表可见 <https://www.webpagefx.com/tools/emoji-cheat-sheet/>

#### 锚点

`[内连接](#目录)`

[内连接](#目录)

#### HTML

支持HTML标签

#### 绘图

<https://www.jianshu.com/p/7ddbb7dc8fec>

### 中文文案排版指北

#### 空格

- 中英文之間需要增加空格
- 中文與數字之間需要增加空格

- 数字与单位之间需要增加空格

  ​	度／百分比與數字之間不需要增加空格

- 全形标点与其他字符之间不加空格

#### 标点符号

- 不重复使用标点
- 全形标点和半形标点
- 使用全形中文标点
- 数字使用半形字符
- 遇到完整的英文整句、特殊名詞，其內容使用半形標點

#### 名词

- 专有名字正确使用大小写
- 不使用不地道的缩写