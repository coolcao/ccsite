---
title: 增强vscode中js代码提示
date: 2017-07-06 14:32:03
tags: [vscode,编辑器,开发工具]
categories:
- 技术博客
- 原创
---

## 使用 types 增强vscode中javascript代码提示功能

> 微软的vscode编辑器是开发typescript项目的不二首选，其本身也是采用typescript开发的。
> 使用过ts的同学都知道 `*.d.ts` 类型声明文件，其管理工具，从最初的 `tsd`,到后来的 `typings`,一直到现在的`@types`,类型声明文件为ts的智能提示，类型检查提供了有力支持。
我们也可以使用类型声明文件，增强vscode编辑javascript时的智能提示。
关于vscode这方面更深的说明，请访问以下链接：
* [https://code.visualstudio.com/docs/languages/javascript](https://code.visualstudio.com/docs/languages/javascript)
* [https://github.com/Microsoft/TypeScript/wiki/JavaScript-Language-Service-in-Visual-Studio](https://github.com/Microsoft/TypeScript/wiki/JavaScript-Language-Service-in-Visual-Studio)
* [https://code.visualstudio.com/docs/editor/intellisense](https://code.visualstudio.com/docs/editor/intellisense)

<!-- more -->

### 安装 types 文件
现在，我们可以不依赖typings直接使用npm安装所需要的types类型文件。
比如，我们要安装sequelize的类型文件，可以直接使用：
```
npm install @types/sequelize --save-dev
```
安装完成后，我们在 node_modules目录下发现有一个@types目录，该目录里就是所安装的所有的类型声明文件。
如果有的第三方npm包官方未提供类型声明文件时，可能会安装出错，找不到相应的包。这时，就没法利用其增强js代码的提示功能。
如果你熟悉使用ts如何编写`*.d.ts`文件，也可以自己写一个。

### 配置 jsconfig.json 文件
对于jsconfig.json文件的详细说明，请参照[这里](https://code.visualstudio.com/docs/languages/javascript#_javascript-project-jsconfigjson)。
在jsconfig.json文件中添加：
```json
"include": [
    "model/**",
    "service/**"
],
"typeAcquisition": {
    "include": [
        "sequelize"
    ]
}
```
其中typeAcquisition参数是必配的，标识启用类型感知功能，里面的include标识对哪个包启用。
上面的include不是必须的，只是用来标识jsconfig.json文件对哪些文件起作用。
开启后，如图：
![vscode](/pic/vscode1.png)
我们上图中例子提示的就是sequelize包中Model类的实例方法和属性。
vscode对智能感知的图标，也给了一定的汇总：
![vscode感知图标](/pic/vscode2.png)

## 在js文件中启用语义检查
如果要在js中启用类型检查，可以在文件最上面添加 `// @ts-check` 注释。
```javascript
// @ts-check
let easy = 'abc'
easy = 123 // Error: Type '123' is not assignable to type 'string'
```
或者在 `jsconfig.json`中进行配置：
```json
{
    "compilerOptions": {
        "checkJs": true
    },
    "exclude": [
        "node_modules"
    ]
}
```
详情请参阅[文档](https://code.visualstudio.com/docs/languages/javascript#_type-checking-and-quick-fixes-for-javascript-files)
