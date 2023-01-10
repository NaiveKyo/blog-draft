---
title: 'Project Experiment: Http RequestBody Repeatable Read'
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110746.jpg'
coverImg: /img/20220425110746.jpg
cover: false
toc: true
mathjax: false
date: 2023-01-10 23:32:54
summary: "Servlet 项目 HTTP 请求体重复读"
categories: "Project Experiment"
keywords: "Project Experiment"
tags: "Project Experiment"
---

# 需求分析

项目中如果有这样的需求：针对 POST 请求，需要对请求体中的 JSON 数据进行预处理，比如接口鉴权或者参数校验，此时需要在 Spring MVC 读取 body 之前获取到 body 数据，但是实际情况是请求体的数据是在流中，而流只会被读取一次，为了处理这种场景，我们就需要对 Servlet 容器处理 HTTP 请求体的逻辑进行重构。

技术需求：

- Servlet 容器技术（比如 Tomcat，了解容器针对 HTTP 请求的处理部分，Servlet、Filter 知识，包装器模式、门面模式、职责链模式）；
- Spring MVC（Handler Mapping、HandlerAdapter，了解从请求映射到具体处理的流程，拦截器知识，模板方法模式，建造者模式）；
- Spring Boot（Embed Container，了解 Spring Boot 嵌入 Servlet 容器的相关知识）；
- HTTP 请求具体的处理过程（web 原生过滤器链、Spring 代理过滤器链、拦截器、AOP 的执行顺序）；
- Java I/O （各种类型的流，装饰器模式）；

 Spring MVC 中的一些接口或类：

- `org.springframework.web.servlet.mvc.method.annotation.RequestBodyAdvice`；
  - 在请求体准备好，还未反序列化，被 Handler 处理之前；
- `org.springframework.web.servlet.mvc.method.annotation.ResponseBodyAdvice`；
  - 被 Handler 处理之后；
- `org.springframework.web.util.ContentCachingRequestWrapper`；
  - 在请求体被处理后，对请求体中的内容做缓存处理；
- `org.springframework.mock.web.DelegatingServletInputStream`；
  - Spring test 框架用于测试 controller 而提供的一个 ServletInputStream 的实现，我们可以参考该类对 HttpServletRequestWrapper 做一些扩展；

Servlet 中的某些接口或类：

- `javax.servlet.ServletInputStream`；
  - 从客户端请求中获取数据的输入流；
- `javax.servlet.http.HttpServletRequestWrapper`；
  - 对 HttpServletRequest 的包装，提供了一些便捷实现，是包装器（装饰器）模式的应用；
  - 开发者可以根据需要扩展该接口，用来调整 Servlet 请求；

# 实现

想要实现业务需求，只需要扩展 HttpServletRequestWrapper 即可：

首先提供一个 Filter：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import javax.servlet.FilterChain;
import javax.servlet.ReadListener;
import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.StandardCharsets;

public class ReadRequestBodyFilter extends HttpFilter {
    
    private static final Log log = LogFactory.getLog(ReadRequestBodyFilter.class);

    private ObjectMapper objectMapper;
    
    public ReadRequestBodyFilter(ObjectMapper objectMapper) {
        super();
        this.objectMapper = objectMapper;
    }

    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        CustomRequestWrapper wrapper = new CustomRequestWrapper(request);
        String body = wrapper.getBody();
        
        log.warn("request body json: " + body);
        
        // TODO 这里根据业务需求对数据进行处理
        
        chain.doFilter(wrapper, response);
    }
    
    // 流只会被读取一次, 无法重复读, 但是可以先从流中读取数据, 备份数据后将其再次写入到另一个流中
    static class CustomRequestWrapper extends HttpServletRequestWrapper {
        
        private final String body;
        
        InputStream delegate = null;

        public CustomRequestWrapper(HttpServletRequest request) throws IOException {
            super(request);
            ServletInputStream is = request.getInputStream();
            BufferedInputStream bis = new BufferedInputStream(is, 1024);
            ByteArrayOutputStream baos = new ByteArrayOutputStream(1024);
            byte[] buf = new byte[1024];
            int len;
            while ((len = bis.read(buf)) != -1) {
                baos.write(buf, 0, len);
            }
            body = new String(baos.toByteArray(), StandardCharsets.UTF_8);
            
            baos.close();
            bis.close();
            is.close();

            delegate = new ByteArrayInputStream(body.getBytes(StandardCharsets.UTF_8));
        }

        public String getBody() {
            return body;
        }

        @Override
        public ServletInputStream getInputStream() throws IOException {
            return new ServletInputStream() {
                @Override
                public boolean isFinished() {
                    return false;
                }
                
                @Override
                public boolean isReady() {
                    return true;
                }

                @Override
                public void setReadListener(ReadListener listener) {
                    throw new UnsupportedOperationException();
                }

                @Override
                public int read() throws IOException {
                    return delegate.read();
                }

                @Override
                public int read(byte[] b, int off, int len) throws IOException {
                    return delegate.read(b, off, len);
                }

                @Override
                public int read(byte[] b) throws IOException {
                    return delegate.read(b);
                }

                @Override
                public long skip(long n) throws IOException {
                    return delegate.skip(n);
                }

                @Override
                public int available() throws IOException {
                    return delegate.available();
                }

                @Override
                public void close() throws IOException {
                    delegate.close();
                }

                @Override
                public synchronized void mark(int readlimit) {
                    delegate.mark(readlimit);
                }

                @Override
                public synchronized void reset() throws IOException { 
                    delegate.reset();
                }

                @Override
                public boolean markSupported() {
                    return delegate.markSupported();
                }
            };
        }
    }
}
```

简单提一下其中的关键点：

- 日志使用 Springframework 默认使用的 JCL；
- 序列化使用的是 Jackson；
- 在 Spring MVC 从流中提取请求体数据之前，我们先将数据提取出来，然后备份该数据，最后将该数据写入另一个流并提供给 Spring MVC 以供读取数据；
- 对 ServletInputStream 的实现参考：DelegatingServletInputStream

将该过滤器注入 web 原生过滤器链的首部：

```java
@Configuration(proxyBeanMethods = false)
public class TomcatConfiguration {
    
    @Resource
    private ObjectMapper objectMapper;
    
    @Bean
    public FilterRegistrationBean<ReadRequestBodyFilter> readRequestBodyFilter() {
        FilterRegistrationBean<ReadRequestBodyFilter> registrationBean = new FilterRegistrationBean<>(new ReadRequestBodyFilter(this.objectMapper));
        registrationBean.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
        registrationBean.setName("readRequestBodyFilter");
        registrationBean.addUrlPatterns("/api/*");
        
        return registrationBean;
    }
}
```

这里根据需要定制该过滤器行为；

测试 Controller：

```java
@RestController
public class RequestBodyReadController {
    
    @PostMapping("/api/test")
    public String test(@RequestBody Object map) {
        return map.toString();
    }
}
```

