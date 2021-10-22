## 前言
**webpack** 是我经常用到的工具，并且 **webpack** 也逐渐成为主流前端构建工具，所以我觉得有必要透彻理解它，由于配置实现的写法有很多种，让人很头疼，一直没有仔细研究一下他的原理和实现方式。通过这篇文章，全面了解 **webpack**。  

## webpack简介
>**webpack** 是一个开源的前端模块构建工具，由德国开发者 *Tobias Koppers* 在2012年首发并开源。  

由于近年来前端技术发展迅猛，越来越重视用户的操作体验，现在早已经不是那个 html、css、jQuery 一把梭的年代了。  
越来越多的新思想、新框架孕育而生。这导致前端越来越重，需要 **引入依赖**、**高级语法降级解析**，**Css预编译**，**使用更为严格的**`TypeScript`**避免问题**， **使用现代 mvvm 框架**`Vue`，`React`**快捷开发**。最后 **构建** 成浏览器 **可识别** 的`.css` `.js`等基本文件再部署到生产环境，这其中构建一步就是`webpack`做的事情。 

![WechatIMG2925.png](./img/how-to-use-webpack/EOzt1WTKBaXkgIr.png)

## webpack基础使用
> 全局已安装 webpack@4.44.2 和 nodejs@10.13.0  

主要是两个配置：

- **entry**：构建入口, `webpack`会从指定文件开始识别依赖模块关系并开始构建。`webpack4`中默认入口在`/src/index.js`。
- **output**：构建出口，告诉`webpack`，构建完的文件放在哪里，叫什么名字，`webpack4`中默认出口在`/dist/main.js`。

先来演示一下webpack最基本的操作。
新建文件夹，初始化项目`npm init` ，一路回车，生成`package.json`，在这个目录新建两个文件，`main.js`，`chunk.js` ，内容如下：  

main.js   

```shell
const chunk = require('./chunk.js');
console.log(chunk);
```  
chunk.js

```shell
module.exports = 'I am chunk.js';
```  
创建好之后在项目下安装webpack：

```shell
npm install --save-dev webpack webpack-cli
```  
执行第一个命令，`webpack 构建入口 --output 构建出口`： 

```shell
$ webpack ./main.js --output ./dist.js
...
...
Hash: 8cc82b2736e97001d58e
Version: webpack 4.44.2
Time: 269ms
Built at: 2020-09-24 16:46:34
  Asset        Size  Chunks             Chunk Names
dist.js  1000 bytes       0  [emitted]  main
Entrypoint main = dist.js
[0] ./main.js 56 bytes {0} [built]
[1] ./chunk.js 32 bytes {0} [built]
```   
上命令是把`main.js`使用`webpack`打包生成`dist.js`，由于`main.js`中引入`chunk.js`，所以`dist.js`是`main.js`和`chunk.js`的整合。  

可以看到`webpack`输出一些打包信息后，并成功在`./`下创建`dist.js`，看看效果，创建一个`index.html`，并引入`dist.js`

当前目录结构:  

```shell
.
├── chunk.js 
├── index.html
├── main.js 
├── package.json 
```  

打开`index.html`控制台面板可以看到向屏幕输出了「 I am chunk.js 」，看到这里，其实就可以理解`webpack`的作用了。都是在这个基础上，**变形转换**，**以及打包过程中增加一些处理**。这次简单演示使用终端打包，实际开发需要设置相当多的参数，命令不方便处理，会创建一个`webpack.conf.js`，专门来编写配置。  
## webpack4的改变  

 - 速度相比3更快了(提升至少50%，没测过)
 - 默认配置出入口(就是网上说的0配置)
 - 增加了环境配置
  
#### 默认配置出入口
> 这里需要说明一下，在webpack4中实现了默认配置，去除了老版本繁琐的配置，webpack真正的本领是构建，webpack官方也注意到了这一点，这是一个非常明智的改变。  

版本3中需要用`entry`和`output`来指定构建入口和出口，在版本4中有默认配置。  

 - 默认入口在`/src/index.js`
 - 默认出口在`/dist/main.js`

如果你想修改，可以在`package.json`修改script，例：  

```shell
webpack ./entry/js/index.js --output ./output/js/main.js
```  

`webpack.conf.js`：  

```javascript 
const path = require('path')

// 引入path为了实现相对路径转换成绝对路径：
path.join(__dirname, '路径')

module.exports = {
  // entry为开始构建的文件入口
  entry: path.join(__dirname, '/src/index.js'),
  // output为构建后的文件出口，其中path为出口文件的路径，filename为出口文件名
  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'bundle.js'
  }
}
```  
#### 增加了环境配置
4以前版本配置不同环境，根据`nodejs`环境变量需要编写两套配置：开发环境中配置`live reloading` 和`webpack-dev-server`；生产环境配置`sourceMap`和`uglifyJSPlugin`美化压缩代码。在4版本中只需要在`package.json`的中`script` 增加`--mode`参数指定环境，并自动处理上面不同环境区别：

