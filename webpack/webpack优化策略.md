## 1、Webpack优化

## 1.1、 分析工具

#### 1.1.1、体积分析

##### 1.1.1.1、初级分析

通过官方提供的stat.json文件分析打包结果，stat.json文件可以通过下面的语句快速生成：

````shell
webpack --profile --json > stats.json
````

通过官网[stats.json 分析工具](http://webpack.github.io/analyse/)进行分析，上传stats.json文件后，可以得到分析结果，其中包括 `webpack` 的版本、打包时间、打包过程的 `hash` 值、模块数量( `modules` )、`chunk` 数量、打包生层的静态文件 `assets` 以及打包的警告和错误数。

##### 1.1.1.2、第三方工具

[webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 是打包分析神器，在webpack.config.js的插件中使用

#### 1.1.2、速度分析

通过 [speed-measure-webpack-plugin](https://github.com/stephencookdev/speed-measure-webpack-plugin) 插件分析整个打包的总耗时，以及每一个`loader` 和每一个 `plugins` 构建所耗费的时间，从而快速定位到可以优化 `Webpack` 的配置。

### 1.2、优化策略

#### 1.2.1、体积优化

##### 1.2.1.1、js压缩

`webpack4.0` 默认在生产环境的时候是支持代码压缩的，即 `mode=production` 模式下。

实际上 `webpack4.0` 默认是使用 [terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin) 压缩插件，在此之前是使用 [uglifyjs-webpack-plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)，两者的区别是后者对 ES6 的压缩不是很好，同时我们可以开启 `parallel` 参数，使用多进程压缩，加快压缩。

##### 1.2.1.2、CSS压缩

我们可以借助 `optimize-css-assets-webpack-plugin` 插件来压缩 `css`，其默认使用的压缩引擎是 `cssnano`。 具体使用如下：

````js
const OptimizeCSSAssetsPlugin = require("optimize-css-assets-webpack-plugin");

module.exports = {
  plugins: [
    new OptimizeCSSAssetsPlugin({
      assetNameRegExp: /\.optimize\.css$/g,
      cssProcessor: require('cssnano'),
      cssProcessorPluginOptions: {
        preset: ['default', { discardComments: { removeAll: true } }],
      },
      canPrint: true
    })
  ]
}
````

**擦除无用的 `CSS`**

使用 `PurgeCSS` 来完成对无用 `css` 的擦除，它需要和 `mini-css-extract-plugin` 配合使用。

##### 1.2.1.3、图片压缩

首先要做的就是对于图片的优化，手动的去通过线上的图片压缩工具，如 [tiny png](https://tinypng.com/) 来压缩图片。

在项目中借助 [image-webpack-loader](https://github.com/tcoopman/image-webpack-loader)实现图片自动压缩，它是基于 [imagemin](https://github.com/imagemin/imagemin) 这个 Node 库来实现图片压缩的。

````js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[name]_[hash].[ext]',
              outputPath: 'images/',
            }
          },
          {
            loader: 'image-webpack-loader',
            options: {
              // 压缩 jpeg 的配置
              mozjpeg: {
                progressive: true,
                quality: 65
              },
              // 使用 imagemin**-optipng 压缩 png，enable: false 为关闭
              optipng: {
                enabled: false,
              },
              // 使用 imagemin-pngquant 压缩 png
              pngquant: {
                quality: '65-90',
                speed: 4
              },
              // 压缩 gif 的配置
              gifsicle: {
                interlaced: false,
              },
              // 开启 webp，会把 jpg 和 png 图片压缩为 webp 格式
              webp: {
                quality: 75
              }
            }
          }
        ]
      },
    ]
  }  
}
````

##### 1.2.1.4、拆分代码

使用 `splitChunksPlugin` 把一个大的文件分割成几个小的文件，可以有效的提升 `webpack` 的打包速度

#### 1.2.2、 速度优化

##### 1.2.2.1、分离两套配置

