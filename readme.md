webpack4 sass babel vue2 标准工程模板
但是，webpack4还是需要n多优化部分，配置下来，实为不易。而实际开发也不需要浪费这个时间——了解即可
> webpack 一直以来最饱受诟病的就是其配置门槛极高，配置内容极其复杂和繁琐，容易让人从入门到放弃，而它的后起之秀如 rollup、parcel 等均在配置流程上做了极大的优化，做到开箱即用，所以webpack 4 也从中借鉴了不少经验来提升自身的配置效率。愿世间再也不需要 webpack 配置工程师。


目前node-sass sass-loader 替换为 sass.js 与sassjs-loader 免除node-sass 安装困难的烦恼


启动:
```bash
npm run start 
```
打包：
```bash
npm run build
```





# 目录结构

# 团队规范
遵从平台发布前端规范标准，节选以下要点：

## 命名规范
遵从Camel命名

### 变量命名规范：

#### js规范，请遵从eslint
+ 常量全部大写，单词间下划线分隔
+ 类采用Pascal命名
### scss 规范
+ css 按照工程结构 嵌套书写，嵌套层级不超过三层——采用 @at-root 
+ 非页面引用scss文件，加前缀 _  如：_fun.scss _mixin.scss

# 构建过程 节选关键步骤
### 构建目录初始化
```bash
mkdir yourFileName
cd yourFileName
```
根据工程目录结构，构建相关文件
……
___
```bash
npm init 
npm install webpack webpack-cli  --save-dev
```
##### 注：--save-dev和--save的区别：
development很明显就是我们开发所需要的依赖包，而打包好上线的话是不需要这些包的，一来各种包加起来太大，二来它只是我们开发提高效率的工具而已；
由于本工程只在本地跑，最终还是sftp自动dist 到服务器，所以暂略

修改package.json ,npm run dev 检查打包结果
```json
{
 "scripts": {
   "dev": "webpack --mode development",
    "build": "webpack --mode production"
  }
}
```
##### 注：webpack4只需要一个--mode选项 指定 production||development
参考http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html
+如果是并行执行（即同时的平行执行），可以使用&符号。
+如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用&&符号。
npm run script1.js & npm run script2.js
npm run script1.js && npm run script2.js


___
#### 配置webpack配置文件 webpack.config.js
##### rule对象参数说明
+ test: A condition that must be met   必须满足的条件
+ exclude: A condition that must not be met  不能满足的条件
+ include: A condition that must be met  必须满足的条件
+ loader: A string of “!” separated loaders   用 “！”分割loaders
+ loaders: An array of loaders as string  loaders的字符串数组

#### 基础loader

```bash
npm install  css-loader style-loader  html-loader url-loader file-loader --save-dev
```

```javascript
    [
                   {
                       test: /\.html$/,
                       use: 'html-loader'
                   },
                   {
                       test: /\.css$/,
                       use: [
                           {
                               loader: 'style-loader',
                               options:{
                                   // singleton:true //处理为单个style标签
                               }
                           },
                           {
                               loader: 'css-loader',
                               options:{
                                   // minimize:true //压缩css
                               }
                           }
                       ]
                   },
                   {
                       test:/\.(png|jpg|jpeg|gif)$/,//图片处理
                       use:[
                           {
                               loader: 'url-loader',
                               options:{
                                   limit:2048,
                                   name:'[name][hash].[ext]'
                               }
                           },
                           {
                               loader: 'file-loader',
                               publicPath:publicPath,
                               outputPath: 'dist/',
                               useRelativePath: true
                           }
                       ]
                   },
                   {
                       test: /\.(woff|woff2|eot|ttf|otf)$/,//字体处理
                       use: ['url-loader']
                   },

    ]
```
#### 配置babel 编译js
```bash
npm install --save-dev  babel-loader @babel/core  @babel/preset-env 
```

```javascript
    [
        {
            test: /\.js$/,
            loader: 'babel-loader',
            exclude: /node_modules/ //设置node_modules里的js文件不用解析
        }
    ]
```

参考：https://segmentfault.com/a/1190000010468759

babel7.0后，需要@ @babel/core vs babel-core  babel插件和版本需要对应上，不然掉坑
参考https://www.w3ctech.com/topic/2150
babel-preset-es2015 babel-plugin-transform-runtime   babel-plugin-add-module-exports babel-plugin-transform-runtime babel-plugin-transform-class-properties

