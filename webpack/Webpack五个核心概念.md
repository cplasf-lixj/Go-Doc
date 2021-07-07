## 1、Webpack五个核心概念

### 1.1、入口（entry)

**入口起点(entry point)** 指示 webpack 应该使用哪个模块，来作为构建其内部 [依赖图(dependency graph)](https://webpack.docschina.org/concepts/dependency-graph/) 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 `./src/index.js`，但你可以通过在 [webpack configuration](https://webpack.docschina.org/configuration) 中配置 `entry` 属性，来指定一个（或多个）不同的入口起点。

### 1.2、 输出(output) 

**output** 属性告诉 webpack 在哪里输出它所创建的 *bundle*，以及如何命名这些文件。主要输出文件的默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist` 文件夹中。

### 1.3、loader

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader** 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 [模块](https://webpack.docschina.org/concepts/modules)，以供应用程序使用，以及被添加到依赖图中。

在更高层面，在 webpack 的配置中，**loader** 有两个属性：

1. `test` 属性，识别出哪些文件会被转换。
2. `use` 属性，定义出在进行转换时，应该使用哪个 loader。

***PS: webpack是一个领导，很多具体的事情需要安排手下去做，而loader就好比领导知道哪些事情谁可以去做。***

### 2.4、插件(plugin)

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

想要使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建一个插件实例。

### 2.5、模式(mode)

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`。

生产环境比开发环境多一个压缩js代码的功能。

````js
const { resolve } = require('path');

module.exports  {
  // 入口七点
  entry: './src/index.js',
  // 输出
  output: {
    // 输出文件名
    filename: 'bundle.js',
    // 输出路径
    path: resolve(__dirname, 'dist')
  },
  // loader的配置
  module: {
    rules: [
       {
         // 匹配规则
         test: /\.css$/,
         // 使用loader信息
         use: [
           // 创建style标签，将js中的样式资源插入到head中
           'style-loader',
           // 将css文件变成commonjs模块加载js中，里面内容是样式字符串
           'css-loader'
         ]
       }
    ]
  },
  // plugins的配置
  plugins: [
    
  ],
  // 模式
  mode: 'development', // 开发模式
  // mode: 'production', // 成产模式
}