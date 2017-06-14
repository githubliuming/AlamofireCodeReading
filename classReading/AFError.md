* swift 中到枚举相比 oc中的枚举改变比较大，不熟悉到可以先看后面这篇教程 http://swift.gg/2015/11/20/advanced-practical-enum-examples/

* AFError 实现Error协议， 注意：Error协议是一个空协议

1、扩展枚举部分
-----
```
AFError 内嵌枚举 参数编码发生错误到情况
  public enum ParameterEncodingFailureReason {
        case missingURL
        case jsonEncodingFailed(error: Error)
        case propertyListEncodingFailed(error: Error)
    }
```
* missingURL 无 url
* jsonEncodingFailed json序列化过程中抛出到错误
* propertyListEncodingFailed  属性列表序列化过程中出错

```
扩展内嵌枚举 ParameterEncodingFailureReason 
extension AFError.ParameterEncodingFailureReason {
    var underlyingError: Error? {
        switch self {
        case .jsonEncodingFailed(let error), .propertyListEncodingFailed(let error):
            return error
        default:
            return nil
        }
    }
}
```
*  扩展 underlyingError 属性，实现get方法 当前的ParameterEncodingFailureReason枚举是jsonEncodingFailed 返回 json序列化失败到原因 否则 返回 nil

```
扩展内嵌枚举 ParameterEncodingFailureReason
extension AFError.ParameterEncodingFailureReason {
    var localizedDescription: String {
        switch self {
        case .missingURL:
            return "URL request to encode was missing a URL"
        case .jsonEncodingFailed(let error):
            return "JSON could not be encoded because of error:\n\(error.localizedDescription)"
        case .propertyListEncodingFailed(let error):
            return "PropertyList could not be encoded because of error:\n\(error.localizedDescription)"
        }
    }
}
```
* 扩展 localizedDescription属性，实现get方法。返回ParameterEncodingFailureReason各种错误的 描述字符串

```
AFError 内嵌枚举
    public enum MultipartEncodingFailureReason {
        case bodyPartURLInvalid(url: URL)
        case bodyPartFilenameInvalid(in: URL)
        case bodyPartFileNotReachable(at: URL)
        case bodyPartFileNotReachableWithError(atURL: URL, error: Error)
        case bodyPartFileIsDirectory(at: URL)
        case bodyPartFileSizeNotAvailable(at: URL)
        case bodyPartFileSizeQueryFailedWithError(forURL: URL, error: Error)
        case bodyPartInputStreamCreationFailed(for: URL)

        case outputStreamCreationFailed(for: URL)
        case outputStreamFileAlreadyExists(at: URL)
        case outputStreamURLInvalid(url: URL)
        case outputStreamWriteFailed(error: Error)

        case inputStreamReadFailed(error: Error)
    }
```
* bodyPartURLInvalid 提供到url 不是 文件URL (file Url验证失败)
* bodyPartFilenameInvalid 提供的文件路径中文件名为空(文件名丢失)
* bodyPartFileNotReachable 文件路径不存在
* bodyPartFileNotReachableWithError 文件路径不存在 携带错误信息
* bodyPartFileIsDirectory 文件路径是一个文件目录
* bodyPartFileSizeQueryFailedWithError 获取文件大小时出错
* bodyPartInputStreamCreationFailed 根据文件路径创建InputStream流时出错
* outputStreamCreationFailed  OutputStream创建失败
* outputStreamFileAlreadyExists outpuStream 写入文件时，写入文件已经存在
* outputStreamURLInvalid    outputStream输出路径为非文件路径
* outputStreamWriteFailed   outpuStream 写入文件失败
* inputStreamReadFailed     InputStream读取文件失败


