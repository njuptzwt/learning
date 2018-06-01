springMVC中的请求参数的传递问题，获取不到参数的问题！



在http请求的头中去设置参数传递的格式，Content-Type：

@RequestParam

用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容。（Http协议中，如果不指定Content-Type，则默认传递的参数就是application/x-www-form-urlencoded类型）

参数的请求类型为: key=value的形式，可以直接进行对象的Getter和Setter方式:



@RequestBody

处理HttpEntity传递过来的数据，一般用来处理非`Content-Type: application/x-www-form-urlencoded`编码格式的数据。

处理如下的数据格式：

@RequestBody这个一般处理的是在ajax请求中声明contentType: "application/json; charset=utf-8"时候。也就是json数据或者xml(我没用过这个，用的是json)

