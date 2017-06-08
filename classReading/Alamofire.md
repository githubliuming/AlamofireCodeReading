# Alamofire 源码 
----

## 协议部分
1、  URLConvertible  
*  URL转换协议，实现该该协议的 
```
// 转换成功则返回一个 URL对象， 失败则抛出AFError异常
func asURL() throws -> URL

```


2、 URLRequestConvertible
* URL Requst 转换协议

```
//转换成功 则返回一个 URLRequset 对象 失败则抛出AFError异常
 func asURLRequest() throws -> URLRequest
```

## 扩展 extension部分
1、extension String: URLConvertible
* String 类实现 URL转换协议

```
// 通过 String 初始化 URL对象， 成功则返回URL对象 失败则 抛出 AFError的验证URL失败异常
  public func asURL() throws -> URL {
        guard let url = URL(string: self) else { throw AFError.invalidURL(url: self) }
        return url
    }
```

2、extension URL: URLConvertible 
* URL类型实现 URLConvertible协议
```
//直接返回 self
public func asURL() throws -> URL { return self }
```

3、extension URLComponents: URLConvertible

* URLComponents 实现 URLConvertible
```
    //如果  URLComponents 的 url属性不为空 则直接返回 url属性的值，否则抛出AFError的验证URL失败异常
    public func asURL() throws -> URL {
        guard let url = url else { throw AFError.invalidURL(url: self) }
        return url
    }
```

4、extension URLRequest: URLRequestConvertible

 *  扩展URLRequestConvertible 协议内容

```
扩张属性 urlRequest 并实现 get方法。使用该属性等同调用  asURLRequest()
public var urlRequest: URLRequest? { return try? asURLRequest() }
```

5、extension URLRequest

URLRequest方法扩展 
```
 //扩展初始化方法。
    public init(url: URLConvertible, method: HTTPMethod, headers: HTTPHeaders? = nil) throws {
        let url = try url.asURL()

        self.init(url: url)

        httpMethod = method.rawValue

        if let headers = headers {
            for (headerField, headerValue) in headers {
                setValue(headerValue, forHTTPHeaderField: headerField)
            }
        }
    }

    //扩展适配器方法，通过一个实现 RequestAdapter协议的对象生成 一个 URLRequest对象
    func adapt(using adapter: RequestAdapter?) throws -> URLRequest {
        guard let adapter = adapter else { return self }
        return try adapter.adapt(self)
    }
}
```

## 方法部分

```

//通过 url method parameters encoding header 创建 DataRequest对象

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
*  _ url: URLConvertible 实现URLConvertible协议的对象 (可以是 String或者 URL)
*  HTTPMethod = .get HTTPMethod 提交方法 默认为GET
*  parameters: Parameters? = nil, 请求参数。 默认 没有参数
*  headers: HTTPHeaders? = nil HTTP请求头 默认不设置


```
// 通过 实现  URLRequestConvertible协议对象创建 DataRequest对象
public func request(_ urlRequest: URLRequestConvertible) -> DataRequest {
    return SessionManager.default.request(urlRequest)
}
```

* _ urlRequest: URLRequestConvertible 实现URLRequestConvertible的对象


``` 
创建 DownloadRequest 
public func download(
    _ urlRequest: URLRequestConvertible,
    to destination: DownloadRequest.DownloadFileDestination? = nil)
    -> DownloadRequest
{
    return SessionManager.default.download(urlRequest, to: destination)
}
```
* 实现 URLRequestConvertible协议对象 
* destination (_ temporaryURL: URL,_ response: HTTPURLResponse)-> (destinationURL: URL, options: DownloadOptions) 类型代码块 默认为nil


```
//创建上传 UploadRequest
public func upload(_ fileURL: URL, with urlRequest: URLRequestConvertible) -> UploadRequest {
    return SessionManager.default.upload(fileURL, with: urlRequest)
}
```
*  fileURL 文件url 
*  实现 URLRequestConvertible 协议的对象 

```
//创建上传 UploadRequest
public func upload(
    _ data: Data,
    to url: URLConvertible,
    method: HTTPMethod = .post,
    headers: HTTPHeaders? = nil)
    -> UploadRequest
{
    return SessionManager.default.upload(data, to: url, method: method, headers: headers)
}
```

* data 上传文件data
* url  实现URLConvertible协议的服务器地址对象
* method 提交数据方式 默认 POST
* headers HTTTP请求头 默认nil

```
//创建上传 UploadRequest
public func upload(_ data: Data, with urlRequest: URLRequestConvertible) -> UploadRequest {
    return SessionManager.default.upload(data, with: urlRequest)
}
```
* data 上传的数据
* urlRequest  实现URLRequestConvertible协议的 request对象


```
//创建上传 UploadRequest

public func upload(
    _ stream: InputStream,
    to url: URLConvertible,
    method: HTTPMethod = .post,
    headers: HTTPHeaders? = nil)
    -> UploadRequest
{
    return SessionManager.default.upload(stream, to: url, method: method, headers: headers)
}
```
* stream  上传数据流
* url 实现 URLConvertible url对象
* method 提交数据方式 默认POST
* HTTPHeaders HTTP请求头 默认nil

```
//创建上传 UploadRequest
public func upload(_ stream: InputStream, with urlRequest: URLRequestConvertible) -> UploadRequest {
    return SessionManager.default.upload(stream, with: urlRequest)
}
```

* stream 上传数据流
* urlRequest 实现URLRequestConvertible request对象


```
public func upload(
    multipartFormData: @escaping (MultipartFormData) -> Void,
    usingThreshold encodingMemoryThreshold: UInt64 = SessionManager.multipartFormDataEncodingMemoryThreshold,
    to url: URLConvertible,
    method: HTTPMethod = .post,
    headers: HTTPHeaders? = nil,
    encodingCompletion: ((SessionManager.MultipartFormDataEncodingResult) -> Void)?)
{
    return SessionManager.default.upload(
        multipartFormData: multipartFormData,
        usingThreshold: encodingMemoryThreshold,
        to: url,
        method: method,
        headers: headers,
        encodingCompletion: encodingCompletion
    )
}
```
* multipartFormData  (MultipartFormData) -> Void 类型的 代码块。拼接上传数据的代码块
* usingThreshold  编码内存阈值，以字节为单位  如果multipartFormData的长度少于usingThreshold值则在内存中进行编码，否则将数据流存储到磁盘
* url  上传服务器地址 实现 URLConvertible协议
* method 提交方式 默认POST
* headers HTTP请求头 默认nil
* encodingCompletion  multipartFormData 数据编码完成的回掉


```
public func upload(
    multipartFormData: @escaping (MultipartFormData) -> Void,
    usingThreshold encodingMemoryThreshold: UInt64 = SessionManager.multipartFormDataEncodingMemoryThreshold,
    with urlRequest: URLRequestConvertible,
    encodingCompletion: ((SessionManager.MultipartFormDataEncodingResult) -> Void)?)
{
    return SessionManager.default.upload(
        multipartFormData: multipartFormData,
        usingThreshold: encodingMemoryThreshold,
        with: urlRequest,
        encodingCompletion: encodingCompletion
    )
}
```
* multipartFormData (MultipartFormData) -> Void 类型的 代码块。拼接上传数据的代码块
* usingThreshold  编码内存阈值，以字节为单位  如果multipartFormData的长度少于usingThreshold值则在内存中进行编码，否则将数据流存储到磁盘
* urlRequest  上传 request对象 实现URLRequestConvertible协议
* encodingCompletion multipartFormData编码完成回掉