```
MultipartEncodingFailureReason 枚举扩展
extension AFError.MultipartEncodingFailureReason {
    var url: URL? {
        switch self {
        case .bodyPartURLInvalid(let url), .bodyPartFilenameInvalid(let url), .bodyPartFileNotReachable(let url),
             .bodyPartFileIsDirectory(let url), .bodyPartFileSizeNotAvailable(let url),
             .bodyPartInputStreamCreationFailed(let url), .outputStreamCreationFailed(let url),
             .outputStreamFileAlreadyExists(let url), .outputStreamURLInvalid(let url),
             .bodyPartFileNotReachableWithError(let url, _), .bodyPartFileSizeQueryFailedWithError(let url, _):
            return url
        default:
            return nil
        }
    }

    var underlyingError: Error? {
        switch self {
        case .bodyPartFileNotReachableWithError(_, let error), .bodyPartFileSizeQueryFailedWithError(_, let error),
             .outputStreamWriteFailed(let error), .inputStreamReadFailed(let error):
            return error
        default:
            return nil
        }
    }
}
```
*  url 扩展url属性，并实现get方法。返回各种url错误下错误到url路径
*  underlyingError  扩展 underlyingError属性 返回 错误的error信息

```
MultipartEncodingFailureReason 枚举扩展

extension AFError.MultipartEncodingFailureReason {
    var localizedDescription: String {
        switch self {
        case .bodyPartURLInvalid(let url):
            return "The URL provided is not a file URL: \(url)"
        case .bodyPartFilenameInvalid(let url):
            return "The URL provided does not have a valid filename: \(url)"
        case .bodyPartFileNotReachable(let url):
            return "The URL provided is not reachable: \(url)"
        case .bodyPartFileNotReachableWithError(let url, let error):
            return (
                "The system returned an error while checking the provided URL for " +
                "reachability.\nURL: \(url)\nError: \(error)"
            )
        case .bodyPartFileIsDirectory(let url):
            return "The URL provided is a directory: \(url)"
        case .bodyPartFileSizeNotAvailable(let url):
            return "Could not fetch the file size from the provided URL: \(url)"
        case .bodyPartFileSizeQueryFailedWithError(let url, let error):
            return (
                "The system returned an error while attempting to fetch the file size from the " +
                "provided URL.\nURL: \(url)\nError: \(error)"
            )
        case .bodyPartInputStreamCreationFailed(let url):
            return "Failed to create an InputStream for the provided URL: \(url)"
        case .outputStreamCreationFailed(let url):
            return "Failed to create an OutputStream for URL: \(url)"
        case .outputStreamFileAlreadyExists(let url):
            return "A file already exists at the provided URL: \(url)"
        case .outputStreamURLInvalid(let url):
            return "The provided OutputStream URL is invalid: \(url)"
        case .outputStreamWriteFailed(let error):
            return "OutputStream write failed with error: \(error)"
        case .inputStreamReadFailed(let error):
            return "InputStream read failed with error: \(error)"
        }
    }
}
```

* localizedDescription 扩展localizedDescription属性并实现get方法 返回MultipartEncodingFailureReason枚举的错误描述信息


```
AFError 内嵌枚举 ResponseValidationFailureReason ---服务器响应错误
    public enum ResponseValidationFailureReason {
        case dataFileNil
        case dataFileReadFailed(at: URL)
        case missingContentType(acceptableContentTypes: [String])
        case unacceptableContentType(acceptableContentTypes: [String], responseContentType: String)
        case unacceptableStatusCode(code: Int)
    }
```
* dataFileNil 服务器响应到数据文件不存在
* dataFileReadFailed 服务器响应的数据文件不能read
* missingContentType  响应不包含“Content-Type”，提供的“acceptableContentTypes”不包含通配符类型
* unacceptableContentType response中到`Content-Type`在设置到`acceptableContentTypes`匹配不到
* unacceptableStatusCode 服务器返回的status code不正确

```
ResponseValidationFailureReason的扩展
extension AFError.ResponseValidationFailureReason {
    var acceptableContentTypes: [String]? {
        switch self {
        case .missingContentType(let types), .unacceptableContentType(let types, _):
            return types
        default:
            return nil
        }
    }

    var responseContentType: String? {
        switch self {
        case .unacceptableContentType(_, let responseType):
            return responseType
        default:
            return nil
        }
    }

    var responseCode: Int? {
        switch self {
        case .unacceptableStatusCode(let code):
            return code
        default:
            return nil
        }
    }
}

```
* acceptableContentTypes 扩展到属性，返回错误的acceptableContentTypes列表
* responseContentType    扩展属性, 返回未匹配的`Content-Type`
* responseCode           扩展属性  返回服务器错误码

