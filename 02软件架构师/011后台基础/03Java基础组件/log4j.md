https://logging.apache.org/log4j/2.x/manual/thread-context.html


Thread Context
Introduction

Log4j introduced the concept of the Mapped Diagnostic Context or MDC. It has been documented and discussed in numerous places including Log4j MDC: What and Why and Log4j and the Mapped Diagnostic Context. In addition, Log4j 1.x provides support for a Nested Diagnostic Context or NDC. It too has been documented and discussed in various places such as Log4j NDC. SLF4J/Logback followed with its own implementation of the MDC, which is documented very well at Mapped Diagnostic Context.

Log4j 2 continues with the idea of the MDC and the NDC but merges them into a single Thread Context. The Thread Context Map is the equivalent of the MDC and the Thread Context Stack is the equivalent of the NDC. Although these are frequently used for purposes other than diagnosing problems, they are still frequently referred to as the MDC and NDC in Log4j 2 since they are already well known by those acronyms.


Chapter 8: Mapped Diagnostic Context
https://logback.qos.ch/manual/mdc.html



