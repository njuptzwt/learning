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