```
extension AFError.ResponseValidationFailureReason {
    var localizedDescription: String {
        switch self {
        case .dataFileNil:
            return "Response could not be validated, data file was nil."
        case .dataFileReadFailed(let url):
            return "Response could not be validated, data file could not be read: \(url)."
        case .missingContentType(let types):
            return (
                "Response Content-Type was missing and acceptable content types " +
                "(\(types.joined(separator: ","))) do not match \"*/*\"."
            )
        case .unacceptableContentType(let acceptableTypes, let responseType):
            return (
                "Response Content-Type \"\(responseType)\" does not match any acceptable types: " +
                "\(acceptableTypes.joined(separator: ","))."
            )
        case .unacceptableStatusCode(let code):
            return "Response status code was unacceptable: \(code)."
        }
    }
}
```
* localizedDescription 扩展属性 返回ResponseValidationFailureReason枚举各项到描述文本

```
AFError内嵌枚举 记录 respose序列化失败原因
    public enum ResponseSerializationFailureReason {
        case inputDataNil
        case inputDataNilOrZeroLength
        case inputFileNil
        case inputFileReadFailed(at: URL)
        case stringSerializationFailed(encoding: String.Encoding)
        case jsonSerializationFailed(error: Error)
        case propertyListSerializationFailed(error: Error)
    }

```
* inputDataNil  服务器响应无内容
* inputDataNilOrZeroLength  服务器响应无内容或者长度为0
* inputFileNil  服务器响文件不存在
* inputFileReadFailed 服务器响应文件不能read
* stringSerializationFailed  字符用提供到编码编码失败
* jsonSerializationFailed  json序列化失败
* propertyListSerializationFailed  属性列表序列化失败


```
ResponseSerializationFailureReason枚举扩展属性
extension AFError.ResponseSerializationFailureReason {
    var failedStringEncoding: String.Encoding? {
        switch self {
        case .stringSerializationFailed(let encoding):
            return encoding
        default:
            return nil
        }
    }

    var underlyingError: Error? {
        switch self {
        case .jsonSerializationFailed(let error), .propertyListSerializationFailed(let error):
            return error
        default:
            return nil
        }
    }
}
```
* failedStringEncoding 扩展属性 返回编码失败的编码
* underlyingError 扩展属性 json序列化失败到error

```
extension AFError.ResponseSerializationFailureReason {
    var localizedDescription: String {
        switch self {
        case .inputDataNil:
            return "Response could not be serialized, input data was nil."
        case .inputDataNilOrZeroLength:
            return "Response could not be serialized, input data was nil or zero length."
        case .inputFileNil:
            return "Response could not be serialized, input file was nil."
        case .inputFileReadFailed(let url):
            return "Response could not be serialized, input file could not be read: \(url)."
        case .stringSerializationFailed(let encoding):
            return "String could not be serialized with encoding: \(encoding)."
        case .jsonSerializationFailed(let error):
            return "JSON could not be serialized because of error:\n\(error.localizedDescription)"
        case .propertyListSerializationFailed(let error):
            return "PropertyList could not be serialized because of error:\n\(error.localizedDescription)"
        }
    }
}
```
* localizedDescription 扩展属性 返回各种错误原因的描述

```
内嵌结构体
struct AdaptError: Error {
    let error: Error
}
```
* error error信息

```
extension Error {
    var underlyingAdaptError: Error? { return (self as? AdaptError)?.error }
}
```
* underlyingAdaptError 扩展属性 返回携带到error信息

2、属性部分
----

```
    case invalidURL(url: URLConvertible)
    case parameterEncodingFailed(reason: ParameterEncodingFailureReason)
    case multipartEncodingFailed(reason: MultipartEncodingFailureReason)
    case responseValidationFailed(reason: ResponseValidationFailureReason)
    case responseSerializationFailed(reason: ResponseSerializationFailureReason)
```

* invalidURL 验证url
* parameterEncodingFailed  参数编码错误
* multipartEncodingFailed  multipart编码失败
* responseValidationFailed 服务器响应验证失败
* responseSerializationFailed 服务器响应数据序列化失败

