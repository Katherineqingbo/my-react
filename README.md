### 一、创建一个新的APP
```
$sudo npm install -g create-react-app
$sudo create-react-app my-app
$cd my-app
$npm start
```
npm 5.2.0以以上版本的话
```
$npx create-react-app my-app
$cd my-app
$npm start
```
[摘自](https://www.cnblogs.com/DDante/p/452b65f76126caad768edc007ece85f7.html)

#从零开始搭建一个react项目

```
如果只是想试试 React，那么建议使用 create-react-app来创建一个react项目。快速开始 
因为 create-react-app 和 vue-cli 不一样，create-react-app将webpack的相关配置直接封装好了，所以自定制化程度不高，所以考虑手动构建一个 React项目
```
## 准备工作

```
安装node环境。
配置cnpm(看个人需求)。
准备一个空的文件夹react-demo。
```
## 初始化工程

### 1. 初始化工程目录

```
$cd react-demo
$npm init
```
一路回车，我们将得到一个最简单的npm目录，会包含一个package.json。

```
// package.json
{
 "name": "react-demo",
 "version": "1.0.0",
 "description": "",
 "main": "index.js",
 "scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
 },
 "author": "",
 "license": "ISC"
}
```
然后在react-demo目录下新建一个程序的主目录" src " 目录。mddir src
### 2. 配置工程
#### 1. webpack初体验
首先安装webpack和webpack-cli
```
$npm install --save-dev webpack webpack-cli -g
```
##### 新建并配置webpack
新建webpack.config.js 文件。首先要理解webpack的几个基础概念入口(entry)、出口(output)、载入器(loader)、插件(plugins)、模式(mode) 。

[webpack中文文档](https://www.webpackjs.com/concepts/)

```
// __dirname是node.js中的一个全局变量，它指向当前执行脚本所在的目录
// path是node.js中提供的处理文件路径的小工具。 (http://www.runoob.com/nodejs/nodejs-path-module.html)
const path = require('path');
module.exports = {
    // 项目入口，webpack从此处开始构建
    entry: {
        main: path.join(__dirname, 'src/index.js'), // 指定入口，可以指定多个。参考webpack文档
    },
    output: {
        path: path.join(__dirname, "dist"), // bundle生成(emit)到哪里
        filename: "bundle.js", // bundle生成文件的名称
    },
}
```
    - 这样就完成了最简单的webpack配置文件。相应的我们需要在src目录下新建一个index.js 文件。
    - 接下来在命令行输入 webpack --config ./webpack.config.js就可以输出dist文件夹。
    - 为了方便起见，通常我们会在package.json里配置脚本命令。在scripts标签下，添加一句"build": "webpack --config ./webpack.config.js"这样，我们就可以通过npm run build完成webpack打包。
##### 配置开发应用服务器
正常情况下，我们需要以应用服务器打开我们的网页，webpack-dev-server提供了一个简单的web服务器，并且能够实时重新加载。指南 首先需要安装webpack-dev-server npm install -D webpack-dev-server。 
接下来，修改配置文件，告诉开发服务器，在哪里寻找文件：

```
// webpack.config.js
module.exports = {
    devServer: {
        contentBase: './dist'
    }
}
```
这段配置告诉webpack-dev-server，在默认host和port下建立服务，并将contentBase目录下的目录，作为可访问文件。 
接下来让我们的服务器跑起来，在package.json配置如下的命令脚本：

```
"scripts" : {
    "start": "webpack-dev-server --mode development --open",
    "build: "webpack --mode production --config ./webpack.config.js"
}
```
其中mode是上文中提到模式概念，webpack会有相应的内置优化。
##### Babel & React
ES6已经极为流行了，不过目前仍有浏览器不兼容。同时react的jsx语法，也需要babel来将其转化为能兼容的js代码。

```
$npm install react react-dom react-router-dom -S
$npm install babel-core babel-loader babel-preset-env babel-preset-react -D
```
安装完之后，我们需要在webpack中配置使其生效。在webpack.config.js 的module中添加rules规则，如下：

```
module.exports = {
    // ...省略
    module: {
        rules: [

{
    test: /\.(js|jsx)$/,
    exclude: /node_modules/,
    enforce: 'pre',
    use: [{
        loader: 'babel-loader',
    }, {
        loader: 'eslint-loader', // 指定启用eslint-loader
        options: {
            formatter: require('eslint-friendly-formatter'),
            emitWarning: false
        }
    }]
},
    ]
}
```
同时考虑到，后期可能会别的规则需求，例如使用antDesign的按需引入，我们将babel的配置提出来，在根目录下新建文件 .babelrc，并书写以下代码。

```
{
  "presets": ["env","react"],
  // antd按需引入
  // "plugins": ["react-hot-loader/babel", ["import", { "libraryName": "antd", "libraryDirectory": "es","style": "css" }], "transform-runtime"]
}
```
到这里，我们就完成babel的相关配置，并且安装了react相关依赖，可以书写jsx语法了。
##### 关于样式
前置安装less-loader/scss-loader、style-loader、css-loader、postcss-loader。

    - less-loader是将less编译成css的loader
    - postcss是将css加上浏览器Hack的loader
    - css-loader用于在js中import、require等方法引入css
    - style-loader用于将css最终写入html文件。

```
test: /\.(css|less)$/,
exclude: /node_modules/,
include: /src/,
use: [
    {loader: "style-loader"},
    {
        loader: 'css-loader',
        options: {
            minimize: process.env.NODE_ENV === 'production',
            importLoaders: 2,
            localIdentName: '[name]-[local]-[hash:base64:5]',
            modules:true
        }
    }, {
        loader: 'postcss-loader',
        options: {           // 如果没有options这个选项将会报错 No PostCSS Config found
            plugins: (loader) => [
                require('autoprefixer')(), //CSS浏览器兼容
            ]
        }
    },{
        loader: 'less-loader',
        options: {
            javascriptEnabled: true,
        }
    }],
```
其中test对应的可以匹配的正则文件，include是需要编译的目录，exclude是跳过的目录，use里面可以书写跟loader相关的配置。

至此，完成了一个简单的react工程的配置。只包含有js和css相关的内容。
# 工程优化
##### ExtractTextWebapckPlugin
在上面的配置中，我们没有单独打包样式文件，样式文件会被打包在js里面。现在通过ExtractTextWebpackPlugin单独打包样式文件。npm install -D extract-text-webapck-plugin引入依赖，配置如下：

```
$npm install -D extract-text-webapck-plugin
```

```
// module-> rules
{
    test: /\.less$/,
    exclude: /node_modules/,
    include: /src/,
    // loader:['style-loader','css-loader']
    use: ExtractTextWebapckPlugin.extract({
        fallback:'style-loader',
        use: [
            {
                loader: 'css-loader',
                options: {
                    minimize: process.env.NODE_ENV === 'production',
                    importLoaders: 2,
                    localIdentName: '[name]-[local]-[hash:base64:5]',
                    modules:true
                }
            }, {
                loader: 'postcss-loader',
                options: {           // 如果没有options这个选项将会报错 No PostCSS Config found
                    plugins: (loader) => [
                        require('autoprefixer')(), //CSS浏览器兼容
                    ]
                }
            },{
                loader: 'less-loader',
                options: {
                    javascriptEnabled: true,
                }
            }],
        }
    )
},
 // plugins 下新增
new ExtractTextWebapckPlugin({
    filename: 'css/[name]-[hash].css',
    // Setting the following option to `false` will not extract CSS from codesplit chunks.
    // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
    // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`,
    allChunks: true
}),
```
##### 为打包后的文件增加hash
如果浏览器加载发现远端文件没有发生变化时，将会启用缓存，导致新修改的页面并没有同步，这时候为了避免缓存，我们就需要让每次打包后的文件有不同的文件名，以减少缓存。

```
// webpack.config.js -> output
output: {
    path: path.join(__dirname, "dist"),
    publicPath: '/',
    filename: "js/[name]-[hash]" + ".js",
    chunkFilename: "js/[name]-[hash]" + ".js",
},
```
##### 打包静态文件
当页面图片较多时，会发送很多http请求，降低页面性能。url-loader引入图片编码，生成dataURI，把图片翻译成一串字符串。再把字符串打包到文件中，最终只需要引入文件就可以访问图片了。但是当图片较大时，编码会消耗性能。因此url-loader提供了一个limit参数，小于limit的文件会被转为dataURI，大于limit会使用file-loader传入。首先引入依赖npm install -D file-loader url-loader

```
// module->rules
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        limit: 10000,
        name: 'static/img/[name].[hash:7].[ext]'
    }
},
{
    test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        limit: 10000,
        name: 'static/media/[name].[hash:7].[ext]'
    }
},
{
    test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        limit: 10000,
        name: 'static/fonts/[name].[hash:7].[ext]'
    }
},
```
##### html-webpack-plugin
这是一个 webpack 插件，为我们在生成有 hash 标识符的 css,js时非常方便，它需要我们指定一个模板，然后它会生成一个自动引入我们生成的文件的新模板。 
首先安装依赖npm install -D html-webpack-plugin ，webpack配置如下：
```
const HtmlWebPackPlugin = require('html-webpack-plugin')
// plugins下添加
new HtmlWebPackPlugin({
    template: './src/index.html',
    minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
    },
    filename: 'index.html'
}),
```
##### devServer

```
// webpack.config.js
devServer: {
    // contentBase: path.join(__dirname, ""),
    contentBase: false, //since we use CopyWebpackPlugin.
    clientLogLevel: 'warning',
    publicPath: '/',
    hot: true,
    progress: true,
    overlay: { warnings: false, errors: true },
    historyApiFallback: {
        rewrites: [
            { from: /.*/, to: path.posix.join('/', 'index.html') },
        ],
    },
    // historyApiFallback: true,
    // quiet: true, // necessary for FriendlyErrorsPlugin
    compress: true,
    inline: true,
    port: 8083,
    host: '127.0.0.1',
    watchOptions: {
        poll: false,
    }
},
```
因为配置了contentBase = false，所以使用CopyWebpackPlugin，还是先安装依赖npm install -D copy-webpack-plugin，然后代码如下：

```
// plugins
// copy custom static assets
new CopyWebpackPlugin([
    {
        from: path.resolve(__dirname, './src/static'),
        to: 'static',
        ignore: ['.*']
    }
]),
```
##### 提取公共代码
利用webpack4的splitChunks来分割代码，配置如下：

```
// webpack.config.js
//4.0配置
optimization: {
    /*splitChunks: {
        chunks: 'all',//"initial" | "async" | "all"
        cacheGroups: {
            default: false,
            vendors: false,
        },
    },*/
    /*splitChunks: {
        cacheGroups: {
            commons: {
                test: /[\\/]node_modules[\\/]/,
                name: "vendor",
                chunks: "all"
            }
        }
    }*/
    runtimeChunk: {
        name: "manifest"
    },
    splitChunks: {
        cacheGroups: {
            commons: {
                test: /[\\/]node_modules[\\/]/,
                name: "vendor",
                chunks: "all"
            }
        }
    }
},
```
