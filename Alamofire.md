
![](https://raw.githubusercontent.com/Alamofire/Alamofire/assets/alamofire.png)

曾经的AFNetworking在iOS开发中几乎成了标配，到处都能看到它的身影。现如今它的 Swift版的异性兄弟 - Alamofire 大有赶超大哥的趋势，自从Swift3.0发布，它也逐步走向成熟。所以是时候对它好好熟悉一下了。今天就来揭开她的面纱，看一看她是怎么样的本质。

##### 环境：Xcode8.3.1, 通过pod导入 Alamofire 4.4 
##### 目标：主要来分析下Alamofire 创建一个任务发起请求的一系列的背后过程和操作。

### 一个简单的使用
```
Alamofire
.request("https://httpbin.org/get")
.responseJSON 
{ response in
    print("Request: \(String(describing: response.request))")   // original url request
    print("Response: \(String(describing: response.response))") // http url response
    print("Result: \(response.result)")                         // response serialization result

    if let json = response.result.value {
        print("JSON: \(json)") // serialized json response
    }

    if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
        print("Data: \(utf8Text)") // original server data as UTF8 string
    }
}

```
以上是一个简单的`GET`请求，然后使用JSON格式返回结果的序列化。下面依次来看其中的操作过程：

### 首先调用`request方法`

```
public func request(
    _ url: URLConvertible,
    method: HTTPMethod = .get,
    parameters: Parameters? = nil,
    encoding: ParameterEncoding = URLEncoding.default,
    headers: HTTPHeaders? = nil)
    -> DataRequest
{
    return SessionManager.default.request(
        url,
        method: method,
        parameters: parameters,
        encoding: encoding,
        headers: headers
    )
}

```
然后在`SessionManager.swift`中进行创建任务，发起请求等主要的操作处理。

```
open static let `default`: SessionManager = {
        let configuration = URLSessionConfiguration.default
        configuration.httpAdditionalHeaders = SessionManager.defaultHTTPHeaders

        return SessionManager(configuration: configuration)
    }()
 
 	//SessionManager 初始化操作
    public init(
        configuration: URLSessionConfiguration = URLSessionConfiguration.default,
        delegate: SessionDelegate = SessionDelegate(),
        serverTrustPolicyManager: ServerTrustPolicyManager? = nil)
    {
        self.delegate = delegate
        self.session = URLSession(configuration: configuration, delegate: delegate, delegateQueue: nil)

        commonInit(serverTrustPolicyManager: serverTrustPolicyManager)
    }    
    
```
* 创建`SessionManager`的单例对象，通过`static let`作为修饰单例对象，`'default'`之所以用引号包起来，由于default是系统保留字。在swift中可以使用保留字作为标识符，但需要在其前后增加反引号。
* 创建`SessionManager`的单例对象的`delegate `,即`SessionDelegate`。此delegate作为session的代理，实现了以下协议中的方法：

| 	协议名称  		           |
|----------------------		|
| URLSessionDelegate  		|
| URLSessionTaskDelegate  	|
| URLSessionDataDelegate  	|
| URLSessionDownloadDelegate|
| URLSessionStreamDelegate  |

所以当开始请求后，就开始调用`SessionDelegate `中代理方法。

###根据已序列化好的`urlRequest`真正的发起任务请求
```
    open func request(_ urlRequest: URLRequestConvertible) -> DataRequest {
        var originalRequest: URLRequest?

        do {
            originalRequest = try urlRequest.asURLRequest()
            let originalTask = DataRequest.Requestable(urlRequest: originalRequest!)

            let task = try originalTask.task(session: session, adapter: adapter, queue: queue)
            let request = DataRequest(session: session, requestTask: .data(originalTask, task))

            delegate[task] = request

            if startRequestsImmediately { request.resume() }

            return request
        } catch {
            return request(originalRequest, failedWith: error)
        }
    }
    
```
* 创建任务`task`
* 将创建好的`request`以 task 为键存入 delegate 的下标中，其实就是放入一个字典类型的变量中，便于稍后取出使用。
* 创建DataRequest类型的请求实体 `request`，创建请求的任务代理 `taskDelegate`，此代理才是真正该请求的执行部分，如何执行稍后叙述。

创建taskDelegate操作，在此除了生成taskDelegate还创建了taskDelegate 的队列，初始化如下：

```
    init(session: URLSession, requestTask: RequestTask, error: Error? = nil) {
        self.session = session

        switch requestTask {
        case .data(let originalTask, let task):
            taskDelegate = DataTaskDelegate(task: task)
            self.originalTask = originalTask
        case .download(let originalTask, let task):
            taskDelegate = DownloadTaskDelegate(task: task)
            self.originalTask = originalTask
        case .upload(let originalTask, let task):
            taskDelegate = UploadTaskDelegate(task: task)
            self.originalTask = originalTask
        case .stream(let originalTask, let task):
            taskDelegate = TaskDelegate(task: task)
            self.originalTask = originalTask
        }

        delegate.error = error
        delegate.queue.addOperation { self.endTime = CFAbsoluteTimeGetCurrent() }
    }
    
    //taskDelegate 队列的初始化，可看出队列每次执行一个操作，默认挂起。
    所有加入队列的操作都是按照添加到队列的顺序依次执行。
    init(task: URLSessionTask?) {
        self.task = task

        self.queue = {
            let operationQueue = OperationQueue()

            operationQueue.maxConcurrentOperationCount = 1
            operationQueue.isSuspended = true
            operationQueue.qualityOfService = .utility

            return operationQueue
        }()
    }
    
```

###请求成功返回
调用`SessionDelegate`中代理方法，部分代码如下

```
open func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?)
{

   let completeTask: (URLSession, URLSessionTask, Error?) -> Void = { [weak self] session, task, error in
            guard let strongSelf = self else { return }

            strongSelf.taskDidComplete?(session, task, error)

            strongSelf[task]?.delegate.urlSession(session, task: task, didCompleteWithError: error)
            strongSelf[task] = nil
        }
		....

        completeTask(session, task, error)

    }
    
```

* ` strongSelf[task]` 从`SessionDelegate`下标中获取任务对应的请求，然后获取`TaskDelegate`来执行最后的处理操作。




































