## 环境
 - vue@2.6.10
 - vue-cli@4.10  
 
## UglifyJS Webpack Plugin  

> 一个依赖[ Uglify.js ](https://github.com/mishoo/UglifyJS)来优化和压缩`javascript`代码的`webpack`插件，此插件要求的最小版本为`Node@6.9.0`和`Webpack@4.0.0`版本。  

#### 安装  

```shell
npm install --save-dev uglifyjs-webpack-plugin
```  

#### 使用  

```javascript
// vue.config.js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
	...
	configureWebpack: config => {
		config.plugins.push(
	      new UglifyJsPlugin({
	        uglifyOptions: {
	          compress: { // 参Uglify.js参数
	            warnings: true, // 警示存在安全风险的代码
	            drop_debugger: true, // 是否过滤调试(debugger)代码
	            drop_console: true, // 是否过滤输出(console)代码
	          },
	        },
	        sourceMap: false, // 是否启用文件缓存
	        parallel: true  // 使用多进程并行运行来提高构建速度
	      })       
	 	)
}
```  

[更多`compress`参数](http://lisperator.net/uglifyjs/compress)  

[官方参数文档](https://www.html.cn/doc/webpack2/plugins/uglifyjs-webpack-plugin/)