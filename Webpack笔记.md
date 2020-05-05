#### 学习Webpck过程中的一些知识点

- npm初始化可以使用 npm init -y 
> -y 的作用的是初始化选项全部使用默认值，省事

- 在终端输入 which node 可以查看node的安装路径

- 文件监听，有两种方式
> - 启动 webpack 命令时，带上 --warch 参数 （但需要手动刷新浏览器）
> - 在配置 wbepack.config.js 中设置 watch: true

- 热更新： webpack-dev-server
> 使用 HotModuleReplacementPlugin 插件

- 热更新2: webpack-dev-middleware
```js
    const express = require('express');
    const webpack = require('webpack');
    const webpackDevMiddleware = require('webapck-dev-middleware');

    const app = express();
    const config = require('./webpack.config.js');
    const compiler = webpack(config);

    app.use(webpackDevMiddleware(compiler, {
        publicPath: config.output.publicPath
    }));

    app.listen(3000, () => {
        console.log('app listening on port 3000!\n');
    })
```
- 文件指纹生成
> - Hash：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改
> - Chunkhash：和 webpack 打包的 chunk 有关，不同的 entry 会生成不同的 chunkhash 值
> - Contenthash：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变

- CSS 的文件指纹设置
> 设置 MiniCssExtractPlugin 的 filename，使用 [contenthash]

- clean-webpack-plugin 可以在构建前，删除output指定输出目录  
 
- css预处理，添加前缀
> 安装postcss-loader 和 autoprefixer 两个插件，示例代码如下  
```js
    {
        loader: 'postcss-loader',
        options: {
            plugins: () => [
                require('autoprefixer')({
                    browsers: ['last 2 version', '>1%', 'ios 7']
                })
            ]
        }
    }
```  
- css px 自动转换为 rem
> px2rem-loader（开发环境） lib-flexible（引入代码，需前置引入，因为要计算页面大小，从而给出根元素的font-size）
```js
    {
        loader: 'px2rem-loader',
        options: {
            remUnit: 75, // 转换单位 1rem=75px 适合750的设计稿
            remPrecesion: 8 // px转化为rem小数点后的位数
        }
    }
```  
- 资源内联
> - 页面框架的初始化脚本  
> - 上报相关打点  
> - css内联避免页面闪动
> - 小图片或者字体内联（url-loaders），减少网络请求数  

```js
    // row-laoder 内联 html 可以一些meta标签抽出来，做成公共的文件来内联引入
    ${require('raw-loader!./meta.html')}
    // raw-loader 内联 JS 比如刚才需要在页首内联引入的库，就可以以下面这种形式来书写,如果内联脚本里包含了ES6之类的新的规范的代码，需要用babel来进行一次转换
    <script>${require('raw-loader!babel-loader!../node_modules/lib-flexble')}
    
    </script>
```  
- CSS 内联
> - 方案一、借助 style-loader
> - 方案二、html-inline-css-webpack-plugin  

- 多页面通用打包方案
> glob库的使用
> 以一定的规则来设置目录和文件名
> 动态输出 entry 以及 htmlWebpackPlugin 的内容即可 

- devtool 设置不同的值，可以生成不同的source-map的文件，一下为几种可以设置的值
> - eval：使用eval包裹模块代码
> - source-map：产生.map文件
> - cheap-source-map：不包含列信息，代码错误等只能定位到行
> - inline-source-map：将.map作为DataURI嵌入，不单独生成.map文件
> - module：包含loader的sourcemap

- 提取页面公共资源
> SplitChunksPlugin
> Webpack 4内置，替代CommonsChunkPlugin插件
```js
    module.exports = {
        optimization: {
            splitChunks: {
                chunks: 'async',
                minSize: 30000, // 被引用的文件最小为多大才会被抽取
                maxSize: 0,
                minChunks: 1,
                maxAsyncRequests: 5,
                maxInitialRequests: 3,
                automaticNameDelimiter: '~',
                name: true,
                cacheGroups: {
                    vendors: {
                        test: /[\\/]node_modules[\\/]/,
                        priority: -10
                    }
                }
            }
        }
    }
```
> chunks参数说明：
> - async 异步引入的库进行分离（默认）
> - initial 同步引入的库进行分离
> - all 所有引入的库进行分离（推荐）

