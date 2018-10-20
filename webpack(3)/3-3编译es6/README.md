# 编译es6
js作为宿主语言，很依赖执行的环境（浏览器、node等），不同环境对js语法的支持不尽相同，特别是ES6之后，ECMAScrip对版本的更新已经到了一年一次的节奏，虽然每年更新的幅度不大，但是每年的提案可不少。babel的出现就是为了解决这个问题，把那些使用新标准编写的代码转译为当前环境可运行的代码，简单点说就是把ES6代码转译（转码+编译）到ES5。

经常有人在使用babel的时候并没有弄懂babel是干嘛的，只知道要写ES6就要在webpack中引入一个babel-loader，然后胡乱在网上copy一个.babelrc到项目目录就开始了。理解babel的配置很重要，可以避免一些不必要的坑，比如：代码中使用Object.assign在一些低版本浏览器会报错，以为是webpack打包时出现了什么问题，其实是babel的配置问题。

ES6，ES即ECMAScript，6表示第六个版本(也被称为是ES2015，因为是2015年发布的)，它是javascript的实现标准。

被纳入到ES标准的语法必须要经过如下五个阶段:

Stage 0: strawman
Stage 1: proposal
Stage 2: draft   -   必须包含2个实验性的具体实现，其中一个可以是用转译器实现的，例如Babel。
Stage 3: candidate  -  至少要有2个符合规范的具体实现。
Stage 4: finished

可以看到提案在进入stage3阶段时就已经在一些环境被实现，在stage2阶段有babel的实现。所以被纳入到ES标准的语法其实在大部分环境都已经是有了实现的，那么为什么还要用babel来进行转译，因为不能确保每个运行代码的环境都是最新版本并已经实现了规范。