在开发阶段：我们需要 `webpack-dev-server` 来帮我们进行快速的开发，同时需要 **HMR 热更新** 帮我们进行页面的无刷新改动，而这些在 **生产环境** 中都是不需要的。

在生产阶段：我们需要进行 **代码压缩**、**目录清理**、**计算 hash**、**提取 CSS** 等等；

实现起来很简单，我们前面也提到过，就新建三个 `webpack` 的配置文件就行：

- `webpack.dev.js`：开发环境的配置文件
- `webpack.prod.js`：生产环境的配置文件
- `webpack.common.js`：公共配置文件

通过 `webpack-merge` 来整合两个配置文件共同的配置 `webpack.common.js`

##### 1.2.2.2、减少查找过程

对 `webpack` 的 `resolve` 参数进行合理配置，使用 `resolve` 字段告诉 `webpack` 怎么去搜索文件。

**合理使用 `resolve.extensions`**

在导入语句没带文件后缀时，`webpack` 会自动带上后缀后去尝试询问文件是否存在，查询的顺序是按照我们配置 的 `resolve.extensions` 顺序从前到后查找，`webpack` 默认支持的后缀是 `js` 与 `json`。

**优化 `resolve.modules`**

这个属性告诉 `webpack` 解析模块时应该搜索的目录，绝对路径和相对路径都能使用。使用绝对路径之后，将只在给定目录中搜索，从而减少模块的搜索层级：

```js
// config/webpack.common.js
const commonConfig = {
  resolve: {
    extensions: ['.js', '.jsx'],
    mainFiles: ['index', 'list'],
    alias: {
      alias: path.resolve(__dirname, '../src/alias'),
    },
    modules: [
      path.resolve(__dirname, 'node_modules'), // 指定当前目录下的 node_modules 优先查找
      'node_modules', // 将默认写法放在后面
    ]
  }
}
```

**使用 `resolve.alias` 减少查找过程**

`alias` 的意思为 **别名**，能把原导入路径映射成一个新的导入路径。

##### 1.2.2.3、缩小构建目标

排除 `Webpack` 不需要解析的模块，即使用 `loader` 的时候，在尽量少的模块中去使用。

可以借助 `include` 和 `exclude` 这两个参数，规定 `loader` 只在那些模块应用和在哪些模块不应用。

修改公共配置文件 `webpack.common.js`：

```js
// config/webpack.common.js
const commonConfig = {
  module: {
    rules: [
      { 
        test: /\.js|jsx$/, 
        exclude: /node_modules/,
        include: path.resolve(__dirname, '../src'),
        use: ['babel-loader']
      }
    ]
  },
}
```

##### 1.2.2.4、利用多线程提升构建速度

由于运行在 `Node.js` 之上的 `webpack` 是单线程模型的，所以 `webpack` 需要处理的事情需要一件一件的做，不能多件事一起做。

如果 `webpack` 能同一时间处理多个任务，发挥多核 `CPU` 电脑的威力，那么对其打包速度的提升肯定是有很大的作用的。

##### `HappyPack`

原理：每次 `webapck` 解析一个模块，`HappyPack` 会将它及它的依赖分配给 `worker` 线程中。处理完成之后，再将处理好的资源返回给 `HappyPack` 的主进程，从而加快打包速度。 

![](https://segmentfault.com/img/bVcLbDn )

将 `HappyPack` 引入配置文件，将相应的 `loader` 替换成 `happypack/loader`，同时将被替换的 `loader` 放入其插件的 `loaders` 选项，我们暂且替换一下 `babel-loader`：

`````js
const HappyPack = require('happypack');

module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: [
          'happypack/loader'
          // 'babel-loader'
        ]
      }
    ]
  },
  plugins: [
    new HappyPack({
      loaders: ['babel-loader']
    })
  ]
}
`````

更多参数可以参考 [HappyPack 官网](https://github.com/amireh/happypack)

##### `thread-loader`

`webpack` 官方推出的一个多进程方案，用来替代 `HappyPack`。

原理和 `HappyPack` 类似，`webpack` 每次解析一个模块，`thread-loader` 会将它及它的依赖分配给 `worker` 线程中，从而达到多进程打包的目的。

使用很简单，直接在我们使用的 `loader` 之前加上 `thread-loader` 就行

````js
module.exports = {
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: [
					{
            loader: 'thread-loader',
            options: {
              workers: 3,// 开启几个 worker 进程来处理打包，默认是 os.cpus().length - 1
            }
          },
          'babel-loader'
        ]
      }
    ]
  }
}
````

