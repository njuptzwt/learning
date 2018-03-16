自定义HtttpMesssageConvert实现Jsonp的跨域传输请求：Jsonp格式是一个跨域请求的数据格式，通常情况下自定义带了一个callback的参数：



SpringMVC中自定义实现HttpMessageConvert的实现如下所示：

```
import java.io.IOException;
import java.io.OutputStream;

import javax.servlet.http.HttpServletRequest;

import org.apache.commons.lang.StringEscapeUtils;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpOutputMessage;
import org.springframework.http.converter.HttpMessageNotWritableException;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
//浏览器端跨域请求，返回Jsonp的格式，如果不是跨域请求，返回Json的格式
//如果有带callback参数跨域请求，使用Jsonp的格式输出
//如果没有带callback参数，那么实际上是直接输出的Json格式数据有进行判断
//可以为springMVC添加解析器，处理自定义的Http请求消息
public class FastJsonpHttpMessageConverter<T> extends FastJsonHttpMessageConverter {

   public static final String CALLBACK = "callback";

   //重写，处理输出数据到response中
   @Override
   protected void writeInternal(Object obj, HttpOutputMessage outputMessage)
         throws IOException, HttpMessageNotWritableException {
      HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
            .getRequest();
      String result;

      HttpHeaders headers = outputMessage.getHeaders();
      //将返回的对象以自定义的Json格式输出
      String jsonString = JSON.toJSONString(obj, getFastJsonConfig().getSerializeConfig(), //
            getFastJsonConfig().getSerializeFilters(), //
            getFastJsonConfig().getDateFormat(), JSON.DEFAULT_GENERATE_FEATURE, //
            getFastJsonConfig().getSerializerFeatures());
      String callbackParamValue = request.getParameter(CALLBACK);
      if (callbackParamValue != null && !callbackParamValue.equals("")) {// jsonp格式返回，callback参数的形式
         result = new StringBuilder(StringEscapeUtils.escapeHtml(callbackParamValue)).append("(").append(jsonString)
               .append(")").toString();
      } else {
         result = jsonString;
      }

        headers.setContentLength(result.getBytes(getFastJsonConfig().getCharset()).length);
      OutputStream out = outputMessage.getBody();
      out.write(result.getBytes(getFastJsonConfig().getCharset()));
      return;
   }
}

```

跨域传输的请求。同时需要在SpringMVC中配置相应的消息转换器，实现最终的配置。

```
//配置自定义的Jsonp的HttpConvertMessage对象,当需要使用springMVC的功能时又要使用自定义的方法时候！
@Configuration
@EnableWebMvc
@ComponentScan("com.netease.mail.vip.juwan.web.controller")
public class MyMvcConfig extends WebMvcConfigurerAdapter {


//该方法不会覆盖其他的消息转换器，默认添加一个消息的转换器。
   @Override
   public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
      converters.add(converter());
   }

   @Bean
   public FastJsonpHttpMessageConverter<?> converter() {
      //添加自定义的FastJsonp
      FastJsonpHttpMessageConverter<Object> f = new         FastJsonpHttpMessageConverter<Object>();
      f.setFastJsonConfig(fastJsonConfig());
      List<MediaType> l = new ArrayList<>();
      l.add(MediaType.TEXT_PLAIN);
      l.add(MediaType.APPLICATION_JSON_UTF8);
      f.setSupportedMediaTypes(l);
      return f;
   }

   @Bean
   public FastJsonConfig fastJsonConfig() {
      //修改FastJson返回的Json数据格式，自定义Json返回的格式
      FastJsonConfig fastJsonConfig = new FastJsonConfig();
      fastJsonConfig.setDateFormat("yyyy-MM-dd HH:mm");
      fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteMapNullValue);
      return fastJsonConfig;
   }
}
```
知乎上对Jsonp格式的描述：

It's actually not too complicated...

Say you're on domain example.com, and you want to make a request to domain example.net. To do so, you need to cross domain boundaries, a no-no in most of browserland.

The one item that bypasses this limitation is <script> tags. When you use a script tag, the domain limitation is ignored, but under normal circumstances, you can't really **do** anything with the results, the script just gets evaluated.

Enter JSONP. When you make your request to a server that is JSONP enabled, you pass a special parameter that tells the server a little bit about your page. That way, the server is able to nicely wrap up its response in a way that your page can handle.

For example, say the server expects a parameter called "callback" to enable its JSONP capabilities. Then your request would look like:

```
http://www.example.net/sample.aspx?callback=mycallback
```

Without JSONP, this might return some basic JavaScript object, like so:

```
{ foo: 'bar' }
```

However, with JSONP, when the server receives the "callback" parameter, it wraps up the result a little differently, returning something like this:

```
mycallback({ foo: 'bar' });
```

As you can see, it will now invoke the method you specified. So, in your page, you define the callback function:

```
mycallback = function(data){
  alert(data.foo);
};
```

And now, when the script is loaded, it'll be evaluated, and your function will be executed. Voila, cross-domain requests!

It's also worth noting the one major issue with JSONP: you lose a lot of control of the request. For example, there is no "nice" way to get proper failure codes back. As a result, you end up using timers to monitor the request, etc, which is always a bit suspect. The proposition for [JSONRequest](http://www.json.org/JSONRequest.html)is a great solution to allowing cross domain scripting, maintaining security, and allowing proper control of the request.

These days (2015), [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) is the recommended approach vs. JSONRequest. JSONP is still useful for older browser support, but given the security implications, unless you have no choice CORS is the better choice.