# Notes after SuperAgent source code

Take notes when read SuperAgent source code commit-by-commit.

From https://github.com/visionmedia/superagent

## Content
* [Initial commit](#f808-Initial-commit)
* [v0.1.1 ](#v0.1.1-Release-0.1.1)

## f808 Initial commit

### Makefile

1. `@node` 以@开头是bash命令的特征，表示命令不会被回显
2. `.PHONY: test clean` `.PHONEY`指明test和clean是伪目标，即便这两个文件存在，也是伪目标
3. 大括号表达式 `rm -fr SuperAgent{,.min}.js`删除SuperAgent.js和SuperAgent.min.js
4. Makefile默认使用sh，sh不支持大括号展开语法，为了设置Makefile的shell，需要在开头加上
   SHELL=/bin/bash或SHELL=bash
5. npm i uglify-js 压缩js文件

### test.js

1. `o = $` 看似普通的一句话，可以体现作者的思想和良好习惯，使得对外部的依赖最小，将来一旦jquery
    导出的不是$，只需修改一句话，而不是多处
2. `window.onerror` 可以处理未被catch的异常，对于需要简单处理回调函数抛出异常时，有奇效
3. `Function.prototype.length` 返回函数定义时的参数个数

### superagent.js

#### superagent的基本结构

导出request是一个构造函数，同时又有一些get, post， del，put等属性作为常用的函数，构造函数
返回Request对象，并且每一个属性函数也可以创建Request对象并返回，Request对象的成员函数都返
回`this`以支持链式调用

这种结构是返回一个构造函数，并且为这个构造函数添加一些shortcut的属性函数来提供方便，其实所有的功能已经通过构造函数导出来了

思考：
如果是我做的，那么首先想到的一定是导出一个类，也就是把Request导出，但是事实上，对类进行封装
要好得多：
1. 简化外部使用，原来需要new Request，现在只需request()
2. 更重要的是，可以在request函数函数上导出一些shortcut函数

## v0.1.1 Release 0.1.1

### node.js
* `querystring`
`querystring`是`node`的一个操作url中的`querystring`的库，js标准只提供全局函数`decodeURIComponent`、`encodeURIComponent`等，只是对url中特殊字符做转义而没有对url encoded的`querystring`解析和编码，但是node提供了`querystring`，类似`JSON`提供转换
