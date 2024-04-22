# 简介

GitBook 是一款现代化的文档平台，支持团队协作，可以在上面写产品文档、内部知识分享、接口文档等。可以通过 Gitbook 可以自动转换生成 HTML 静态页面或 PDF 文件，方便快捷的管理文件。GitBook 有网页版和本地版两种，网页版通过 https://www.gitbook.com/ 网址进行访问，本地版主要基于 Node.js 环境进行开发

这是 [gitbook](https://github.com/GitbookIO/gitbook) 项目主页上对 gitbook 的定义，gitbook 首先是一个软件，正如上面定义的那样，它使用 Git 和 Markdown 来编排书本，如果用户没有听过 Git 和 Markdown，那么 gitbook 可能不适合你！

## 环境准备

### 配置 Node.js 环境

使用 Gitbook 需要配置 Node.js 环境，安装步骤可查看 [官方文档](https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md) ，Node.js 官方下载地址 https://nodejs.org/en/download 。需要注意的是 Gitbook 对 Node.js 的版本有要求，建议安装老版本的 Node.js ，例如 v10.24.1 ，这样可以避免后期的很多问题，或者通过 Windows 系统下的 Node.js 版本管理器 nvm-windows（Node.js Version Manager for Windows）实现在同一台设备上进行多个 Node.js

 版本之间的切换。

```powershell
# 查看node版本
node -v
# 查看npm版本
npm -v
```

### 安装 Gitbook

安装 Gitbook 的最佳方式是通过NPM。在终端提示下，只需运行一下命令即可安装 Gitbook

```powershell
# 安装gitbook包
npm install gitbook-cli -g
# 查看全局已安装的包
npm list -g
# 查看gitbook的版本，会自动做版本升级
gitbook.cmd -V
```

### 初始化项目

创建一个目录，并在该目录下初始化 gitbook 项目

```powershell
# 创建新demo目录作为测试
cd E:\gitbook\demo\
mkdir demo
cd demo
# 初始化demo目录
gitbook.cmd init
```

初始化目录后，在demo目录下会产生2个文件，README.md 文件展示出来的效果类似于文档首页、SUMMARY.md 文件是作为目录使用。主要需要调整的文件是 SUMMARY.md ，README.md 文件也是在 SUMMARY.md 文件中指定的，因此 README.md 文件名称也可修改，只不过首页使用此文件命名是一个常见的习惯

#### NPM初始化项目

实际上本地的 Gitbook 不需要 npm 初始化也能够提供访问，只不过考虑到后期可能会安装一些插件，如果将 Gitbook 项目转换为 NPM 项目更方便管理插件版本

```powershell
# NPM初始化
npm init
```

NPM 初始化过程中会产生一些交互选项，全部默认回车确认即可，执行成功后会在目录下产生一个 package.json 文件

#### 启动 Gitbook 服务

```powershell
# 直接使用gitbook工具启动服务
gitbook.cmd serve

# 使用npm启动Gitbook服务
vim E:\gitbook\demo\package.json
  "scripts": {
    //"test": "echo \"Error: no test specified\" && exit 1"    # 注释掉
    "serve": "gitbook.cmd serve",    # 新增两条命令
    "build": "gitbook.cmd build"     # Gitbook的打包命令
# npm运行serve脚本
npm run serve
```

启动 Gitbook 服务后会在目录下产生一个 _book 目录，Gitbook 会在这个目录下自动生成静态页面文件。使用 NPM 启动项目之前需要先修改 \_book 目录下的 `package.json` 文件内容，添加的两个脚本分别是 gitbook 在本地启动的命令，和 gitbook 打包成 HTML 静态文件的命令。使用 npm 命令本地演示时其实就是执行了 `package.json` 文件的 `scripts` 中的 `serve` 脚本，即 `gitbook.cmd serve`

#### 章节配置

Gitbook 使用文件 SUMMARY.md 来定义书本的章节和子章节的结构，简单一点理解就是书本的目录。SUMMARY.md 文件的格式是一个简单的链接列表，链接的名字是章节的名字，链接的指向是章节文件的路径，子章节被简单的定义为一个内嵌与父章节的列表

```
# 概要

- [首页](README.md)
- [章节一](chapter1.md)
- [章节二](chapter2.md)
- [章节三](chapter3.md)
  - [1.1 第一节](directory1/chapter3_1.md)
  - [1.2 第二节](directory2/chapter3_2.md)
```

### 忽略文件

任何在目录下的文件，在最后生成电子书时都会被拷贝到输出目录中，如果想要忽略某些文件，和 Git 一样，Gitbook 会依次读取 `.gitignore` 和 `.ignore` 文件来将一些文件和目录排除

### 配置文件

Gitbook 在编译书籍的时候会读取书籍源码顶层目录中的`book.js`或`book.json`，参考 [gitbook 文档](https://github.com/GitbookIO/gitbook) 可以知道

`book.json`示例：

```json
{
    // 书籍信息
    "title": "书名",
    "description": "描述",
    "isbn": "图书编号",
      "author": "作者",
      "lang": "zh-cn",
    // 插件列表
    "plugins": [],
    // 插件全局配置
    "pluginsConfig": {
        "fontSettings": {
            "theme": "sepia","night" or "white",
            "family": "serif" or "sans",
            "size": 1 to 4
        }
    },
    // 模板变量
    "variables": {
        // 自定义
    }
}
```

`book.js`示例：

```javascript
module.exports = {
    // 书籍信息
    "title": "书名",
    "description": "描述",
    "isbn": "图书编号",
      "author": "作者",
      "lang": "zh-cn",
    // 插件列表
    "plugins": [],
    // 插件全局配置
    "pluginsConfig": {},
    // 模板变量
    "variables": {
        // 自定义
    },
};
```

### 卸载 Gitbook

```powershell
npm uninstall -g gitbook-cli
npm uninstall -g gitbook
npm cache clean -f
```

## 插件管理

Gitbook 最灵活的地方就是有很多插件可以使用，也可以自己写插件。所有插件的命名都是以`gitbook-plugin-xxx`的形式，在使用插件前，需要在书籍源码顶层目录中，也就是 demo 目录中创建一个`book.js`文件，这是 Gitbook 的配置文件，文件内容可自定义，内容格式如下

```json
// book.js
module.exports = {
    title: 'Gitbook电子书',
    author: 'hebor',
    lang: 'zh-cn',
    description: 'Gitbook电子书示例',
    //plugins:[],    插件列表
    //pluginsConfig: {},    插件全局配置
    //variables: {}    模板变量
};
```

### 搜索插件

1、安装插件

```powershell
# 安装搜索插件
npm i gitbook-plugin-search-pro
# 检查demo目录下的package.json文件内容
cat .\package.json
"dependencies": {
    "gitbook-plugin-search-pro": "^2.0.2"    # 说明已经安装成功
  }
```

2、安装成功后，在`book.js`中添加插件的配置

```javascript
"plugins": [
    "-lunr", "-search", "search-pro"
],
```

通过npm安装插件后会在demo目录下自动生成一个`node_modules`目录，该目录下保存基于模块命名的子目录文件，在模块子目录下会有README.md文件，此README.md说明了模块的用法

### 代码框插件

1、安装插件

```powershell
npm install gitbook-plugin-code
```

2、安装成功后，在`book.js`中添加插件的配置

```javascript
// 插件列表
"plugins": [
    "-lunr", "-search", "search-pro" , "code"
],
// 插件全局配置
"pluginsConfig": {
    "code": {    // gitbook-plugin-code插件提供的功能配置，不允许复制代码，这属于可选项
        "copyButtons": false
}
```

### 自定义主题插件

1、安装插件

```powershell
npm install gitbook-plugin-theme-主题名
```

2、安装成功后，在`book.js`中添加插件的配置

```javascript
{
    plugins: ["theme-主题名"]
}
```

### 菜单折叠插件

1、插件安装

```powershell
npm install gitbook-plugin-expandable-chapters
```

2、安装成功后，在`book.js`中添加插件的配置

```javascript
"plugins": [
    "-lunr", "-search", "search-pro" , "code" , "expandable-chapters"
],
```

### 返回顶部插件

1、插件安装

```powershell
npm install gitbook-plugin-back-to-top-button
```

2、安装成功后，在`book.js`中添加插件的配置

```javascript
"plugins": [
    "-lunr", "-search", "search-pro" , "code" , "expandable-chapters" , "back-to-top-button"
],
```

> Gitbook 更多插件可以通过 https://plugins.gitbook.com/ 或 https://www.npmjs.com/ 搜索`gitbook-plugin`来获取

## 构建项目

使用`gitbook build`构建项目，成功后即可在 _book 目录中生成对应的静态资源

### Pages服务

很多 Git 服务提供商都有 Pages 服务，比如 Github Pages。由于 Github 的不稳定性，在国内访问经常出现问题，因此可以考虑使用 Gitee Pages服务

除了使用 Git 服务提供商提供的 Pages 服务，也可以考虑自己购买云服务器，自己提供服务

### 构建静态页面

```powershell
E:\gitbook\network> npm run build
```

通过`gitbook build`构建后，在 _book 目录下除了会生成 index.html 静态文件以外，可以看到 build 会将 package.json 文件也一起打包，实际上静态页面是不需要这个文件的，因此可以利用到 gitbook 的忽略文件 .bookignore 在 build 打包时忽略指定的文件

```powershell
vim E:\gitbook\network\.bookignore
package.json
.bookignore
E:\gitbook\network> npm run build
```

> **关于上传文件乱码问题**

文件是在 WIndows 下创建的，Windows 的文件名中文编码默认为 GBK ，而Linux中默认文件名编码为 UTF8 ，由于编码不一致所以导致了文件名乱码的问题，解决这个问题需要对文件名进行转码。文件名转码工具 convmv 

```shell
yum install -y convmv
convmv -f GBK -t UTF-8 --notest -r /home/hebor/
    -f enc：源编码
    -t enc：新编码
    -r：递归处理子文件夹
    --list：显示所有可用的编码
    --notest：直接转换不测试
```

