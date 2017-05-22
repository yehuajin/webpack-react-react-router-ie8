## webpack+react+react-router兼容ie8

 这几天解决了`react`在`ie8`下兼容性，解决了我们网站在`ie8`上打开是空白页的问题。亲测完全可用，我们的网站 [http://bp.microyan.com](http://bp.microyan.com)。可以在`ie8`下看看。需要说明的是，我们网站开发的时候完全没有考虑ie8，我们初衷只是兼容`chrome`。后来做兼容`ie8`，代码上基本上没做修改，主要是修改了配置文件(修改后的配置文件见下文)。

1. 首先`github`上这个教程[react-ie8][1]，我们照着先做一遍，其中对我们网站上的修改包括（**重点是`react`降级到`v0.14`**）。

2. 然后打开我们的`ie8`浏览器，看看网站打不开的报错信息！如图所示:<br />
![ie8错误提示图片](http://oqcb8djlw.bkt.clouddn.com/ie8%E9%94%99%E8%AF%AF.jpg)<br />
上面提到的`github`上，有一份表格，每个错误对应的解决方法，如下图所示：<br />
![ie8错误对应图](http://oqcb8djlw.bkt.clouddn.com/ie8%E8%A7%A3%E5%86%B3%E6%96%B9%E5%BC%8F.jpg)<br />
这次我们看下，我们`ie8`上面的每个错误，在下图中都能找到解决办法！每个错误都解决完，我们的网站就能打开了！

3. **请注意，请注意，请注意，重点来了!**
当我照着我上面的方式一个一个修改的时候，我发现很多错误并不能解决！`ie8`还是打不开！这是为什么？听我慢慢道来：我们的项目代码分为两部分（我们自己写的代码和`node_modules`里面引用的外部代码），按照这个`github`上的教程，我们只能解决我们自己代码中的兼容性，并不能解决`node_modules`中应用的代码的兼容性！
比如：`Exception thrown and not caught`这个问题，他给的解决方法是：*把 require('es5-shim') require('es5-shim/es5-sham') 插入到入口文件的最上方，并且在代码中不要使用 export * from 'xxx'*。
这样使用其实可以解决我们代码中的兼容性，但是`node_modules`里面应用的外部库的兼容并不能解决。那我们应该怎么办？
我们把这些编译文件放到`webpack`配置文件中，让webpack打包的时候，也去转义`node_modules`里面的文件！
比如`es5-shim`这个库，我们的引用方式如下面的配置文件，我们的`es3ify`就使用[es3ify-webpack-plugin](https://github.com/BryceHQ/es3ify-webpack-plugin)来代替，全部都集成到`webpack`配置文件中。


### 本文重要思想：不要仅仅考虑自己写的代码，也需要考虑`package.json`中引用的别的文件！！！

**其他的问题，我们可以自行搜索解决。对了我们当时还删除了`webpack-hot-middleware`,因为不兼容好像是～**


具体配置文件如下：
```javascript
var path = require('path');
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var es3ifyPlugin = require('es3ify-webpack-plugin');
module.exports = {
    devtool: 'inline-source-map',
    entry: [
        "es5-shim", "es5-shim/es5-sham", 'babel-polyfill', './src/index'
    ],
    output: {
        path: path.join(__dirname, 'dist'),
        filename: 'bundle.js',
        publicPath: 'http://localhost:3000/'
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(),
        new webpack.NoErrorsPlugin(),
        new webpack.DefinePlugin({
            "process.env": {
                NODE_ENV: JSON.stringify('development')
            }
        }),
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: path.join(__dirname, 'assets/index-template.html')
        }),
        new es3ifyPlugin()
    ],
    resolve: {
        extensions: [
            '', ".js"
        ],
        root: path.join(__dirname, 'src')
    },
    module: {
        postLoaders: [
            {
                test: /\.js$/,
                loaders: ['export-from-ie8/loader']
            }
        ],
        loaders: [
            {
                test: /\.css$/,
                loader: 'style-loader!css-loader'
            }, {
                test: /\.js$/,
                loaders: ['babel-loader?cacheDirectory'],
                include: path.join(__dirname, 'src')
            }, {
                test: /\.styl$/,
                loaders: ['style-loader', 'css-loader', 'stylus-loader']
            }, {
                test: /\.(png|jpg|gif|svg)$/,
                loader: 'url-loader?limit=1'
            }, {
                test: /\.(htc)$/,
                loader: 'url-loader?limit=1'
            }
        ]
    }
};

```

[1]: https://github.com/xcatliu/react-ie8#cn-make-your-react-app-work-in-ie8
