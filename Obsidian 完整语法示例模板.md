# 一、基础文本格式

## 1. 标题（六级）

```markdown
# 一级标题（最大）
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题（最小）
```

## 2. 基础文字样式

```markdown
**粗体文本**
*斜体文本*
***粗斜体文本***
~~删除线文本~~
<u>下划线文本</u>
==高亮文本==
^上标文本^（示例：2^3^=8）
~下标文本~（示例：H~2~O）
```

## 3. 行内代码与注释

```markdown
行内代码：`print("Hello Obsidian")`
单行注释：%% 这是单行注释，Obsidian中默认隐藏 %%
多行注释：
%%
这是多行注释
可换行，预览模式完全隐藏
%%
```

# 二、列表与引用

## 1. 有序列表

```markdown
1. 有序列表项1
2. 有序列表项2
   3. 嵌套有序列表项
   4. 嵌套有序列表项
5. 有序列表项3
```

## 2. 无序列表

```markdown
- 无序列表项1（短横线）
* 无序列表项2（星号）
  - 嵌套无序列表项
  - 嵌套无序列表项
+ 无序列表项3（加号）
```

## 3. 任务列表（待办）

```markdown
- [ ] 未完成任务
- [x] 已完成任务
- [-] 取消的任务
```

## 4. 引用块与标注（Callout）

```markdown
# 基础引用块
> 一级引用
>> 二级嵌套引用
>>> 三级嵌套引用

# 标注（Callout）
> [!note] 可选标题
> 基础标注，支持**所有MD语法**
> [!tip]+ 默认展开的标注
> 内容1：[[内部链接]]
> 内容2：![图片嵌入](https://picsum.photos/200)
> [!warning]- 默认收起的标注
> 点击标题可展开/收起
> [!custom] 自定义标注（需配CSS片段）
> 自定义颜色/图标
```

# 三、链接与跳转（核心特性）

## 1. 内部链接

```markdown
# 跳转到其他笔记
[[笔记名称]]
[[笔记名称|自定义显示文字]]

# 跳转到笔记内标题
[[笔记名称#二级标题]]
[[#当前笔记的标题]]

# 跳转到笔记内指定块
[[笔记名称^块ID]]
[^块ID]（右键行→复制块链接自动生成）

# 嵌入笔记
![[笔记名称]]
![[笔记名称#标题]]
![[笔记名称^块ID]]
```

## 2. 外部链接与邮箱

```markdown
# 外部网页链接
[Obsidian官方网站](https://obsidian.md)
<https://obsidian.md>

# 邮箱链接
[发送邮件](mailto:xxx@xxx.com)
<xxx@xxx.com>
```

## 3. 标签

```markdown
# 基础标签
#标签1
#分类/子分类（嵌套标签）

# 示例
#工作/项目A #学习/Markdown #生活/旅行
```

# 四、媒体嵌入

## 1. 图片嵌入

```markdown
# 网络图片
![图片描述](https://picsum.photos/300/200)
![图片描述](https://picsum.photos/300/200 =200x150)（自定义尺寸）

# 本地图片（推荐相对路径）
![头像](./images/avatar.png)
![封面](C:/Obsidian/库名/images/cover.jpg)（绝对路径）

# 图片嵌入为链接
[!["图片描述"](./images/avatar.png)](https://obsidian.md)
```

## 2. 音频/视频嵌入

```markdown
# 音频嵌入
![[audio.mp3]]（本地）
![[https://xxx.com/audio.mp3]]（网络）

# 视频嵌入
![[video.mp4]]（本地）
![[https://xxx.com/video.mp4]]（网络）
```

# 五、代码块与公式

## 1. 代码块（语法高亮）

````markdown
# 无语法高亮
```
print("Hello Obsidian")
for i in range(10):
    print(i)
```

# 有语法高亮（Python）
```python
def add(a, b):
    return a + b
print(add(1,2))
```

# 有语法高亮（JavaScript）
```javascript
const num = 10;
console.log(num * 2);
```
````

## 2. 数学公式（LaTeX）

```markdown
# 行内公式
勾股定理：$a^2 + b^2 = c^2$

# 块级公式
$$
E=mc^2
$$

$$
\sum_{i=1}^n i = \frac{n(n+1)}{2}
$$
```

# 六、表格与分隔线

## 1. 表格


```markdown
# 基础表格
| 姓名 | 年龄 | 职业 |
| ---- | ---- | ---- |
| 张三 | 25   | 程序员 |
| 李四 | 30   | 设计师 |

# 对齐表格
| 左对齐 | 居中对齐 | 右对齐 |
| :----- | :------: | -----: |
| 内容1  | 内容2    | 内容3  |
```

## 2. 分隔线

```markdown
---（推荐）
***
___
```

# 七、脚注与定义列表

## 1. 脚注

```markdown
Obsidian是一款优秀的本地笔记软件[^1]，支持多种语法扩展[^2]。

[^1]: 基于Markdown语法，本地存储，无云端依赖。
[^2]: 支持标注、块链接、嵌套标签等独家扩展。
```

## 2. 定义列表

```markdown
Markdown
: 一种轻量级标记语言，易写易读。
Obsidian
: 基于Markdown的本地知识库软件，核心特性为双向链接。
双向链接
: Obsidian的核心功能，可实现笔记间的网状关联。
```

# 八、Obsidian 独家高级语法

## 1. 别名（Frontmatter 内设置）


```markdown
---
aliases: [别名1, 别名2, 自定义名称]
---
# 笔记原标题
这是笔记内容...
```

## 2. 元数据（Frontmatter）

```markdown
---
title: Obsidian语法模板
date: 2026-01-27
tags: [Obsidian, Markdown, 笔记]
aliases: [Obsidian语法大全, MD语法模板]
author: 自定义
---
# 笔记正文
元数据为笔记添加属性，支持搜索筛选。
```

## 3. 嵌入文件（PDF/Excel/Word）

```markdown
![[文档.pdf]]
![[表格.xlsx]]
![[文档.docx]]
```

# 九、实用快捷语法

```markdown
1. 快速插入当前时间：Ctrl+Shift+I（Win）/Cmd+Shift+I（Mac）
2. 快速插入空行：Shift+Enter
3. 快速格式化：Ctrl+B（粗体）/Ctrl+I（斜体）/Ctrl+K（链接）
4. 快速创建笔记：[[新笔记名称]]
5. 快速搜索：Ctrl+O（Win）/Cmd+O（Mac）
```