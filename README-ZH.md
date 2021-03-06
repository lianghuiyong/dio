文档语言: [English](https://github.com/flutterchina/dio) | [中文简体](https://github.com/flutterchina/dio/blob/master/README-ZH.md)

# dio 

[![build statud](https://img.shields.io/travis/flutterchina/dio/master.svg?style=flat-square)](https://travis-ci.org/flutterchina/dio)
[![Pub](https://img.shields.io/pub/v/dio.svg?style=flat-square)](https://pub.dartlang.org/packages/dio)
[![coverage](https://img.shields.io/codecov/c/github/flutterchina/dio/master.svg?style=flat-square)](https://codecov.io/github/flutterchina/dio?branch=master)
[![support](https://img.shields.io/badge/platform-flutter%7Cdart%20vm-ff69b4.svg?style=flat-square)](https://github.com/flutterchina/dio)

一个强大的Dart Http请求库，支持Restful API、FormData、拦截器、请求取消、Cookie管理、文件上传/下载、超时等...

### 添加依赖

```yaml
dependencies:
  dio: ^0.0.9
```

## 一个极简的示例

```dart
import 'package:dio/dio.dart';
Dio dio = new Dio();
Response response=await dio.get("https://www.google.com/");
print(response.data);
```

## 内容列表 

- [示例](#示例)

- [Dio APIs](#dio-apis)

- [请求配置](#请求配置)

- [响应数据](#响应数据)

- [拦截器](#拦截器)

- [错误处理](#错误处理)

- [使用application/x-www-form-urlencoded编码](#使用applicationx-www-form-urlencoded编码)

- [FormData](#formdata)

- [转换器](#转换器)

- [设置Http代理](#设置Http代理)

- [请求取消](#请求取消)

- [Cookie管理](#cookie管理)

- [Features and bugs](#features-and-bugs)

  

## 示例

发起一个 `GET` 请求 :

```dart
Response response;
response=await dio.get("/test?id=12&name=wendu")
print(response.data.toString());
// 请求参数也可以通过对象传递，上面的代码等同于：
response=await dio.get("/test",data:{"id":12,"name":"wendu"})
print(response.data.toString());
```

发起一个 `POST` 请求:

```dart
response=await dio.post("/test",data:{"id":12,"name":"wendu"})
```

发起多个并发请求:

```dart
response= await Future.wait([dio.post("/info"),dio.get("/token")]);
```

下载文件:

```dart
response=await dio.download("https://www.google.com/","./xx.html")
```

发送 FormData:

```dart
FormData formData = new FormData.from({
   "name": "wendux",
   "age": 25,
});
response = await dio.post("/info", data: formData)
```

通过FormData上传多个文件:

```dart
FormData formData = new FormData.from({
   "name": "wendux",
   "age": 25,
   "file1": new UploadFileInfo(new File("./upload.txt"), "upload1.txt")
   "file2": new UploadFileInfo(new File("./upload.txt"), "upload2.txt")
});
response = await dio.post("/info", data: formData)
```

…你可以在这里获取所有[示例代码](https://github.com/flutterchina/dio/tree/master/example).

## Dio APIs

### 创建一个Dio实例，并配置它

你可以使用默认配置或传递一个可选 `Options`参数来创建一个Dio实例 :

```dart
Dio dio = new Dio; // 使用默认配置

// 配置dio实例
dio.options.baseUrl="https://www.xx.com/api" 
dio.options.connectTimeout = 5000; //5s
dio.options.receiveTimeout=3000;  

// 或者通过传递一个 `options`来创建dio实例
Options options= new Options(
    baseUrl:"https://www.xx.com/api",
    connectTimeout:5000,
    receiveTimeout:3000
);
Dio dio = new Dio(options);
```

Dio实例的核心API是 :

**Future<Response> request(String path, {data, Options options,CancelToken cancelToken})**

```dart
response=await request("/test", data: {"id":12,"name":"xx"}, new Options(method:"GET"));
```

### 请求方法别名

为了方便使用，Dio提供了一些其它的Restful API, 这些API都是`request`的别名。

**Future<Response> get(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> post(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> put(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> delete(path, {data, Options options,CancelToken cancelToken})**

**Future<Response> head(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> put(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> path(path, {data, Options options,CancelToken cancelToken})** 

**Future<Response> download(String urlPath, savePath,**
    **{OnDownloadProgress onProgress, data, bool flush: false, Options options,CancelToken cancelToken})**


## 请求配置

下面是所有的请求配置选项。 如果请求`method`没有指定，则默认为`GET` :

```dart
{
  /// Http method.
  String method;

  /// 请求基地址,可以包含子路径，如: "https://www.google.com/api/".
  String baseUrl;

  /// Http请求头.
  Map<String, dynamic> headers;

  /// 连接服务器超时时间，单位是毫秒.
  int connectTimeout;

  ///  响应流上前后两次接受到数据的间隔，单位为毫秒。如果两次间隔超过[receiveTimeout]，
  ///  [Dio] 将会抛出一个[DioErrorType.RECEIVE_TIMEOUT]的异常.
  ///  注意: 这并不是接收数据的总时限.
  int receiveTimeout;

  /// 请求数据,可以是任意类型.
  var data;

  /// 请求路径，如果 `path` 以 "http(s)"开始, 则 `baseURL` 会被忽略； 否则,
  /// 将会和baseUrl拼接出完整的的url.
  String path="";

  /// 请求的Content-Type，默认值是[ContentType.JSON].
  /// 如果您想以"application/x-www-form-urlencoded"格式编码请求数据,
  /// 可以设置此选项为 `ContentType.parse("application/x-www-form-urlencoded")`,  这样[Dio]
  /// 就会自动编码请求体.
  ContentType contentType;

  /// [responseType] 表示期望以那种格式(方式)接受响应数据。
  /// 目前 [ResponseType] 接受三种类型 `JSON`, `STREAM`, `PLAIN`.
  ///
  /// 默认值是 `JSON`, 当响应头中content-type为"application/json"时，dio 会自动将响应内容转化为json对象。
  /// 如果想以二进制方式接受响应数据，如下载一个二进制文件，那么可以使用 `STREAM`.
  ///
  /// 如果想以文本(字符串)格式接收响应数据，请使用 `PLAIN`.
  ResponseType responseType;

  /// 用户自定义字段，可以在 [Interceptor]、[TransFormer] 和 [Response] 中取到.
  Map<String, dynamic> extra;
}
```

这里有一个完成的[示例](https://github.com/flutterchina/dio/blob/master/example/options.dart).

## 响应数据

当请求成功时会返回一个Response对象，它包含如下字段：

```dart
{
  /// 响应数据，可能已经被转换了类型, 详情请参考Options中的[ResponseType].
  var data;
  /// 响应头
  HttpHeaders headers;
  /// 本次请求信息
  Options request;
  /// Http status code.
  int statusCode;
  /// 响应对象的自定义字段（可以在拦截器中设置它），调用方可以在`then`中获取.
  Map<String, dynamic> extra;
}
```

示例如下:

```dart
Response response=await dio.get("https://www.google.com");
print(response.data);
print(response.headers);
print(response.request);
print(statusCode);
```

## 拦截器

每一个 Dio 实例都有一个请求拦截器  `RequestInterceptor` 和一个响应拦截器 `ResponseInterceptor`, 通过拦截器你可以在请求之前或响应之后(但还没有被 `then` 或 `catchError`处理)做一些统一的预处理操作。

```dart
 dio.interceptor.request.onSend = (Options options){
     // 在请求被发送之前做一些事情
     return options; //continue
     // 如果你想完成请求并返回一些自定义数据，可以返回一个`Response`对象或返回`dio.resolve(data)`。
     // 这样请求将会被终止，上层then会被调用，then中返回的数据将是你的自定义数据data.
     //
     // 如果你想终止请求并触发一个错误,你可以返回一个`DioError`对象，或返回`dio.reject(errMsg)`，
     // 这样请求将被中止并触发异常，上层catchError会被调用。   
 }
 dio.interceptor.response.onSuccess = (Response response) {
     // 在返回响应数据之前做一些预处理
     return response; // continue
 };
 dio.interceptor.response.onError = (DioError e){
     // 当请求失败时做一些预处理
     return DioError;//continue
 }    
```

如果你想移除拦截器，你可以将它们置为null:

```dart
dio.interceptor.request.onSend=null;
dio.interceptor.response.onSuccess=null;
dio.interceptor.response.onError=null;
```

### 完成和终止请求/响应

在所有拦截器中，你都可以改变请求执行流， 如果你想完成请求/响应并返回自定义数据，你可以返回一个 `Response` 对象或返回 `dio.resolve(data)`的结果。 如果你想终止(触发一个错误，上层`catchError`会被调用)一个请求/响应，那么可以返回一个`DioError` 对象或返回 `dio.reject(errMsg)` 的结果. 

```dart
 dio.interceptor.request.onSend = (Options options){
     return dio.resolve("fake data")    
 }
 Response response= await dio.get("/test");
 print(response.data);//"fake data"
```

### 拦截器中支持异步任务

拦截器中不仅支持同步任务，而且也支持异步任务, 下面是在请求拦截器中发起异步任务的一个实例:

```dart
  dio.interceptor.request.onSend = (Options options) async{
     //...If no token, request token firstly.
     Response response = await dio.get("/token");
     //Set the token to headers 
     options.headers["token"] = response.data["data"]["token"];
     return options; //continue   
 }
```

### Lock/unlock 拦截器

你可以通过调用拦截器的 `lock()`/`unlock` 方法来锁定/解锁拦截器。一旦请求/响应拦截器被锁定，接下来的请求/响应将会在进入请求/响应拦截器之前排队等待，直到解锁后，这些入队的请求才会继续执行(进入拦截器)。这在一些需要串行化请求/响应的场景中非常实用，后面我们将给出一个示例。

```dart
tokenDio=new Dio(); //Create a new instance to request the token.
tokenDio.options=dio;
dio.interceptor.request.onSend = (Options options) async{
     // If no token, request token firstly and lock this interceptor
     // to prevent other request enter this interceptor.
     dio.interceptor.request.lock(); 
     // We use a new Dio(to avoid dead lock) instance to request token. 
     Response response = await tokenDio.get("/token");
     //Set the token to headers 
     options.headers["token"] = response.data["data"]["token"];
     dio.interceptor.request.unlock() 
     return options; //continue   
 }
```

### 别名

当**请求**拦截器被锁定时，接下来的请求将会暂停，这等价于锁住了dio实例，因此，Dio示例上提供了**请求**拦截器`lock/unlock`的别名方法：

**dio.lock() ==  dio.interceptor.request.lock()**

**dio.unlock() ==  dio.interceptor.request.unlock()**

### 示例

假设这么一个场景：出于安全原因，我们需要给所有的请求头中添加一个csrfToken，如果csrfToken不存在，我们先去请求csrfToken，获取到csrfToken后，再发起后续请求。 由于请求csrfToken的过程是异步的，我们需要在请求过程中锁定后续请求（因为它们需要csrfToken), 直到csrfToken请求成功后，再解锁，代码如下：

```dart
dio.interceptor.request.onSend = (Options options) {
    print('send request：path:${options.path}，baseURL:${options.baseUrl}');
    if (csrfToken == null) {
      print("no token，request token firstly...");
      //lock the dio.
      dio.lock();
      return tokenDio.get("/token").then((d) {
        options.headers["csrfToken"] = csrfToken = d.data['data']['token'];
        print("request token succeed, value: " + d.data['data']['token']);
        print('continue to perform request：path:${options.path}，baseURL:${options.path}');
        return options;
      }).whenComplete(() => dio.unlock()); // unlock the dio
    } else {
      options.headers["csrfToken"] = csrfToken;
      return options;
    }
  };
```

完整的示例代码请点击 [这里](https://github.com/flutterchina/dio/blob/master/example/interceptorLock.dart).

## 错误处理

当请求过程中发生错误时, Dio 会包装 `Error/Exception` 为一个 `DioError`:

```dart
  try {
    //404  
    await dio.get("https://wendux.github.io/xsddddd");
   } on DioError catch(e) {
      // The request was made and the server responded with a status code
      // that falls out of the range of 2xx and is also not 304.
      if(e.response) {
        print(e.response.data) 
        print(e.response.headers) 
        print(e.response.request)    
      } else{
        // Something happened in setting up or sending the request that triggered an Error  
        print(e.request)  
        print(e.message)
      }  
  }
```

### DioError 字段

```dart
 {
  /// 响应信息, 如果错误发生在在服务器返回数据之前，它为 `null`
  Response response;

  /// 错误描述.
  String message;
  
  /// 错误类型，见下文
  DioErrorType type;

  /// 错误栈信息，可能为null
  StackTrace stackTrace;
}
```

### DioErrorType

```dart
enum DioErrorType {
  /// Default error type, usually occurs before connecting the server.
  DEFAULT,

  /// When opening  url timeout, it occurs.
  CONNECT_TIMEOUT,

  ///  Whenever more than [receiveTimeout] (in milliseconds) passes between two events from response stream,
  ///  [Dio] will throw the [DioError] with [DioErrorType.RECEIVE_TIMEOUT].
  ///
  ///  Note: This is not the receiving time limitation.
  RECEIVE_TIMEOUT,

  /// When the server response, but with a incorrect status, such as 404, 503...
  RESPONSE,

  /// When the request is cancelled, dio will throw a error with this type.
  CANCEL
}
```



## 使用application/x-www-form-urlencoded编码

默认情况下, Dio 会将请求数据(除过String类型)序列化为 `JSON`. 如果想要以 `application/x-www-form-urlencoded`格式编码, 你可以显式设置`contentType` :

```dart
//Instance level
dio.options.contentType=ContentType.parse("application/x-www-form-urlencoded");
//or works once
dio.post("/info",data:{"id":5}, options: new Options(contentType:ContentType.parse("application/x-www-form-urlencoded")))    
```

这里有一个[示例](https://github.com/flutterchina/dio/blob/master/example/options.dart).

## FormData

Dio支持发送 FormData, 请求数据将会以 `multipart/form-data`方式编码, FormData中可以一个或多个包含文件 .

```dart
FormData formData = new FormData.from({
    "name": "wendux",
    "age": 25,
    "file": new UploadFileInfo(new File("./example/upload.txt"), "upload.txt")
});
response = await dio.post("/info", data: formData)
```

> 注意: 只有 post 方法支持发送 FormData.

这里有一个完整的[示例](https://github.com/flutterchina/dio/blob/master/example/formdata.dart).

## 转换器

转换器`TransFormer` 用于对请求数据和响应数据进行编解码处理。Dio实现了一个默认转换器`DefaultTransformer`作为默认的 `TransFormer`. 如果你想对请求/响应数据进行自定义编解码处理，可以提供自定义转换器，通过 `dio.transformer`设置。

> 请求转换器  `TransFormer.transformRequest(...)`   只会被用于 'PUT'、 'POST'、 'PATCH'方法，因为只有这些方法才可以携带请求体(request body)。但是响应转换器 `TransFormer.transformResponse()` 会被用于所有请求方法的返回数据。

### 执行流

虽然在拦截器中也可以对数据进行预处理，但是转换器主要职责是对请求/响应数据进行编解码，之所以将转化器单独分离，一是为了和拦截器解耦，二是为了不修改原始请求数据(如果你在拦截器中修改请求数据(options.data)，会覆盖原始请求数据，而在某些时候您可能需要原始请求数据). Dio的请求流是：

*请求拦截器* >> *请求转换器* >> *发起请求*  >> *响应转换器*  >> *响应拦截器*  >> *最终结果*。

这是一个自定义转换器的[示例](https://github.com/wendux/dio/blob/master/example/transformer.dart).

## 设置Http代理

Dio 是使用 HttpClient发起的http请求，所以你可以通过配置 `httpClient`来支持代理，示例如下：

```dart
  dio.onHttpClientCreate = (HttpClient client) {
    client.findProxy = (uri) {
      //proxy all request to localhost:8888
      return "PROXY localhost:8888";
    };
  };
```

完整的示例请查看[这里](https://github.com/wendux/dio/tree/master/example/proxy.dart).

## 请求取消

你可以通过 *cancel token* 来取消发起的请求：

```dart
CancelToken token = new CancelToken();
dio.get(url, cancelToken: token)
    .catchError((DioError err){
        if (CancelToken.isCancel(err)) {
            print('Request canceled! '+ err.message)
        }else{
            // handle error.
        }
    })
// cancel the requests with "cancelled" message.
token.cancel("cancelled");
```

> 注意: 同一个cancel token 可以用于多个请求，当一个cancel token取消时，所有使用该cancel token的请求都会被取消。

完整的示例请参考[取消示例](https://github.com/flutterchina/dio/blob/master/example/cancelRequest.dart).

## Cookie管理

你可以通过 `cookieJar` 来自动管理请求/响应cookie.

>  dio cookie 管理 API 是基于开源库 [cookie_jar](https://github.com/flutterchina/cookie_jar).

你可以创建一个`CookieJar` 或 `PersistCookieJar` 来帮您自动管理cookie,  dio 默认使用  `CookieJar` , 它会将cookie保存在内存中。 如果您想对cookie进行持久化,  请使用 `PersistCookieJar` ,  示例代码如下:

```dart
var dio = new Dio();
dio.cookieJar=new PersistCookieJar("./cookies");
```

`PersistCookieJar` 实现了RFC中标准的cookie策略.  `PersistCookieJar` 会将cookie保存在文件中，所以 cookies 会一直存在除非显式调用 `delete` 删除.


更多关于 [cookie_jar](https://github.com/flutterchina/)  请参考 : https://github.com/flutterchina/cookie_jar .

## Copyright & License

此开源项目为Flutter中文网(https://flutterchina.club) 授权 ，license 是 MIT.

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: https://github.com/flutterchina/dio