#### 配置eslint 检查

```bash
npm install --save-dev  eslint eslint-loader babel-eslint eslint-plugin-react   eslint-plugin-vue 
```

```javascript
[
   {//eslint 检查
      test: /\.(js|jsx)$/,
      enforce: 'pre',
      loader: ['eslint-loader'],
      exclude: /node_modules/ //设置node_modules里的js文件不用解析
    },
]
```
增加.eslintrc配置
#####  其实没有多大必要，intellij 会自动检车eslint

#### 处理html
npm install html-webpack-plugin 
```javascript
    new HtmlWebpackPlugin({
            filename: './index.html',//输出文件
            template: 'src/index.html',//模板文件
            inject: 'body',//插入位置
            chunks: ['index'],
            hash: true,
            minify: {
                caseSensitive:false,
                removeComment:true,//移除注释
                collapseWhitespace:false//移除多余空格
            }
        })
```
#### 处理图片 - 压缩图片
参考：http://shirmy.me/2018/05/15/webpack-图片、文件处理/
```bash
npm install image-webpack-loader --save-dev 
```

```javascript
    [
        {
            test: /\.(png|jpg|jpeg|gif)$/i,//图片处理
            use: [
                {
                    loader: 'url-loader',
                    options: {
                        limit: 0,//图片不转base64，增加css的阻塞时间，开启http2，所以也不用雪碧图
                        name: '[name].[hash:5].[ext]',
                    }
                },
            ]
        },
        {//压缩图片
            loader: 'image-webpack-loader',
            options: {
                bypassOnDebug: true,
            }
        },
    ]
```



#### 配置webapck server
```bash
npm install webpack-dev-server open --save-dev
```
参看 webpack.server.js 注释

 "start": "node webpack.server.js",
npm start 启动项目

### 配置css优化设置

```bash
npm install --save-dev postcss-loader autoprefixer postcss autoprefixer  mini-css-extract-plugin
```
##### 注：
+ webpack4已经废弃 extract-text-webpack-plugin 这个插件了，现在使用的是 mini-css-extract-plugin
+ 在项目根目录新建postcss.config.js文件，并对postcss进行配置：
```javascript
module.exports = {
    plugins: {
        'autoprefixer': {
            browsers: [
                "> 1%",
                "last 5 versions",
                "not ie <= 9",
                "ios >= 8",
                "android >= 4.0"
            ]
        }
    }
};
```
不然会报出：Error: No PostCSS Config found  

#### 自动消除冗余的css代码
```bash
npm install --save-dev  optimize-css-assets-webpack-plugin 
```
##### 个人觉得css压缩优化空间不大，nginx开启gzip的情况，很有限，有点画蛇添足
#### 配置sass 
```bash
npm install --save-dev  node-sass sass-loader

```
但是node-sass 是个坑货
所以最好换位
```bash
$ npm install --save-dev sassjs-loader sass.js

```
https://www.npmjs.com/package/sass

https://www.npmjs.com/package/sassjs-loader


## webpack构建优化

#### 多线程 happypack 

```bash
npm install --save-dev  happypack

```

配置第三方包，比如jquery
```bash
npm install imports-loader --save-dev
```
```javascript
[
    {
        loader: 'imports-loader',
        options: {
            // 模块为 value，同样webpack也会解析它，如果没有则从alias中解析它
            $: 'jquery'
        }
    }
]
```
#### 增加manifest.json 配置，缓存校对下载, 增加js integrity 安全校验
```bash
npm install --save-dev webpack-subresource-integrity webpack-assets-manifest
```
两个插件准备写成一个，看来不到春节没有时间


#### 增加webpack 模块分析
```bash
npm install --save-dev webpack-bundle-analyzer
```
配置参看 webpack.analy
参考文章：https://www.cnblogs.com/ssh-007/p/7944491.html

#### 增加上传至服务器
```bash
npm install --save-dev webpack-sftp-client
```





#### 配置vue 
```bash
npm install --save-dev vue vue-router vue-loader vue-template-compiler vue-style-loader  
```

```javascript
    [
        {
            test: /\.vue$/,
            use: 'vue-loader'
        },
    ]
```





webpack 相关优化，可参看：https://www.zhoulujun.cn/html/tools/webpack/2016_0218_7492.html
