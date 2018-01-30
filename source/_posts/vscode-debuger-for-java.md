---
title: 配置vscode调试java代码
date: 2018-01-25 14:34:01
tags: [vscode]
categories:
- 闲聊杂谈
- 开发工具
---

习惯了vscode，现阶段因工作需要需切换到java做项目，因此就想看看能不能配置一下vscode来写java，配置完后，发现效果还不错，也有一些小发现，这里将过程记录一下，以备后续用。

<!-- more -->

## 安装vscode插件
- [Language support for Java](https://marketplace.visualstudio.com/items?itemName=redhat.java)
- [Debugger for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug)

安装两个插件，第一个是redhat官方出的java的支持，第二个是微软出的java调试工具。

## 配置单文件debug
比如我们很简单的目录结构：
```
├── bin
└── src
```
src目录存放java源代码，bin目录存放编译后的class文件。
官方提供的Language support for java是通过Eclipse ™ JDT Language Server,Buildship来启动一个服务监听并编译源代码，这里我们需要手动创建两个文件来配置该编译服务的相关项。
### 创建.project和.classpath文件
- .project
```xml
<?xml version="1.0" encoding="UTF-8"?>
<projectDescription>
	<name>testjava</name>
	<comment>Project testjava created by Buildship.</comment>
	<projects>
	</projects>
	<buildSpec>
		<buildCommand>
			<name>org.eclipse.jdt.core.javabuilder</name>
			<arguments>
			</arguments>
		</buildCommand>
		<buildCommand>
			<name>org.eclipse.buildship.core.gradleprojectbuilder</name>
			<arguments>
			</arguments>
		</buildCommand>
	</buildSpec>
	<natures>
		<nature>org.eclipse.jdt.core.javanature</nature>
		<nature>org.eclipse.buildship.core.gradleprojectnature</nature>
	</natures>
</projectDescription>
```

- .classpath
```xml
<?xml version="1.0" encoding="UTF-8"?>
<classpath>
	<classpathentry kind="src" path="src"/>
	<classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8/"/>
	<classpathentry kind="con" path="org.eclipse.buildship.core.gradleclasspathcontainer"/>
	<classpathentry kind="output" path="bin"/>
</classpath>
```

第一个.project文件，vscode会识别该项目为eclipse项目，第二个文件.classpath配置了源码目录以及编译输出目录等。

### 配置launch.json
- 如果项目里没有.vscode，先创建一个.vscode的目录。
- 在.vscode目录里创建launch.json文件：
 ```json
 {
   "version": "0.2.0",
   "configurations": [
     {
       "type": "java",
       "name": "Debug (Launch)",
       "request": "launch",
       "cwd": "${workspaceFolder}/bin",
       "sourcePaths": [
         "$(workspaceRoot)/src"
       ],
       "classPaths": [
         "",
         "$(workspaceRoot)/bin",
				 "/Users/coolcao/.gradle/caches/modules-2/files-2.1/com.fasterxml.jackson.core/jackson-databind/2.4.1/f07c773f7b3a03c3801d405cadbdc93f7548e321/jackson-databind-2.4.1.jar",
       ],
       "mainClass": "com.coolcao.test.${fileBasenameNoExtension}",
       "args": ""
     }
   ]
 }
 ```
 简单说明一下，cwd参数配置javac命令运行的根目录，这里应指定编译后的.class文件所在的根目录。
 sourcePaths目录配置源码目录，包括自己写的源码和第三方库的源码目录。
 classPaths配置class目录，自己的源码编译后的目录，以及依赖的第三方的jar包的目录，如果依赖了第三方包，则这里要配依赖的jar包的所在目录，否则在debug时会出现找不到类的错误。
 mainClass配置要运行的主类文件。这里有一点不好的是，需要手动补全类的包名。如果文件都不在一个同一个包下，那么每次debug时需要指定当前运行类所在的包。
 以上几个参数是最重要的，用了vscode内置的几个变量。
 args是运行class文件需要添加的参数，根据需要配置即可。

最后文件目录结构差不多如下：
```
├── .classpath
├── .project
├── .vscode
│   └── launch.json
├── bin
│   └── com
│       └── coolcao
│           └── test
│               └── App.class
└── src
    └── com
        └── coolcao
            └── test
                └── App.java
```
这样，每次修改java代码时，vscode会自动编译代码到指定的class目录，需要debug，点击debug按钮，运行Debug (Launch)即可。

## 配置gradle管理的spring boot项目
使用gradle创建的spring boot项目，配置起来和上面差不多，思路都是一样的。
1. 先配置.project和.classpath文件，注意源码目录和编译后字节码文件.class目录。
2. 配置launch.json，注意cwd参数指定javac运行目录，以及sourcePaths和classPaths对应着上面第一步的配置。
3. 指定运行的主类即可。

## 题外话：使用idea创建gradle项目，在java9下提示gradle版本不兼容java9的解救方法
使用idea创建gradle项目时，如果提示gradle不兼容java9:
```
Could not determine java version from '9.0.1'.
```
有两种解救办法：
* 创建项目时，自己指定gradle为本地已下载的高版本gradle
 该方法前提是本地已下载了支持java9的gradle版本，比如 4.4.1，下载完后一般放到`~/.gradle/wrapper/dists`目录下，然后创建的时候选择该版本即可
  ![pic](http://7xt3oh.com2.z0.glb.clouddn.com/blog/idea_gradle.png)
* 使用 gradlew 配置
创建完成后，如果提示gradle版本不对或者要修改gradle版本，可以使用gradle wrapper：
```
> gradle wrapper
然后在项目里的gradle目录里修改文件gradle-wrapper.properties，修改distributionUrl为：
https\://services.gradle.org/distributions/gradle-4.4.1-bin.zip
```
