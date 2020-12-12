![AFNetworking](../assets/68747470733a2f2f7261772e6769746875622e636f6d2f41464e6574776f726b696e672f41464e6574776f726b696e672f6173736574732f61666e6574776f726b696e672d6c6f676f2e706e67-20201213020615453.png)

# 简介

AFNetworking（下面简称“afn”）是我们非常熟悉的一个开源组件，该文章主要就是通过讲解源代码的方式去讲解afn是怎么工作的，这对刚接触iOS开发的同学来说还是蛮重要的。

并且我们会根据文件去挨个讲解具体的源代码内容

# AFURLSessionManager.h/.m

## AFURLSessionManager.h

### 简介

AFURLSessionManager创建并且管理着NSURLSession对象（该对象基于NSURLSessionConfiguration去创建的），并且AFURLSessionManager满足协议`<NSURLSessionTaskDelegate>`, `<NSURLSessionDataDelegate>`, `<NSURLSessionDownloadDelegate>`, and `<NSURLSessionDelegate>`.

AFURLSessionManager是AFHTTPSessionManager的父类，而AFHTTPSessionManager就是针对http请求进行针对性地添加功能，也就是说，假如你想要专门为了http去扩展AFURLSessionManager，那么你可以考虑直接使用AFHTTPSessionManager，因为已经写好了！

### 实现的代理方法

AFURLSessionManager实现了下列的代理方法：

\### `NSURLSessionDelegate`

 \- `URLSession:didBecomeInvalidWithError:`

 \- `URLSession:didReceiveChallenge:completionHandler:`

 \- `URLSessionDidFinishEventsForBackgroundURLSession:`



 \### `NSURLSessionTaskDelegate`

 \- `URLSession:willPerformHTTPRedirection:newRequest:completionHandler:`

 \- `URLSession:task:didReceiveChallenge:completionHandler:`

 \- `URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`

 \- `URLSession:task:needNewBodyStream:`

 \- `URLSession:task:didCompleteWithError:`



 \### `NSURLSessionDataDelegate`

 \- `URLSession:dataTask:didReceiveResponse:completionHandler:`

 \- `URLSession:dataTask:didBecomeDownloadTask:`

 \- `URLSession:dataTask:didReceiveData:`

 \- `URLSession:dataTask:willCacheResponse:completionHandler:`



 \### `NSURLSessionDownloadDelegate`

 \- `URLSession:downloadTask:didFinishDownloadingToURL:`

 \- `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`

 \- `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`

### 暴露的属性

#### 核心

**@property** (**readonly**, **nonatomic**, **strong**) NSURLSession *session;	// 受到管理的session

**@property** (**readonly**, **nonatomic**, **strong**) NSOperationQueue *operationQueue;	// delegate callbacks are running on this queue

**@property** (**nonatomic**, **strong**) **id** <AFURLResponseSerialization> responseSerializer;	// 不能为nil，data task中从server发过来的response会用`dataTaskWithRequest:success:failure:`创建并run，通过使用`GET`/`POST`等便利方法自动地解析，默认的，这个属性会设置为一个`AFJSONResponseSerializer`的实例。这部分翻译得让我晕头转向的，不过我想这个应该就是一个解析器，专门用来解析url response中的消息内容的，而为什么不是http response而是取名为http response，后面有待考究，这可能跟苹果的URLSession的原理有关，所以在这里就先不去考虑了。

#### 管理Security Policy

**@property** (**nonatomic**, **strong**) AFSecurityPolicy *securityPolicy;	// 创建session的时候会评估server是否可信，用这个securityPolicy，如果不指定，AFURLSessionManager就会使用defaultPolicy

#### 监控网络状态

**@property** (**readwrite**, **nonatomic**, **strong**) AFNetworkReachabilityManager *reachabilityManager;	// 如果不是watch_os系统的话就会有这个属性（通过#if !TARGET_OS_WATCH和#endif来设置），就是个reachability manager，默认的话AFURLSessionManager会使用sharedManager

#### 获取Session Tasks

**@property** (**readonly**, **nonatomic**, **strong**) NSArray <NSURLSessionTask *> *tasks;	//  The data, upload, and download tasks currently run by the managed session.

