http://www.scienjus.com/document-apache-thrift/


在 RPC 选型中，相较于最基础的 HTTP/JSON API，基于 IDL 约束的 Thrift 在跨语言、序列化性能上占有很多优势。但是在实际使用中由于无法享受 HTTP 丰富的资源库，也带来了不少困扰，其中一个比较常见的麻烦问题就是 IDL 的共享以及协议迭代。

一般在相同语言的多个项目中如何共享 IDL 呢？

在我们大部分的 Java 项目中，服务提供方都会定义一个** $project-api 的模块**，专门用来放给其他项目调用相关类，其中自然也包括了 Thrift IDL 生成后的类。我们甚至会在 Thrift.Iface/Client 上再包一层自己实现的 Client，使整个接口定义与 Thrift 这个实现方案彻底无关，即使未来有一天我们将通讯协议换成了其他方案（HTTP/gRPC），使用方也只需要更新一下依赖版本，而不需要改任何代码。

而在多语言的项目中又该如何共享 IDL 呢？

一般的 RPC 服务会存在一个 Service 和多个 Client，在我们开发的 Python 和 Java 项目中，除了 Java Client 使用 Java Service 可以用上述的方式直接共享生成后的 class 文件，其他使用方式都需要将 IDL 文件放入项目源码中，这样就会导致一份 IDL 会存在与多个项目中，而随着项目的迭代，很难做到所有项目之间的版本是相同的，混乱由此而生。而 Thrift 序列化的高效只建立在 Field Id 作为序列化索引实现的基础上，一旦 Field Id 出现了不一致，就会出现很难排查的数据丢失问题。






