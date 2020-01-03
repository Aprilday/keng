## 学习Webpck过程中的一些知识点

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
