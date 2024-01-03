# Reference

- https://www.baeldung.com/spring-mvc-sse-streams
- https://docs.spring.io/spring-framework/docs/5.3.30/reference/html/web.html#mvc-ann-async-sse

使用案例：

- https://medium.com/@vijitsh/asynchronous-sse-server-sent-events-in-spring-d9e5e0ec94f6
- https://liakh-aliaksandr.medium.com/server-sent-events-sse-in-spring-5-with-web-mvc-and-web-flux-44c926b59f36



# Spring MVC Support

Spring MVC 中 Asynchronous Requests 下的 HTTP Streaming 章节提出了有关 SSE 的支持。

几个核心的类：

- org.springframework.web.servlet.mvc.method.annotation.ResponseBodyEmitter、
- org.springframework.web.servlet.mvc.method.annotation.SseEmitter
- org.springframework.web.servlet.mvc.method.annotation.StreamingResponseBody