> - 用法
```js
    // 老的用法
    const HtmlWebpackExternalsPlugin = require(html-webpack-externals-plugin);
    new HtmlWebpackExternalsPlugin({
        externals: [
            {
                module: 'react',
                entry: 'cdn地址',
                global: 'React'
            },
            {
                module: 'react-dom',
                entry: 'cdn地址',
                global: 'ReactDOM'
            }
        ]
    })

    // splitChunks用法，此时需要把 vendors 添加到 htmlWebpackPlugin的chunks数组中
    optimization: {
        splitChunks: {
            cacheGroups: {
                commons: {
                    test: /(react|react-dom)/,
                    name: 'vendors',
                    chunks: 'all'
                }
            }
        }
    }
```

- tree shaking
> mode 值为 production 自动开启 tree-shaking，设置为 none 可以取消 tree-shaking
> DCE(Elimination)
>> - 代码不会被执行
>> - 代码执行的结果不会被用到
>> - 代码只会影响死变量（只写不读）

> 只支持ES6模块，不支持cjs
原理
> 利用ES6模块的特点
>> - 只能作为模块顶层的语句出现
>> - import 的模块名只能是字符串常量
>> - import biding 是 immutable的
> 代码擦除：uglify 阶段删除无用代码

- scope hoisting （必须是ES6代码，cjs不支持）
> 原理：将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当地重命名一些变量以防止变量名冲突
> 通过 scope hoisting 可以减少函数声明代码和内存开销

- Webpack4 动态按需加载JS文件，动态import  
> - 安装babel插件  
>> npm install @babel/plugin-syntax-dynamic-import --save-dev  
> - .babelrc  
>> plugin: ["@babel/plugin-syntax-dynamic-import"]  
> 可以在代码中动态加载文件，返回一个promise，示例如下
```js
    import('../test.js').then(res => {
        // res为文件返回的内容
    })
```

- 制定团队的 ESLint 规范
> 比较典型的是 airbnb 的规范  
> eslint-config-airbnb eslint-config-airbnb-base

- Webpack 打包发布一个库
> ...

- Webpack ssr 打包存在的问题  
> - 浏览器的全局变量（Node.js中没有 document，window）
>> - 组件适配：将不兼容的组件根据打包环境进行适配
>> - 请求适配：将 fetch 或者 ajax 发送请求的写法改成 isomorphic-fetch 或者 axios
> - 样式问题（Node.js无法解析css）
>> - 方案一、服务端打包通过 ignore-loader 忽略掉css的解析
>> - 方案二、将 style-loader 替换成 isomorphic-style-loader

- 一个构建输出日志的插件 friendly-errors-webpack-plugin
> 可以输出比较友好直观的构建信息，推荐使用

- webpack配置文件中可以设置 stats 属性来控制输出信息的类型，可以结合 'friendly-errors-webpack-plugin' 插件来优化输出信息的格式和内容

- 团队对于构建配置的选择
> 通过多个配置文件管理不同环境的webpack配置
>> 基础配置：webpack.base.js 
>> 开发环境：webpack.dev.js
>> 生产环境：webpack.prod.js
>> SSR环境：webpack.ssr.js

- 发布包到 npm
> 添加用户
>> npm adduser
> 升级版本
>> 升级补丁版本号：npm version patch
>> 升级小版本号：npm version minor
>> 升级大版本号： npm version major
> 发布版本
>> npm publish

- webpack 构建速度分析：使用 speed-measure-webpack-plugin
```js
    const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
    cosnt smp = new SpeedMeasurePlugin();

    const webpackConfig = smp.wrap({
        plugins: [
            new MyPlugins(),
        ]
    })
```
- webpack 构建体积分析：webpack-bundle-analyzer
```js
    // 构建完成后在本地 8888 端口展示信息
    const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
    module.exports = {
        plugins: [
            new BundleAnalyzerPlugin(),
        ]
    }
```

- 多进程/多实例构建：资源并行解析可选方案
> thread-loader
>> parallel-webpack
>> HappyPack

- 多进程/多实例：并行压缩
> 方法一、使用 parallel-uglify-plugin 插件
> 方法二、uglifyjs-webpack-plugin 开启 parallel 参数

- 使用 DLLPlugin 进行分包

- 擦除没有用到的css属性
> 使用 purge-css-plugin

- 图片压缩
> 基于 Node 库的 imagemin 或者 tinypng API
> 配置 image-webpack-loadersimage-webpack-loaders

- 动态 Polyfill
> 使用 Polyfill service
> polyfill.io官方提供的服务

- loader
> loader只是导出为函数的JavaScript模块

```js
    module.exports = function(source) {
        return source;
    }
```

> 多个loader串行执行，顺序从后到前