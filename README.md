### vue-cli-多页面
vue init webpack vue-cli-multipage

#### 最终效果
![Alt text](//pt-starimg.didistatic.com/static/starimg/img/1505396621512L1W0ooE11HriSPhbJrr.png)

#### 多入口打包
```
// webpack 多入口
{
	entry: utils.getEntry(['./src/**/page.js'])
}
```
```
// 获取多页面入口
exports.getEntry = function(globPaths) {
  var entries = {}, pathname;
  for (globPath of globPaths) {
    glob.sync(globPath).forEach(function(entry) {
      pathname = entry.match(/\.\/src\/(.+)\.(html|js)/)[1];
      entries[pathname] = entry;
    });
  }
  console.log(entries);
  return entries;
}
```

####  输出多个页面
`关键：每个页面指定输出打包的内容`
```
// HtmlWebpackPlugin html插件输出多个页面
// 独立个性化template，输出相应page.html
var pages = utils.getEntry(['./src/**/page.html']);
for (var pathname in pages) {
  var conf = {
    filename: pathname + '.html',
    template: pages[pathname],
    inject: true,
   };
  // 该页面中指定输出特定打包结果 重要
  conf.chunks = ['vendor','manifest', pathname];
  module.exports.plugins.push(new HtmlWebpackPlugin(conf));
}
```

#### 多页面通用资源
```
   // keep module.id stable when vender modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
```

#### 每个入口,抽取独立样式
`抽取样式，是每个入口一个，而非所有入口`
```
    new ExtractTextPlugin({
      filename: utils.assetsPath('[name].css')
    }),
```