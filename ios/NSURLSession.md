# 简介

NSURLSession是NSURLConnection的替代者，而现在使用最广泛的第三方框架：AFNetworking, SDWebImage等等都使用了NSURLSession为底层。Session指的是会话，因此我们可以将7层网络协议中的物理层->数据链路层->网络层->传输层->会话层->表示层->应用层中的会话层当作NSURLSession工作的地方

NSURLSession将NSURLConnection替代为了NSURLSession和NSURLSessionConfiguration，以及3个NSURLSessionTask的子类：NSURLSessionDataTask, NSURLSessionUploadTask, NSURLSessionDownloadTask

<img src="../assets/1" alt="Snip20170211_1" style="zoom:50%;" />

NSURLSessionTask以及三个子类的继承关系：

<img src="../assets/1-20201219195513452" alt="7f6418d2-de8b-3006-a57f-95b3bd6c96cf" style="zoom:100%;" />

*NSURLSessionDataTask*: 主要用于读取服务端的简单数据，比如 JSON 数据。

*NSURLSessionDownloadTask*: 这个 task 的主要用途是进行文件下载，它针对大文件的网络请求做了更多的处理，比如下载进度，断点续传等等。

*NSURLSessionUploadTask*: 和下载任务对应，这个 task 主要是用于对服务端发送文件类型的数据使用的。

# NSURLSession的使用

NSURLSession 本身是不会进行请求的，而是通过创建 task 的形式进行网络请求（resume() 方法的调用），同一个 NSURLSession 可以创建多个 task，并且这些 task 之间的 cache 和 cookie 是共享的。NSURLSession的使用有如下几步：

- 第一步：创建NSURLSession对象
- 第二步：使用NSURLSession对象创建Task
- 第三步：启动任务

## 第一步：创建NSURLSession对象

创建NSURLSession对象的方法：（1）直接创建（2）配置后创建（3）设置加代理获得

关于NSURLSession的配置有三种：（1）默认，将缓存存储在磁盘上（2）短暂的会话，不会创建持久性存储的缓存（3）后台会话模式允许程序在后台进行上传和下载

## 第二步：使用NSURLSession对象创建Task

使用NSURLSession对象创建Task（NSURLSessionTask以及子类的创建需要根据具体类型来创建）：（1）**NSURLSessionDataTask**的创建：通过NSURLRequest或者NSURL对象创建（可指定回调代码块，也可以不指定）（2）**NSURLSessionUploadTask**的创建：通过NSURLRequest创建，并且也要指定文件源或者数据源（可指定回调闭包）（3）**NSURLSessionDownloadTask**的创建：通过NSURLRequest和NSURL来创建，如果想要使用下载任务断点续传功能，那么也可以通过NSData（resumeData，这个data肯定不能是随便的NSData，应该是有数据格式的，所以才能支持断点续传的功能）去创建（可指定回调闭包）

我们在使用三种Task的任意一种的时候都可以指定相应的代理。NSURLSession的代理对象结构如下：

![a01337ad-ac97-3b9a-8ee7-f5b261747202](../assets/1-20201219200723620)

*NSURLSessionDelegate* – 作为所有代理的基类，定义了网络请求最基础的代理方法。

*NSURLSessionTaskDelegate* – 定义了网络请求任务相关的代理方法。

*NSURLSessionDownloadDelegate* – 用于下载任务相关的代理方法，比如下载进度等等。

*NSURLSessionDataDelegate* – 用于普通数据任务和上传任务。

## 第三步：启动任务

[task resume];

# GET/POST

