
## 安装
### 文件下载
[obsidian下载安装官网](https://blog.csdn.net/qq_41821678/article/details/119870496) ：包括window、linux(snap)、linux(appimage)百度云

 其实ubuntu环境下有三种安装包：
	- 从这里进可以看到 deb 包：https://bsidian.md/download

我的从官网下好的：
	- 0.14.2 linux(appimage)：链接: https://pan.baidu.com/s/1Yusykvc-eigLJ8TlQLwTIw 提取码: 4c3d 

我之前安装的Appimage，但是没有科学上网安装不了插件，换成了deb安装，推荐deb，虽然版本落后一些。

### 文件安装
#### Deb 格式
该文件只需要直接双击就可以安装，或者
```
sudo dpkg -i ./obsidian_0.13.31_amd64.deb
```

#### AppImage 格式安装
参考[[Appimage格式文件安装]]

## 插件
### 插件的安装
系统：ubuntu 20.04

使用deb包安装有插件文件夹，AppImage不行，snap包没试过
1. deb安装好之后打开一个库，可以新建一个文件夹用来放obsidian笔记
2. 打开设置，找到第三方插件，关闭安全模式
3. 然后这个文件夹下有隐藏文件夹.obsidian，里面有plugins文件夹
4. https://gitee.com/whghcyx/obsidian-plugin - 大部分插件下载
5. 下载后是zip，解压到plugins文件夹里面，重启就可以了
### 插件推荐
1. https://mp.weixin.qq.com/s/MEPva7Os_nOyl1vgTBZTwQ - 二一的笔记推文
2. https://zhuanlan.zhihu.com/p/353449575 - 比较全面，有链接
3. https://client.sspai.com/post/72426 - 几个比较高级的插件
4. https://zhuanlan.zhihu.com/p/368487154 - 更全面的推荐，有链接和介绍

### 一些插件的说明：
##### 1.[Advanced table](https://github.com/tgrosinger/advanced-tables-obsidian)
表格插件，可插入excel公式
To create a table, create a single `|` character, then type the table's first heading and press Tab. Continue entering headings and pressing Tab until all the headings are created. Press Enter to go to the first row. Continue filling cells as before, and press Enter again for each new row.

When a cursor is in a markdown table...

When a cursor is in a markdown table...

| Hotkey | Action |
| ------ | ------ |
|Tab|Next Cell|
|Shift + Tab|Previous Cell|
|Enter|Next Row|
|Ctrl + Shift + D|Open table controls sidebar|

Or use the command palette and search "Advanced Tables". There are many commands available, don't forget to scroll!

##### 2. [Calender](https://github.com/liamcain/obsidian-calendar-plugin)
日记，周记，以及可插入模板，预览
>等日记多一点再看看

##### 3.[Day Planner](https://github.com/lynchjames/obsidian-day-planner/blob/main/README.md#usage)
按照时间顺序列出每日任务即可，可以加sub-todo，和列表说明

##### 4. Media Extended & bili plugin
- https://github.com/aidenlx/media-extended - media extend
- https://github.com/aidenlx/mx-bili-plugin - 上面插件的bilibili支持
从release中，下载除source code以外的js、json文件，分别作为两个插件进行安装
在阅读模式点击视频链接。

##### 5. [admonition](https://github.com/valentine195/obsidian-admonition)
各种块插入，一些设置的代码块，丰富文本样式
- Options
```ad-<type> # Admonition type. See below for a list of available types.
title:                  # Admonition title.
collapse:               # Create a collapsible admonition.
icon:                   # Override the icon.
color:                  # Override the color.
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod nulla.
```

Please note that as of **4.4.1**, the `title`, `collapse`, `icon` and `color` parameters must be at the _top_ of the block, in any order.
	- titles
		- Leave the title field blank to only display the admonition.
		- they support the full Obsidian Markdown syntax.
		- if don't write title, it will use tpye as title
	- `collapse: open` will start the admonition opened on render, but allow collapse on click.
	- color
		- The color entered must be an RGB triad.
	- icon
		- The icon name entered must be the exact icon name from FontAwesome or RPGAwesome.
	- nested
		- 缩进和感叹号可以产生多层
	- code blocks
		- in an nested admonition, use `~~~` to signal code blocks
- Admonition Types
The following admonition types are currently supported:
| Type | Aliases |
| ---- | ------- |
|note|note, seealso|
|abstract|abstract, summary, tldr|
|info|info, todo|
|tip|tip, hint, important|
|success|success, check, done|
|question|question, help, faq|
|warning|warning, caution, attention|
|failure|failure, fail, missing|
|danger|danger, error|
|bug|bug|
|example|example|
|quote|quote, cite|

See [this](https://squidfunk.github.io/mkdocs-material/reference/admonitions/) for a reference of what these admonitions look like.

The default admonitions are customizable by creating a user-defined admonition of the same name.
```ad-note
color: 200, 200, 200
collapse: open
icon: triforce
Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod nulla.

```
```ad-note
title: Nested Admonitions
collapse: open

Hello!

!!! ad-note
	title: This admonition is nested.
	This is a nested admonition!
	!!! ad-warning
		title: This admonition is closed.
		collapse: close


This is in the original admonition.
```
````ad-info

```ad-bug
title: I'm Nested!
~~~javascript
throw new Error("Oops, I'm a bug.");
~~~
```

```javascript
console.log("Hello!");
```

````

##### 6. Annotator 插件
参考： https://mp.weixin.qq.com/s/MEPva7Os_nOyl1vgTBZTwQ?utm_source=pocket_mylist - 二一的笔记
Obsidian 原生支持 PDF 阅读，你只需要将 PDF 文件**拖到文件夹上**即可载入，而安装 `Annotator 插件` 除了能让 Obsidian **额外支持阅读 EPUB 格式**的文件，还能进行高亮、批注等操作。

![](https://pocket-image-cache.com//filters:format(jpg):extract_focal()/https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_gif%2FoiabfUBO7nd798sY99oamUhGqdrE6KAlHcKBZ1Jr4E3Cmv3ibzde9nJvzbicQFIuq5weuCibRuQlX5M3Rfs68tw4Ug%2F640%3Fwx_fmt%3Dgif)

- 插件用法

1.  将你的 PDF 文件拖入 Obsidian 的文件夹上
2.  新建一个文档，然后在文档的最开头添加下面这样的 YAML 语法

```
---annotation-target: 路径/文件名.pdf---
```

例如，如果我要打开存放在`附件`文件夹中的 `基督山伯爵.pdf`，则正确的语法应该是   

```
---annotation-target: 附件/基督山伯爵.pdf---
```

然后 Ctrl+E 进入阅读视图，如果无法正确读取 PDF 文档，可以点击右上角的三个小点，然后再点击 `Annotate`

插件效果

1.  所有高亮和批注都会成为侧边栏中的卡片
2.  点击卡片即可跳转回 PDF 原文位置.
3.  你还可以为批注打上可被 Obsidian 识别的标签

![](https://pocket-image-cache.com//filters:format(jpg):extract_focal()/https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_gif%2FoiabfUBO7nd6h4TuzuRJgb4ulBiaBRGDpISyYEibRPVcskY6kExRpOClqBfgw7EdKkDWWfuyw2YUCYc6FICQiaXSBA%2F640%3Fwx_fmt%3Dgif)

点击右上角的 Open as MD 将关闭批注模式，然后你就能看到，所有的高亮和批注都被搜集起来组成一篇文档了。


##### 7. kanban + quickadd
quickadd setting：
[在obsidian中如何使用QuickAdd和Kanban插件来管理日程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1mb4y1y7R6/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7)
todo：[06:54](https://www.bilibili.com/video/BV1mb4y1y7R6/?spm_id_from=333.337.search-card.all.click&vd_source=f8bf73f9a2b495eaf6f8446fa6016bc7#t=414.848546)

#### 待学习使用集合
##### 1. search(obsidian system)
https://help.obsidian.md/Plugins/Search -> documents

##### 2.[Tasks](https://schemar.github.io/obsidian-tasks)
>Track tasks across your entire vault. 
  Query them and mark them as done wherever you want. 
	Supports due dates, recurring tasks (repetition), done dates, sub-set of checklist items, and filtering.
- Ctrl+T新建任务或者 -[ ]

[【效率办公】Obsidain插件之Tasks-任务管理利器 - 知乎](https://zhuanlan.zhihu.com/p/440969902)
##### - usage

##### - sorting

##### 3. [Template](https://silentvoid13.github.io/Templater/settings.html)

##### 4. [Dataview](https://github.com/blacksmithgu/obsidian-dataview)


## 图床的使用
1. Picgo配置和安装：[[图床设置]]
2. obsidian设置：
	- 先安装xclip，使能剪切板`sudo apt install xclip`
		- 可以配合截图软件：[[简单安装软件]]
	- 下载配置obsidian-image auto upload插件，并启用
		- https://gitee.com/whghcyx/obsidian-plugin/blob/master/plugin/obsidian-image-auto-upload-plugin.zip - gitee下载
		- 在设置中找到该插件，填写picGo Server地址：[http://127.0.0.1:36677/upload](http://127.0.0.1:36677/upload)
	- 记录：我使用的是picgo，上传到github中cdn/obsidian

### 语法教程
[Obsidian教程一 - 知乎](https://zhuanlan.zhihu.com/p/492198616#:~:text=%E4%B8%8D%E6%98%AF%E6%89%80%E6%9C%89%20MD%E7%BC%96%E8%BE%91%E5%99%A8%20%E9%83%BD%E6%94%AF%E6%8C%81%E7%9B%AE%E5%BD%95%E7%94%9F%E6%88%90%20Obsidian%20%E5%B0%B1%E4%B8%8D%E6%94%AF%E6%8C%81%EF%BC%8C%E4%B8%8D%E8%BF%87,OB%20%E6%98%AF%E8%87%AA%E5%B8%A6%E5%A4%A7%E7%BA%B2%E7%9A%84%EF%BC%8C%E5%B0%B1%E6%98%AF%E7%9B%AE%E5%BD%95%E7%9A%84%E6%95%88%E6%9E%9C%20%E8%BE%93%E5%85%A5%E4%B8%8B%E6%96%B9%E5%86%85%E5%AE%B9%E4%BC%9A%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AA%E7%9B%AE%E5%BD%95%EF%BC%9A%20%5Btoc%5D%202.)包括甘特图，图片等等语法

[Markdown格式文档图片设置居中_汉木木的博客-CSDN博客_markdown图片居中](https://blog.csdn.net/Jivove/article/details/123496699)令人头痛的居中问题