[参考文章](https://juejin.im/post/59ec657ef265da431b6c5b03)

### 1、Babel的版本变更
在babel5的时代，babel属于全家桶型，只要安装babel就会安装babel相关的所有工具，即装即用。

但是到了babel6，具体有以下几点变更：

- 移除babel全家桶安装，拆分为单独模块，例如：babel-core、babel-cli、babel-node、babel-polyfill等；
可以在babel的github仓库看到babel现在有哪些模块。

- 新增 .babelrc 配置文件，基本上所有的babel转译都会来读取这个配置；

- 新增 plugin 配置，所有的东西都插件化，什么代码要转译都能在插件中自由配置；

- 新增 preset 配置，babel5会默认转译ES6和jsx语法，babel6转译的语法都要在perset中配置，preset简单说就是一系列plugin包的使用。


### 2、.babelrc配置文件其实就是一个json文件

.babelrc是babel的全局配置文件，所有的babel操作（包括babel-core、babel-node）基本都会来读取这个配置。

后面的后缀rc来自linux中，使用过linux就知道linux中很多rc结尾的文件，比如.bashrc，rc是run command的缩写，翻译成中文就是运行时的命令，表示程序执行时就会来调用这个文件。

在配置.babelrc文件时，需要把配置项写成json规范的格式，要使用双引号。

### 3、babel-core

babel-core是作为babel的核心存在，babel的核心api都在这个模块里面

#### 4、babel-polyfill

polyfill这个单词翻译成中文是垫片的意思，详细点解释就是桌子的桌脚有一边矮一点，拿一个东西把桌子垫平。polyfill在代码中的作用主要是用已经存在的语法和api实现一些浏览器还没有实现的api，对浏览器的一些缺陷做一些修补。例如Array新增了includes方法，想使用，但是低版本的浏览器上没有，就得做兼容处理。

因为babel的转译只是语法层次的转译，例如箭头函数、解构赋值、class，对一些新增api以及全局函数（例如：Promise）无法进行转译，这个时候就需要在代码中引入babel-polyfill，让代码完美支持ES6+环境。

```
引入方法：
//在js文件，代码的最顶部进行require或者import

require("babel-polyfill");

import "babel-polyfill";

//如果使用webpack，也可以在文件入口数组引入，或者在对应的js文件中引入

module.exports = {
  entry: ["babel-polyfill", "./app/js"]
};
```

但很多时候我们并不会使用所有ES6+语法，全局添加所有垫片肯定会让我们的代码量上升。

### 5、plugins

plugins是babel中的插件，通过配置不同的插件才能告诉babel，我们的代码中有哪些是需要转译的。

```
{
    "plugins": [
        "transform-es2015-arrow-functions", //转译箭头函数
        "transform-es2015-classes", //转译class语法
        "transform-es2015-spread", //转译数组解构
        "transform-es2015-for-of" //转译for-of
    ]
}
//如果要为某个插件添加配置项，按如下写法：
{
    "plugins":[
        //改为数组，第二个元素为配置项
        ["transform-es2015-arrow-functions", { "spec": true }]
    ]
}
```

上面这些都只是语法层次的转译，前面说过有些api层次的东西需要引入polyfill，同样babel也有一系列插件来支持这些。

### 6、transform-runtime

transform-runtime这个插件依赖于babel-runtime，所以安装transform-runtime的同时最好也安装babel-runtime，为了防止一些不必要的错误。

比较transform-runtime与babel-polyfill引入垫片的差异：

- 使用runtime是按需引入，需要用到哪些polyfill，runtime就自动帮你引入哪些，不需要再手动一个个的去配置plugins，只是引入的polyfill不是全局性的，有些局限性。而且runtime引入的polyfill不会改写一些实例方法，比如Object和Array原型链上的方法，像前面提到的Array.protype.includes。

- babel-polyfill就能解决runtime的那些问题，它的垫片是全局的，而且全能，基本上ES6中要用到的polyfill在babel-polyfill中都有，它提供了一个完整的ES6+的环境。babel官方建议只要不在意babel-polyfill的体积，最好进行全局引入，因为这是最稳妥的方式。

- 一般的建议是开发一些框架或者库的时候使用不会污染全局作用域的babel-runtime，而开发web应用的时候可以全局引入babel-polyfill避免一些不必要的错误，而且大型web应用中全局引入babel-polyfill可能还会减少你打包后的文件体积（相比起各个模块引入重复的polyfill来说）。

### 7、presets

一个一个配置插件会非常的麻烦，为了方便，babel为我们提供了一个配置项叫做persets（预设）。

预设就是一系列插件的集合，就好像修图一样，把上次修图的一些参数保存为一个预设，下次就能直接使用。

如果要转译ES6语法，只要按如下方式配置即可：

```
//先安装ES6相关preset： cnpm install -D babel-preset-es2015
{
    "presets": ["es2015"]
}

//如果要转译的语法不止ES6，还有各个提案阶段的语法也想体验，可以按如下方式。
//安装需要的preset： cnpm install -D babel-preset-stage-0 babel-preset-stage-1 babel-preset-stage-2 babel-preset-stage-3
{
    "presets": [
        "es2015",
        "stage-0",
        "stage-1",
        "stage-2",
        "stage-3",
    ]
}

//同样babel也能直接转译jsx语法，通过引入react的预设
//cnpm install -D babel-preset-react
{
    "presets": [
        "es2015",
        "react"
    ]
}
```

不过上面这些preset官方现在都已经不推荐了，官方唯一推荐preset：babel-preset-env。

这款preset能灵活决定加载哪些插件和polyfill，不过还是得开发者手动进行一些配置。

```
// cnpm install -D babel-preset -env
{
    "presets": [
        ["env", {
            "targets": { //指定要转译到哪个环境
                //浏览器环境
                "browsers": ["last 2 versions", "safari >= 7"],
                //node环境
                "node": "6.10", //"current"  使用当前版本的node
                
            },
             //是否将ES6的模块化语法转译成其他类型
             //参数："amd" | "umd" | "systemjs" | "commonjs" | false，默认为'commonjs'
            "modules": 'commonjs',
            //是否进行debug操作，会在控制台打印出所有插件中的log，已经插件的版本
            "debug": false,
            //强制开启某些模块，默认为[]
            "include": ["transform-es2015-arrow-functions"],
            //禁用某些模块，默认为[]
            "exclude": ["transform-es2015-for-of"],
            //是否自动引入polyfill，开启此选项必须保证已经安装了babel-polyfill
            //参数：Boolean，默认为false.
            "useBuiltIns": false
        }]
    ]
}
```