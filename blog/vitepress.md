## 参考文档
### 官方文档入口
[VitePress | 由 Vite 和 Vue 驱动的静态站点生成器](https://vitejs.cn/vitepress/)
### 一些参考博客
比较喜欢的布局，也是主要参考，开源代码里面有很多没有，可以参考他的布局再完善
[Daily Notes 日常笔记 | 茂茂物语](https://maomao.fe-mm.com/daily-notes/)
[VitePress 生成站点地图 | 茂茂物语](https://maomao.fe-mm.com/daily-notes/issue-39#vitepress-%E7%94%9F%E6%88%90%E7%AB%99%E7%82%B9%E5%9C%B0%E5%9B%BE)
[使用 VitePress 打造个人前端导航网站 | 茂茂物语](https://maomao.fe-mm.com/daily-notes/issue-38)
开源的比较多，也是上一篇的主要参考，但是上面那位的代码更清晰
[查尔斯的知识库 | 个人技术知识库，记录和分享个人碎片化、结构化、体系化的技术知识内容](https://blog.charles7c.top/)
还没深入
[🔧 一篇教你用VitePress + Github Pages搭建博客 | 是柠新呀的知识库](https://xuxing409.github.io/my-blog/technology/article/building-blog-with-vitepress.html)

## 开始
### 从零开始
[入门 | VitePress 中文网](https://vitepress.qzxdp.cn/guide/getting-started.html)
```
npm install -D vitepress
```

```
npx vitepress init
```
记得安装和升级 npm、node 

得到几个问题：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202410271709576.png)

### 从模板开始
下载好的模板文件夹里
解决一些依赖问题
```
npm install
```
本地运行调试，一般下好的模板是可以直接跑的
```
npm run dev
```
本地编译
```
npm run build
```

编译好的文件在：.vitepress/dist 下面，把这个上传 github page 或者服务器

### 服务器连接
[[阿里云服务器+域名配置]]
阿里云下载宝塔
把编译好的文件上传服务器，可以用 ftp(filezila)或者 ssh(scp)
```
scp -r source destination
```
直接新建站点，把站点文件位置选在刚刚上传的那个文件夹，index.html 的父文件夹

## 其他待实现参考
[推送vitepress到阿里云将vitepress推送到阿里云实例中,使用简单的JS语言即可完成.导入ssh2依赖,依靠 - 掘金](https://juejin.cn/post/7351690896918167615)
👍git action 自动上传服务器
[使用Vitepress搭建并发布个人网站-CSDN博客](https://blog.csdn.net/AKALI822/article/details/134180744)
[配置多个Git账号（windows 10）\_win10系统git配置多账号-CSDN博客](https://blog.csdn.net/q13554515812/article/details/83506172)
😍倒计时网站（基于vue）
[GitHub - abc55667788/ccf-deadlines: ⏰ CCF recommendation conference Deadline Countdowns / Please star this project, thanks\~](https://github.com/abc55667788/ccf-deadlines)