##### 1.2.2.5、预先编译资源模块（DllPlugin）

一般来说第三方模块是不会变化的，所以只要在第一次打包的时候去打包一下第三方模块，并将第三方模块打包到一个特定的文件中，当第二次 `webpack` 进行打包的时候，就不需要去 `node_modules` 中去引入第三方模块，而是直接使用我们第一次打包的第三方模块的文件就行。

`webpack.DllPlugin` 就是来解决这个问题的插件。

1、在配置文件目录 `config` 下新建一个 `webpack.dll.js`，此文件用于将我们的第三方包文件打包到 `dll` 文件夹中去：

```js
// config/webpack.dll.js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  mode: 'production', // 环境
  entry: {
    vendors: ['lodash'], // 将 lodash 打包到 vendors.js 下
    react: ['react', 'react-dom'], // 将 react 和 react-dom 打包到 react.js 下
  },
  output: {
    filename: '[name].dll.js', // 输出的名字
    path: path.resolve(__dirname, '../dll'), // 输出的文件目录
    library: '[name]' // 将我们打包出来的文件以全部变量的形式暴露，可以在浏览器变量的名字进行访问
  },
  plugins: [
    // 对生成的库文件进行分析，生成库文件与业务文件的映射关系，将结果放在 mainfest.json 文件中
    new webpack.DllPlugin({
      name: '[name]', // 和上面的 library 输出的名字要相同
      path: path.resolve(__dirname, '../dll/[name].manifest.json'),
    })
  ]
}
```

- 上面的 `library` 的意思其实就是将 `dll` 文件以一个全局变量的形式导出出去，便于接下来引用
- `mainfest.json` 文件是一个映射关系，它的作用就是帮助 `webpack` 使用我们之前打包好的 `***.dll.js` 文件，而不是重新再去 `node_modules` 中去寻找。

2、修改公共配置文件 `webpack.common.js`，借助一个插件 `add-asset-html-webpack-plugin`，将之前生成的 `dll` 文件导入到 `html` 中去。

```js
// config/webpack.common.js
const webpack = require('webpack');
const AddAssetHtmlWebpackPlugin = require('add-asset-html-webpack-plugin');

const commonConfig = {
  plugins: [
    // ...
    new AddAssetHtmlWebpackPlugin({
      filepath: path.resolve(__dirname, '../dll/vendors.dll.js')
    }),
    new AddAssetHtmlWebpackPlugin({
      filepath: path.resolve(__dirname, '../dll/react.dll.js')
    }),
    new webpack.DllReferencePlugin({
      manifest: require(path.resolve(__dirname, '../dll/vendors.dll.mainfest.json'))
    }),
    new webpack.DllReferencePlugin({
      manifest: require(path.resolve(__dirname, '../dll/react.dll.mainfest.json'))
    }),
  ]
}
```

##### 1.2.2.6、缓存Cache相关

开启相应 `loader` 或者 `plugin` 的缓存，来提升二次构建的速度。一般我们可以通过下面几项来完成：

- `babel-loader` 开启缓存
- `terser-webpack-plugin` 开启缓存
- 使用 `cache-loader` 或者 [hard-source-webpack-plugin](https://github.com/mzgoddard/hard-source-webpack-plugin)

如果项目中有缓存的话，在 `node_modules` 下会有相应的 `.cache` 目录来存放相应的缓存。