3、扩展部分
----
```
extension AFError {
    /// Returns whether the AFError is an invalid URL error.
    public var isInvalidURLError: Bool {
        if case .invalidURL = self { return true }
        return false
    }

    /// Returns whether the AFError is a parameter encoding error. When `true`, the `underlyingError` property will
    /// contain the associated value.
    public var isParameterEncodingError: Bool {
        if case .parameterEncodingFailed = self { return true }
        return false
    }

    /// Returns whether the AFError is a multipart encoding error. When `true`, the `url` and `underlyingError` properties
    /// will contain the associated values.
    public var isMultipartEncodingError: Bool {
        if case .multipartEncodingFailed = self { return true }
        return false
    }

    /// Returns whether the `AFError` is a response validation error. When `true`, the `acceptableContentTypes`,
    /// `responseContentType`, and `responseCode` properties will contain the associated values.
    public var isResponseValidationError: Bool {
        if case .responseValidationFailed = self { return true }
        return false
    }

    /// Returns whether the `AFError` is a response serialization error. When `true`, the `failedStringEncoding` and
    /// `underlyingError` properties will contain the associated values.
    public var isResponseSerializationError: Bool {
        if case .responseSerializationFailed = self { return true }
        return false
    }
}

```
* isInvalidURLError  扩展属性 判断当前错误是不是验证url失败
* isParameterEncodingError 扩展属性 判断是否为参数编码错误
* isMultipartEncodingError 扩展属性 判断是否为multipart编码错误
* isResponseValidationError 是否为服务器响应验证错误
* isResponseSerializationError 是否为服务器返回数据序列化失败

```
extension AFError {
    /// The `URLConvertible` associated with the error.
    public var urlConvertible: URLConvertible? {
        switch self {
        case .invalidURL(let url):
            return url
        default:
            return nil
        }
    }

    /// The `URL` associated with the error.
    public var url: URL? {
        switch self {
        case .multipartEncodingFailed(let reason):
            return reason.url
        default:
            return nil
        }
    }

    /// The `Error` returned by a system framework associated with a `.parameterEncodingFailed`,
    /// `.multipartEncodingFailed` or `.responseSerializationFailed` error.
    public var underlyingError: Error? {
        switch self {
        case .parameterEncodingFailed(let reason):
            return reason.underlyingError
        case .multipartEncodingFailed(let reason):
            return reason.underlyingError
        case .responseSerializationFailed(let reason):
            return reason.underlyingError
        default:
            return nil
        }
    }

    /// The acceptable `Content-Type`s of a `.responseValidationFailed` error.
    public var acceptableContentTypes: [String]? {
        switch self {
        case .responseValidationFailed(let reason):
            return reason.acceptableContentTypes
        default:
            return nil
        }
    }

    /// The response `Content-Type` of a `.responseValidationFailed` error.
    public var responseContentType: String? {
        switch self {
        case .responseValidationFailed(let reason):
            return reason.responseContentType
        default:
            return nil
        }
    }

    /// The response code of a `.responseValidationFailed` error.
    public var responseCode: Int? {
        switch self {
        case .responseValidationFailed(let reason):
            return reason.responseCode
        default:
            return nil
        }
    }

    /// The `String.Encoding` associated with a failed `.stringResponse()` call.
    public var failedStringEncoding: String.Encoding? {
        switch self {
        case .responseSerializationFailed(let reason):
            return reason.failedStringEncoding
        default:
            return nil
        }
    }
}
```
* urlConvertible 判断是否为invalidURL类型，是的话返回 url 否则返回nil
* underlyingError 返回错误信息
* responseCode 返回服务器error code
* failedStringEncoding 返回字符串编码失败时使用到编码


```
extension AFError: LocalizedError {
    public var errorDescription: String? {
        switch self {
        case .invalidURL(let url):
            return "URL is not valid: \(url)"
        case .parameterEncodingFailed(let reason):
            return reason.localizedDescription
        case .multipartEncodingFailed(let reason):
            return reason.localizedDescription
        case .responseValidationFailed(let reason):
            return reason.localizedDescription
        case .responseSerializationFailed(let reason):
            return reason.localizedDescription
        }
    }
}
```

* errorDescription 返回错误的描述信息
