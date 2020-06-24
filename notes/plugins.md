# 插件的运行环境

插件没有像 loader 那样的独立运行环境



只能在 webpack 里面运行



## 插件的基本结构

基本结构：

```javascript
//插件名称
class MyPlugin {
    //插件上的 apply 方法
    apply(compiler) {
        //插件的 hooks
        compiler.hooks.done.tap("My Plugin", (stats) => {
            //插件处理逻辑
            console.log("hello world");
        })
    }
}

module.exports = MyPlugin;
```

插件使用：

```plugins: [new MyPlugin()]```



## 搭建插件的运行环境

```javascript
const path = requir('path');
const DemoPlugin = require('./plugins/demo-plugin.js');

const PATHS = {
    lib: path.join(__dirname, "app", "shake.js"),
    build: path.join(__dirname, "build")
};

module.exports = {
    entry: {
        lib: PATHS.lib
    },
    output: {
        path: PATHS.build,
        filename: "[name].js",
    },
    plugins: [new DemoPlugin()]
}
```



## 插件中如何获取传递的参数

通过插件的构造函数进行获取

```javascript
module.exports = class Myplugin {
	constructor(options) {
        this.options = options;
    }
    apply() {
        console.log("apply", this.options)
    }
}
```



## 插件的错误处理

参数校验阶段可以直接 throw 的方式抛出

```throw new Error("Error Message")```

通过 compilation 对象 warning 和 errors 接收

```javascript
compilation.warning.push("warning");
compilation.error.push("error");
```



## 通过 Compilation 进行文件写入

Compilation 上的 assets 可以用于文件写入

- 可以将 zip 资源包设置到 compilation.assets 对象上

文件写入需要使用 webpack-sources



## 插件扩展：编写插件的插件

插件自身也可以通过暴露 hooks 的方式进行自身扩展，以html-webpack-plugin 为例：

- html-webpack-plugin-alter-chunks (Sync)
- html-webpack-plugin-before-html-generation (Async)
- html-webpack-plugin-alter-asset-tags (Async)
- html-webpack-plugin-after-html-processing (Async)
- html-webpack-plugin-after-emit (Async)



## 编写一个压缩构建资源为zip包的插件

要求：

- 生成的 zip 包文件名称可以通过插件传入
- 需要使用 compiler 对象上的特定 hooks 进行资源的生成



### 准备知识： Node.js 里面将文件压缩为 zip 包

使用 jszip(https://www.npmjs.com/package/jszip)

jszip 使用示例

```javascript
var zip = new JSZip();
zip.file("Hello.txt", "Hello World\n");

var img = zip.folder("images");
img.file("smile.gif", imgData, {base64: true})

zip.generateAsync({type: "blob"}).then(function(content) {
    saveAs(content, "example.zip");
})
```



### 复习： Compiler 上负责文件生成的 hooks

Hooks 是emit， 是一个异步的 hook （AsyncSeriesHook）

emit 生成文件阶段，读取的是 compilation.assets 对象的值

- 可以将 zip 资源包设置到 compilation.assets 对象上