**@property** (**readonly**, **nonatomic**, **strong**) NSArray <NSURLSessionDataTask *> *dataTasks;

**@property** (**readonly**, **nonatomic**, **strong**) NSArray <NSURLSessionUploadTask *> *uploadTasks;

**@property** (**readonly**, **nonatomic**, **strong**) NSArray <NSURLSessionDownloadTask *> *downloadTasks;

#### 管理callback队列

**@property** (**nonatomic**, **strong**, **nullable**) dispatch_queue_t completionQueue;	// the dispatch queue for `completionBlock`, if nil(default)，那就是使用主队列

**@property** (**nonatomic**, **strong**, **nullable**) dispatch_group_t completionGroup;	// the dispatch group for `completionBlock`, if nil(default)，就会使用一个private dispatch group

### 初始化方法

\- (**instancetype**)initWithSessionConfiguration:(**nullable** NSURLSessionConfiguration *)configuration NS_DESIGNATED_INITIALIZER;	//  Creates and returns a manager for a session created with the specified configuration. This is the designated initializer.



\- (**void**)invalidateSessionCancelingTasks:(**BOOL**)cancelPendingTasks resetSession:(**BOOL**)resetSession;	//  Invalidates the managed session, optionally canceling pending tasks and optionally resets given session.大概就是使得session失效，然后可以指定是否使得正在pending的task要不要cancel掉。

### Running Data Task

\- (NSURLSessionDataTask *)dataTaskWithRequest:(NSURLRequest *)request

​                uploadProgress:(**nullable** **void** (^)(NSProgress *uploadProgress))uploadProgressBlock

​               downloadProgress:(**nullable** **void** (^)(NSProgress *downloadProgress))downloadProgressBlock

