ubuntu  javajs 和环境配置
已经安好了 node、npm 就不整 nvm 了
[Ubuntu升级nodejs版本\_ubuntu nodejs升级\_zzqwtc的博客-CSDN博客](https://blog.csdn.net/zzq0523/article/details/122910368#:~:text=Ubuntu%E5%8D%87%E7%BA%A7nodejs%E7%89%88%E6%9C%AC%201%20%E9%A6%96%E5%85%88%E4%B8%8B%E8%BD%BD%20n%20%E8%BF%99%E4%B8%AA%E7%94%A8%E4%BA%8E%E6%9B%B4%E6%96%B0%20node%20%E7%89%88%E6%9C%AC%E7%9A%84%E5%B7%A5%E5%85%B7%20sudo,%E5%8D%B3%E5%8F%AF%EF%BC%8C%E6%88%91%E7%94%A8%E7%9A%84%E6%98%AF%20zsh%20%EF%BC%8C%E6%89%80%E4%BB%A5%E6%89%A7%E8%A1%8C%20hash%20-r%20%EF%BC%88%E6%A0%B9%E6%8D%AE%E4%B8%8A%E5%9B%BE%E5%80%92%E6%95%B0%E7%AC%AC%E4%B8%89%E8%A1%8C%E6%8F%90%E7%A4%BA%EF%BC%89%203%20%E6%9B%B4%E6%96%B0%E5%AE%8C%E6%88%90)

安装 chrome、firefox、edge 扩展，以及默认的一堆 javajs 的扩展
[解决：在 VSCode 中如何设置默认的浏览器为Chrome或Firefox\_vscode设置默认浏览器\_狮子座的男孩的博客-CSDN博客](https://blog.csdn.net/weixin_43405300/article/details/124228615#:~:text=A%E3%80%81%E9%A6%96%E5%85%88%E9%9C%80%E8%A6%81%E5%86%8D%20VSCode%20%E4%B8%AD%E7%9A%84%E6%89%A9%E5%B1%95%E4%B8%AD%E5%AE%89%E8%A3%85%EF%BC%9A%20open%20in%20browser%20%E6%8F%92%E4%BB%B6%20%28%E8%8B%A5%EF%BC%9A%E5%B7%B2%E5%AE%89%E8%A3%85%E7%9A%84%E8%AF%B7%E5%BF%BD%E7%95%A5%E6%AD%A4%E6%AD%A5%E9%AA%A4%29%EF%BC%9B,%7B%22open-in-browser.default%22%3A%22Chrome%22%7D%20%EF%BC%8C%E6%AD%A4%E6%97%B6%E5%B0%B1%E5%B7%B2%E7%BB%8F%20%E5%B0%86%20VSCode%20%E7%9A%84%E9%BB%98%E8%AE%A4%E6%B5%8F%E8%A7%88%E5%99%A8%20%E4%BF%AE%E6%94%B9%E6%88%90%20Chrome%20%E4%BA%86%EF%BC%9B)

firefox、edge 的 save data 和 cookie 的都关了，建议还是不要用 edge，dev 调试贼不好使
[更改html代码后网页不更新\_weixin\_30684743的博客-CSDN博客](https://blog.csdn.net/weixin_30684743/article/details/95290812)
[Html静态页面更新，解决浏览器缓存不更新问题\_html更新页面\_NicestZK的博客-CSDN博客](https://blog.csdn.net/m0_37750806/article/details/119269635
[【web】关于修改html代码后网页不更新问题(自用)\_为什么改html页面没有更新\_代码搬运工小菜狗的博客-CSDN博客](https://blog.csdn.net/qq_51332755/article/details/124210366)

#### run

环境安装
```
npm install
```
本地部署   没有这个就 `npm install -g http_server`
```
http_server
```

vscode   start 的对应 package.json 里的命令设置
```
npm run start
```

lanch.json store
```
{

// Use IntelliSense to learn about possible attributes.

// Hover to view descriptions of existing attributes.

// For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387

"version": "0.2.0",

"configurations": [

{

"type": "firefox",

"request": "launch",

"reAttach": true,

"url": "http://localhost:8080",

"name": "Launch index.html",

"file": "${workspaceFolder}/index.html"

},

{

"name": "Launch Edge",

"request": "launch",

"type": "msedge",

"url": "http://localhost:8080",

"webRoot": "${workspaceFolder}/index.hmtl"

}

]

}
```

#### example
[[图形渲染程序log#sheen]]
package.json 从程序入口
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011443324.png)

index.html  调用scripts
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202309011444308.png)
