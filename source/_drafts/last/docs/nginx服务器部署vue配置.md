## 环境
 - CentOS 7.6 64位
 - nginx@1.19.2
 - vue-cli@4.5.0  
 
## nginx配置
连接服务器，进入要下载的目录，这里用的`/usr/local/src`，你也可以在`/opt`下，这两种算约定俗称的目录结构，便于统一管理。

下载`nginx`，这是使用的是二进制包，你可以通过`yum`包管理安装。   
我这里用的最新的`1.19.2`版本。[更多版本](http://nginx.org/en/download.html)

```shell
wget -c https://nginx.org/download/nginx-1.19.2.tar.gz 
```
解压缩并进入`nginx`文件夹：  

```shell
tar -zxvf nginx-1.19.2.tar.gz
...
cd nginx-1.19.2
```  
编译前环境监测：

```shell
./configure
```  
如果没报错忽略这条。新服务器这里会报错，提示需要安装`g++`，它是c++的编译器，有人做了相关的编译包整合，比一个一个自己装方便，我们这里直接安装整合包，安装完成后重新执行`./configure`。  

```shell
yum groupinstall "Development Tools"
...  
```  
编译并安装：

```shell
make && make install
```    
启动它：  

```shell
cd /usr/local/nginx/sbin
./nginx
```  
更多常用命令：  

```shell
./nginx -s stop // 停止
./nginx -s reload // 重启
```
默认`nginx`在`80端口`运行，记得到服务器购买网站控制台开放一下公网`80端口`权限。   
 
要想修改配置：  

```shell
cd /usr/local/nginx/config
vi ./nginx.config
```  
## 配置vue
克隆你的vue项目，我是在`/opt/vue/`目录下： 

```shell
cd /opt/vue
git clone xxx
npm install
npm run build
```  

找到`http`块中的`server`块。

```shell
server {
        listen       80;  #监听端口
        server_name  127.0.0.1; #监听地址    
        location / {
            root   /opt/vue/dist; # 资源目录，我这里是vue构建后的目录
            index  index.html index.htm; #首页文件设置
            try_files $uri $uri/ /index.html; #vue-router history 路由设置
        }
        # 以下是指定错误时展示的文件文件
        #error_page  404              /404.html; 
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```  
[更多nginx配置参考](https://www.runoob.com/w3cnote/nginx-setup-intro.html)
##### 其中 
 - 设置`localtion`块的`root`属性为你的vue构建后文件    

 - 如果 vue 使用`history`路由，设置`try_files`为`$uri $uri/ /index.html`，用来告诉`nginx`前端路由交给前端的`index`文件去处理了，不用你处理了