​              completionHandler:(**nullable** **void** (^)(NSURLResponse *response, **id** **_Nullable** responseObject, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionDataTask` with the specified request. 

//  **@param** request: The HTTP request for the request.

//  **@param** uploadProgressBlock: A block object to be executed when the upload progress is updated. Note this block is called on the session queue, not the main queue.

//  **@param** downloadProgressBlock: A block object to be executed when the download progress is updated. Note this block is called on the session queue, not the main queue.

//  **@param** completionHandler: A block object to be executed when the task finishes. This block has no return value and takes three arguments: the server response, the response object created by that serializer, and the error that occurred, if any.

### Running Upload Task

\- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request

​                     fromFile:(NSURL *)fileURL

​                     progress:(**nullable** **void** (^)(NSProgress *uploadProgress))uploadProgressBlock

​                completionHandler:(**nullable** **void** (^)(NSURLResponse *response, **id** **_Nullable** responseObject, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionUploadTask` with the specified request for a local file.



\- (NSURLSessionUploadTask *)uploadTaskWithRequest:(NSURLRequest *)request

​                     fromData:(**nullable** NSData *)bodyData

​                     progress:(**nullable** **void** (^)(NSProgress *uploadProgress))uploadProgressBlock

​                completionHandler:(**nullable** **void** (^)(NSURLResponse *response, **id** **_Nullable** responseObject, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionUploadTask` with the specified request for an HTTP body.



\- (NSURLSessionUploadTask *)uploadTaskWithStreamedRequest:(NSURLRequest *)request

​                         progress:(**nullable** **void** (^)(NSProgress *uploadProgress))uploadProgressBlock

​                    completionHandler:(**nullable** **void** (^)(NSURLResponse *response, **id** **_Nullable** responseObject, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionUploadTask` with the specified streaming request.

### Running Download Task

\- (NSURLSessionDownloadTask *)downloadTaskWithRequest:(NSURLRequest *)request

​                       progress:(**nullable** **void** (^)(NSProgress *downloadProgress))downloadProgressBlock

​                     destination:(**nullable** NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination

​                  completionHandler:(**nullable** **void** (^)(NSURLResponse *response, NSURL * **_Nullable** filePath, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionDownloadTask` with the specified request.



\- (NSURLSessionDownloadTask *)downloadTaskWithResumeData:(NSData *)resumeData

​                        progress:(**nullable** **void** (^)(NSProgress *downloadProgress))downloadProgressBlock

​                       destination:(**nullable** NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination

​                    completionHandler:(**nullable** **void** (^)(NSURLResponse *response, NSURL * **_Nullable** filePath, NSError * **_Nullable** error))completionHandler;

//  Creates an `NSURLSessionDownloadTask` with the specified resume data.

### Getting Progress for Tasks

\- (**nullable** NSProgress *)uploadProgressForTask:(NSURLSessionTask *)task;	// return the upload progress of the specified task

\- (**nullable** NSProgress *)downloadProgressForTask:(NSURLSessionTask *)task;	//  Returns the download progress of the specified task.

### Setting Session Delegate Callbacks

\- (**void**)setSessionDidBecomeInvalidBlock:(**nullable** **void** (^)(NSURLSession *session, NSError *error))block;	//  Sets a block to be executed when the managed session becomes invalid, as handled by the `NSURLSessionDelegate` method `URLSession:didBecomeInvalidWithError:`.

\- (**void**)setSessionDidReceiveAuthenticationChallengeBlock:(**nullable** NSURLSessionAuthChallengeDisposition (^)(NSURLSession *session, NSURLAuthenticationChallenge *challenge, NSURLCredential * **_Nullable** **__autoreleasing** * **_Nullable** credential))block;	//  Sets a block to be executed when a connection level authentication challenge has occurred, as handled by the `NSURLSessionDelegate` method `URLSession:didReceiveChallenge:completionHandler:`.

\- (**void**)setTaskNeedNewBodyStreamBlock:(**nullable** NSInputStream * (^)(NSURLSession *session, NSURLSessionTask *task))block;	//  Sets a block to be executed when a task requires a new request body stream to send to the remote server, as handled by the `NSURLSessionTaskDelegate` method `URLSession:task:needNewBodyStream:`.

\- (**void**)setTaskWillPerformHTTPRedirectionBlock:(**nullable** NSURLRequest * **_Nullable** (^)(NSURLSession *session, NSURLSessionTask *task, NSURLResponse *response, NSURLRequest *request))block;	//  Sets a block to be executed when an HTTP request is attempting to perform a redirection to a different URL, as handled by the `NSURLSessionTaskDelegate` method `URLSession:willPerformHTTPRedirection:newRequest:completionHandler:`.

\- (**void**)setAuthenticationChallengeHandler:(**id** (^)(NSURLSession *session, NSURLSessionTask *task, NSURLAuthenticationChallenge *challenge, **void** (^completionHandler)(NSURLSessionAuthChallengeDisposition , NSURLCredential * **_Nullable**)))authenticationChallengeHandler;	//  Sets a block to be executed when a session task has received a request specific authentication challenge, as handled by the `NSURLSessionTaskDelegate` method `URLSession:task:didReceiveChallenge:completionHandler:`.

\- (**void**)setTaskDidSendBodyDataBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionTask *task, int64_t bytesSent, int64_t totalBytesSent, int64_t totalBytesExpectedToSend))block;	//  Sets a block to be executed periodically to track upload progress, as handled by the `NSURLSessionTaskDelegate` method `URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`.

\- (**void**)setTaskDidCompleteBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionTask *task, NSError * **_Nullable** error))block;	//  Sets a block to be executed as the last message related to a specific task, as handled by the `NSURLSessionTaskDelegate` method `URLSession:task:didCompleteWithError:`.



\#if AF_CAN_INCLUDE_SESSION_TASK_METRICS

\- (**void**)setTaskDidFinishCollectingMetricsBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionTask *task, NSURLSessionTaskMetrics * **_Nullable** metrics))block AF_API_AVAILABLE(ios(10), macosx(10.12), watchos(3), tvos(10));

\#endif

//  Sets a block to be executed when metrics are finalized related to a specific task, as handled by the `NSURLSessionTaskDelegate` method `URLSession:task:didFinishCollectingMetrics:`.



\- (**void**)setDataTaskDidReceiveResponseBlock:(**nullable** NSURLSessionResponseDisposition (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSURLResponse *response))block;	//  Sets a block to be executed when a data task has received a response, as handled by the `NSURLSessionDataDelegate` method `URLSession:dataTask:didReceiveResponse:completionHandler:`.

\- (**void**)setDataTaskDidBecomeDownloadTaskBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSURLSessionDownloadTask *downloadTask))block;	//  Sets a block to be executed when a data task has become a download task, as handled by the `NSURLSessionDataDelegate` method `URLSession:dataTask:didBecomeDownloadTask:`.

\- (**void**)setDataTaskDidReceiveDataBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSData *data))block;	//  Sets a block to be executed when a data task receives data, as handled by the `NSURLSessionDataDelegate` method `URLSession:dataTask:didReceiveData:`.

\- (**void**)setDataTaskWillCacheResponseBlock:(**nullable** NSCachedURLResponse * (^)(NSURLSession *session, NSURLSessionDataTask *dataTask, NSCachedURLResponse *proposedResponse))block;	//  Sets a block to be executed to determine the caching behavior of a data task, as handled by the `NSURLSessionDataDelegate` method `URLSession:dataTask:willCacheResponse:completionHandler:`.

\- (**void**)setDidFinishEventsForBackgroundURLSessionBlock:(**nullable** **void** (^)(NSURLSession *session))block AF_API_UNAVAILABLE(macos);	//  Sets a block to be executed once all messages enqueued for a session have been delivered, as handled by the `NSURLSessionDataDelegate` method `URLSessionDidFinishEventsForBackgroundURLSession:`.

\- (**void**)setDownloadTaskDidFinishDownloadingBlock:(**nullable** NSURL * **_Nullable** (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, NSURL *location))block;	//  Sets a block to be executed when a download task has completed a download, as handled by the `NSURLSessionDownloadDelegate` method `URLSession:downloadTask:didFinishDownloadingToURL:`.

\- (**void**)setDownloadTaskDidWriteDataBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, int64_t bytesWritten, int64_t totalBytesWritten, int64_t totalBytesExpectedToWrite))block;	//  Sets a block to be executed periodically to track download progress, as handled by the `NSURLSessionDownloadDelegate` method `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`.

\- (**void**)setDownloadTaskDidResumeBlock:(**nullable** **void** (^)(NSURLSession *session, NSURLSessionDownloadTask *downloadTask, int64_t fileOffset, int64_t expectedTotalBytes))block;	//  Sets a block to be executed when a download task has been resumed, as handled by the `NSURLSessionDownloadDelegate` method `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`.

### Notifications

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidResumeNotification;	// posted when a task resumes

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteNotification;	//  Posted when a task finishes executing. Includes a userInfo dictionary with additional information about the task.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidSuspendNotification;	//  Posted when a task suspends its execution.

FOUNDATION_EXPORT NSString * **const** AFURLSessionDidInvalidateNotification;	//  Posted when a session is invalidated.

FOUNDATION_EXPORT NSString * **const** AFURLSessionDownloadTaskDidMoveFileSuccessfullyNotification;	//  Posted when a session download task finished moving the temporary download file to a specified destination successfully.

FOUNDATION_EXPORT NSString * **const** AFURLSessionDownloadTaskDidFailToMoveFileNotification;	//  Posted when a session download task encountered an error when moving the temporary download file to a specified destination.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteResponseDataKey;	//  The raw response data of the task. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteNotification` if response data exists for the task.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteSerializedResponseKey;	//  The serialized response object of the task. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteNotification` if the response was serialized.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteResponseSerializerKey;	//  The response serializer used to serialize the response. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteNotification` if the task has an associated response serializer.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteAssetPathKey;	//  The file path associated with the download task. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteNotification` if an the response data has been stored directly to disk.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteErrorKey;	//  Any error associated with the task, or the serialization of the response. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteNotification` if an error exists.

FOUNDATION_EXPORT NSString * **const** AFNetworkingTaskDidCompleteSessionTaskMetrics;	//  The session task metrics taken from the download task. Included in the userInfo dictionary of the `AFNetworkingTaskDidCompleteSessionTaskMetrics`