package.json：  

```json
"scripts": {
  "dev": "webpack --mode development",
  "build": "webpack --mode production "
}
```
## webpack进阶使用
主要是两个配置：

- **loader**：`webpack`本身只能处理`.js`文件，安装不同的`loader`使其具备处理其他文件的能力，比如`.css`，`.scss`，`.vue`，`.jsx`，`.jade`。

- **plugins**：用于处理打包过程中一些更复杂的处理，比如`htmlwebpackplugin`自动生成`index.html`并引入构建文件，`uglifyJSPlugin`美化压缩代码。  

#### Loader  
在`/css/index.css`中创建`css`文件，并在`/src/index.js`引入，`css`经过`webpack`构建后会自动作用在`index.html`上。

`/src/index.css `： 

```css
h1{
  color:red;
}
```    
`/src/index.js`：

```javascript
import indexCss from './index.css'; // 或者 import './index.css';

const chunk = require('../chunk.js');
console.log(chunk);
```    
执行`webpack`，可以看到报错了：

```shell  
ERROR in ./src/index.css 1:4
Module parse failed: Unexpected token (1:4)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
> html{
|   color:red;
| }
 @ ./src/index.js 1:0-34
```
你需要安装对应`loader`去处理这个`css`文件，安装`style-loader`和`css-loader`： 

 - **css-loader**：`webpack`默认只能处理`.js`和`.json`文件类型，这个包会使`webpack`具备处理`.css`的能力  
 - **style-loader**：`style-loader`会把`css-loader`构建好的`css`作为`<style>`标签插入到`html`中。

安装他们：  

```shell
npm install style-loader css-loader --save-dev
```  

配置他们：  
> 需要注意：loader 的执行顺序是从后往前执行，本例先执行 css-loader 返回内容再交给 style-loader 执行。链式调用，一环套一环，每个 loader 负责单一指责，方便扩展。

```javascript
module.exports = {
  module: {
    rules: [{ 
      test: /\.css$/, 
      use: [
        { 
          loader: "style-loader",
          options: {
            // options
          }
        },
        { 
          loader: "css-loader" ,
          options: {
            // options
          }
        }
      ]
    }]
  }
}
```  
 - **test**：匹配 loaders 所处理文件的拓展名的正则表达式（必填）
 - **loader**：loader 的名称（必填）
 - **include/exclude**：需要处理的文件夹或屏蔽不需要处理的文件夹（选填）；
 - **options**：为 loaders 提供额外的设置选项（选填）

详细options：  
[style-loader](https://developer.aliyun.com/mirror/npm/package/style-loader)  
[css-loader](https://developer.aliyun.com/mirror/npm/package/css-loader)

重新构建：  

```shell
➜  webpack-demo webpack
Hash: c741b4496ab2094ee570
Version: webpack 4.44.2
Time: 270ms
Built at: 2020-09-30 17:22:42
  Asset     Size  Chunks             Chunk Names
main.js  4.7 KiB       0  [emitted]  main
Entrypoint main = main.js
[1] ./src/index.js 79 bytes {0} [built]
[2] ./src/index.css 529 bytes {0} [built]
[4] ./node_modules/css-loader/dist/cjs.js??ref--4-1!./src/index.css 301 bytes {0} [built]
[5] ./chunk.js 32 bytes {0} [built]
    + 2 hidden modules
```
构建成功了，  
修改`index.html`引入构建后的文件，并打开，可以看到一个红色标题，loader配置生效了  

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>标题</h1>
  <script src='./dist/main.js'></script>
</body>
</html>
```
### Plugin
使用`html-webpack-plugin`和`clean-webpack-plugin`举例：  

 - **html-webpack-plugin**：构建时，在output目录下创建一个index.html文件，并把构建后的文件插入到html中。
 - **clean-webpack-plugin**：每次打包前自动清空output文件夹。  

#### html-webpack-plugin
安装：  

```shell
npm install html-webpack-plugin --save-dev
```
`webpack.config.js`配置：  

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  module: {
    //...
  },
  plugins: [
    new HtmlWebpackPlugin()
  ]
}
```
[html-webpack-plugin更多参数](https://developer.aliyun.com/mirror/npm/package/html-webpack-plugin)

#### clean-webpack-plugin
安装： 

```shell
npm install clean-webpack-plugin --save-dev 
```  

`webpack.config.js`配置：   

```javascript
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  module: {
    //...
  },
  plugins: [
    new CleanWebpackPlugin(['dist'])
  ]
}
```
[clean-webpack-plugin更多参数](https://developer.aliyun.com/mirror/npm/package/clean-webpack-plugin) 

复杂的`plugins`配置：  

```javascript
plugins: [  
	 new HtmlWebpackPlugin(),
     new CleanWebpackPlugin(['dist'])
]
```
`plugins`为一个数组，每一项都是一个插件，插件的配置都是作为参数传入，有数组也有对象类型，`plugin`不像`loader`有很明确的顺序，它类似事件绑定，按需执行。