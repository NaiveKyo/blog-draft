---
title: Java Security Mechanism Summary
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110725.jpg'
coverImg: /img/20220425110725.jpg
cover: false
toc: true
mathjax: false
date: 2022-12-28 23:06:43
summary: "Java 安全机制概要"
categories: "Java"
keywords: ["Java", "Security"]
tags: ["Java", "Security"]
---

# JavaSecurity

参考：（JDK 8）

- https://docs.oracle.com/javase/8/docs/technotes/guides/security/index.html

Java 的 Security 技术包含了很多 API、工具以及通用安全算法、基础设施、协议的实现。Java Security API 有广泛的应用范围，比如密码学、公钥基础设施、加密通信、认证以及访问控制。

Java Security 技术为开发者提供了全面的安全框架用于编写应用程序，同时也为用户和管理员提供了一套安全管理工具。

本文着重学习一些通用的安全机制，更多信息参考官网。



# Java Security Overview

参考：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html

## 1. 介绍

Java 平台的设计非常强调安全性，就其核心而言，Java 语言本身是类型安全的，并提供了自动垃圾收集，从而增强了应用程序代码的健壮性。安全的类加载以及字节码校验机制确保只会执行合法的 Java 代码。

Java 平台最初的版本考虑到运行的代码可能存在潜在的风险（比如从公共网络下载的 Java applet），因此提供了一个较为安全的运行环境，而随着平台的壮大和越打越多的开发者参于进来，Java 的安全体系结构也随之进行了改变，以支持越来越多的服务集合。到目前为止，Java 安全机制已经包含了大量的应用程序编程接口（APIs）、工具、一些通用安全算法、机制、协议的实现。这为开发者编写应用程序提供了综合性的安全框架，也为使用者和管理员提供了一套安全管理工具。

Java 的 Security API 涵盖了广泛的领域。密钥和公钥基础设施（PKI）接口为开发安全的应用程序提供了基础，身份验证和访问控制相关接口使应用程序能够阻止对受保护资源的未经授权的访问。

除了默认的实现外，API 也允许使用者自行提供算法或者其他安全服务的实现。提供者可以自行实现相关服务，然后通过 Java 平台规定的标准接口插入到 Java 平台中，这使得应用程序很容易获得安全服务，而不需要了解它们的具体实现。这使得开发者可以专注于将安全框架集成到程序中，而不是如何实现一套安全服务。

Java 平台已经包含了许多安全服务的核心实现，它还允许安装额外的自定义提供方，这使得开发者可以为平台扩展新的安全机制。

下面看一下 Java 8 中的安全机制。

## 2. Java Language Security and Bytecode Verification

Java 语言被设计成类型安全且非常容易使用。它提供了自动内存管理机制、垃圾收集机制以及数组的范围检查，这减轻了开发人员的负担，可以减少错误的代码并提供更安全、更健壮的代码。

此外，Java 提供了很多访问修饰符用于控制对类、方法和熟悉的访问，允许开发者适当的限制对类的实现者的访问。特别的，Java 语言定义了四种访问级别：`private、protected、public` 以及默认的 `pakcage`。

- public 是开放程度最大的，允许访问所有内容；
- private 是开放程序最低的，在持有私有成员的类之外的任何地方都不能访问；
- protected 则允许子类访问或者访问在同一包下的其他类；
- package 则只能是在同一包下才可以访问。

编译器将 Java 程序转换为和系统机器无关的字节码，而字节码校验机制则确保在 Java 运行时只执行合法的字节码，它检查字节码是否符合 Java 语言规范，并且不违反 Java 语言规则或名称空间限制。该校验器同样会检查违规的内存管理、堆栈下溢或溢出以及非法类型转换。一旦字节码通过了验证，Java 运行时就可以执行它们。



## 3. Basic Security Architecture

Java 平台定义了一组 API 涵盖主要的安全领域，cryptography, public key infrastructure, authentication, secure communication, and access control.  这些 api 可以让开发者很容易的为应用程序提供安全机制，它们是围绕以下理念进行设计的：

> Implementation independence（独立的实现）

应用程序不需要实现 Security API，相反，他们可以从 Java 平台得到 Security 支持。Security services 是由 providers 提供的，通过 standard interface 集成到 Java 平台中。一个应用程序可以集成多个 Security 服务的 providers。

> Implementation interoperability（实现具有互通性）

Providers 提供的安全实现在应用程序中是可以互通的，具体来说，应用程序不会绑定某个特定的 provider，一个 provider 也不会只绑定到某个应用程序。

> Algorithm extensiblity（可扩展的算法实现）

Java 平台内置了一些 providers，它们实现了一组通用的基础安全服务，但是，有一些应用程序会依赖当前流行的安全服务或者专有的服务，不用担心，Java 平台也提供了 provider 的安装功能用于扩展安全服务。

### Security Providers

`java.security.Provider` 类封装了 Java 平台定义的 Security Provider 的概念。它指定了 provider 的名称并列出它实现的安全服务，可以同时配置多个 provider，并按照优先级顺序列出来。当我们需要使用到安全服务时，将选择该服务的实现者中优先级最高的提供者。

应用程序通过使用 `getInstance` 方法从底层的 provider 中获得具体的安全服务。比如说，消息摘要创建服务就是 providers 提供的一种安全服务。应用程序可以调用 `java.security.MessageDigest` 类中的 `getInstance` 方法获取特定摘要算法的实现，比如说 SHA-256；

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
```

当然也可以有选择地从指定的 provider 中获取服务实现：

```java
MessageDigest md = MessageDigest.getInstance("SHA-256", "ProviderC");
```

### File Locations

本文提到的 Java Security 的某些方面，它们使用的默认的 provider 是可以通过配置 security properties 来定制的。可以在存有安全配置属性的文件中声明为静态属性，这个文件叫做 `java.security` 位于 JRE 的 `lib/security/` 目录下，也可以通过 `java.security.Security` 类中合适的方法来动态设置相关属性。

## 4. Cryptography

Java 的加密体系结构是用于访问和开发 Java 平台加密功能的框架。它包括用于各种加密服务的 API，比如：

- Message digest algorithms（消息签名算法）；
- Digital signature algorithms（数字签名算法）；
- Symmetric bulk encryption（对称加密算法）；
- Symmetric stream encryption（对称流加密算法）；
- Asymmetric encryption（非对称加密）；
- Password-base encryption（PBE）（基于密码的加密）；
- Elliptic Curve Cryptography（ECC）（椭圆曲线密码机制）；
- Key agreement algorithms（一致性加密算法）；
- Key generators（生成密钥）；
- Message Authentication Codes（MACs）（消息认证码）；
- （Pseudo-）random number generators（随机数生成器）；

处于历史原因，加密 API 被分到两个包下面：

- `java.security`  下包含的类诸如 Signature 和 MessageDigest 是可以随意使用的，不会受到 export control；
- `javax.crypto` 下面包含的类诸如 Cipher 和 KeyAgreement 会受到 export control；

加密接口也是基于 provider 的，允许多个可互通的加密实现。一些 providers 可能是在软件中执行加密操作；另一些也可能对硬件令牌执行操作，比如在智能卡设备或者硬件加密加速器上。

实现 export control 服务的 providers 必须进行数字签名。

Java 平台内置了很多常用的加密算法的 provider，比如 RSA、DSA 和 ECDSA 签名算法，AES 加密算法、SHA-2 报文摘要算法、Diffie-Hellman（DH）和椭圆曲线 Diffie-Hellman（ECDH）密钥协商算法。大多数内置 provider 都是用 Java 代码实现加密算法的。

Java 平台还提供一些桥接的 provider，比如对 native PKCS#11（v2.x）token 的桥接。这个 provider 叫做 SunPKCS11，允许 Java 程序访问位于 PKCS#11 兼容令牌上的加密服务。

在 Windows 平台，Java 也提供了一个内置的桥接 provider 用于调用 native Microsoft CryptoAPI，它叫做 SunMSCAPI，允许 Java 应用程序通过 CryptoAPI 无缝地访问 Windows 上的加密服务。



## 5. Public Key Infrastructure

Public Key Infrastructure（PKI）是一个形容某些框架的术语，这些框架往往实现了基于公钥密钥匹配来进行信息安全交换的机制。它允许将某种身份（比如个人、组织等等）和数字证书进行绑定，并提供一种证书有效性验证方法。

PKI 包括密钥、证书、公钥加密和生成证书并对证书进行数字签名的可信任的证书颁发机构（CAs）。

Java 平台提供了一些 API 用于支持 X.509 数字证书和  Certificate Revocation Lists (CRLs)（证书吊销列表），以及符合 PKIX 标准的证书构造和校验规则，和 PKI 相关的类在 `java.security` 和 `java.security.cert` 包下。

### Key and Certificate Storage

密钥和证书存储：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html

### PKI Tools

有两个内置工具集用于密钥、证书和密钥存储：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html



## 6. Authentication

Authentication（身份校验） 是确定用户身份的过程，在 Java 运行时上下文中，它是识别正在操作 Java 程序的用户的过程，在某些情况下，这个过程会依赖 "Cryptography" 中的某些服务。

Java 平台提供了 API 使得应用程序可以通过可拔插的登录模块执行用户身份验证。

更多信息参考：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html

## 7. Secure Communication 加密通信

通过网络传输的数据可能会被某些恶意攻击者截取，特别是当这些信息包含某些敏感信息的时候，比如密码和证件，因此需要采取一些措施使得非法获取信息的人无法读取信息。同样重要的是，要确保数据发送给适当的一方，并且在传输过程中数据没有被有意或无意的修改。

密码学是加密通信的基础，Java 平台为许多标准的安全通信协议提供 API 支持和 Provider 实现。

### SSL/TLS

（Tip：TLS 其实就是 SSL，有一些历史来源可自行查阅）

Java 平台为 SSL（Secure Sockets Layer） 和 TLS（Transport Layer Security） 协议提供了 API 和实现支持，包括数据加密、消息完整性、服务器身份验证和可选的客户端身份验证功能。应用程序可以使用 SSL/TLS 在任何应用程序协议（如 TCP/IP 上层的 HTTP 协议）上的两个对等体之间提供数据的安全传输。

`java.net.ssl.SSLSocket` 类在常规套接字流（`java.net.Socket`）的基础上集成了 SSL/TLS 功能。

Java 平台还支持可拔插的（基于 Provider 机制）的 key managers 和 trust managers 两个概念。前者使用 `javax.net.ssl.KeyManager` 接口进行抽象，并管理用于进行身份验证的密钥，后者用同一包下的 `TrustManager` 接口进行抽象，并根据其管理的密钥存储中的证书来决定信任谁。

Java 平台提供了内置的 Provider 实现 SSL/TLS 协议：

- SSLv3；
- TLSv1；
- TLSv1.1；
- TLSv1.2；

### SASL

Simple Authenication and Security Layer（SASL）（简单认证和安全层协议）是一种互联网标准，它提供了一种认证协议，并可选地在客户端和服务器应用程序之间建立安全层。SASL 定义了如何交换身份验证数据，但本身没有指定该数据的内容。它是一个框架，指定身份验证数据的内容和语义的特定身份验证机制可以适用于该框架。Internet 社区为各种安全级别和部署场景定义了许多标准 SASL 机制。

Java SASL API 为使用 SASL 机制的应用程序定义了一些类和接口。使用这些 API 的应用程序不必和某些 SASL 机制硬绑定，而是可以根据所需的安全特性来选择要使用的机制。该 API 同时支持客户端和服务器端应用程序。`javax.security.sasl.Sasl` 类用于创建 `SaslClient` 和 `SaslServer` 实例。

SASL 机制实现由对应的 Provider 包提供。每个 provider 可以支持一个或多个 SASL 机制，并通过标准 provider 结构进行注册和调用。

Java 平台为 SASL 提供了一个内置的 provider 实现：

- CRAM-MD5、DIGEST-MD5、EXTERNAL、GSSAPI、NTLM，and PLAIN client mechanisms；
- CRAM-MD5, DIGEST-MD5, GSSAPI, and NTLM server mechanisms；

### GSS-API and Kerberos

Java 平台包含了用于 Generic Security Service Apllication Programming Interface（GSS-API，通用安全服务应用程序编程接口）的 Java 语言绑定的 API。GSS-API 为应用程序开发人员提供了各种底层安全机制之上的安全服务的统一访问机制。

跟多信息参考：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html

## 8. Access Control

Java 平台的 Access Control 机制用于保护对敏感资源（比如：本地文件）或者敏感程序代码（比如：类中的方法）的访问。所有的访问控制决策都是由 security manager 决定，即 `java.lang.SecurityManager` 类。为了激活访问控制检查机制，必须在运行时注册一个 SecurityManager 实例。

Java applets 或 web Start 程序在启动的时候会自动注入一个 SecurityManager，但是通过 Java 命令行工具启动的应用程序并不会自动注入 SecurityManager，此时我们有两种方法自动注册：

- 程序中硬编码，使用 `java.lang.System#setSecurityManager` 注入实例；
- 启动参数中添加：`-Djava.security.Manager`；

### Permissions

当 Java 类加载器将 Java 代码加载到运行时环境中时，类加载器自定将以下信息和代码关联：

- Where the code was loaded from
- Who signed the code (if anyone)
- Default permissions granted to the code

无论代码是通过不可信的网络（比如：applet）下载还是从文件系统（例如：本地应用程序）加载，这些信息都会和代码关联。

- 加载代码的位置由 URL 表示；
- 代码签名者由签名者的证书链表示；
- 默认权限由 `java.security.Permission` 对象表示。

从网络上下载的代码会自动授予的默认权限包括能够将网络连接到其来源的主机；

从本地文件系统中加载的代码会自动授予的默认权限包括从其所在目录及该目录的子目录中读取文件的能力。

注意，执行代码的用户的身份在类加载时是不可用的。应用程序代码有责任在必要时对用户进行身份验证（比如对敏感资源的操作），一旦用户通过身份验证，应用程序就可以通过调用 `javax.security.auth.Subject` 类中的 `doAs` 方法，动态地将该用户与执行的代码关联起来。

### Policy

前面也提到过，类加载器向代码授予一组有限的默认权限，而管理员可以通过安全策略灵活地管理额外的代码权限。

Java 平台提供了 `java.security.Policy` 这个概念，并且在任何时间，Java 运行时只会注册一个 Policy 对象。Policy 对象的基本职责是确定是否允许编写对受保护资源的访问（它是从哪里加载的？谁对其进行了签名以及谁正在执行它？）。Policy 对象具体的执行逻辑取绝于具体的实现，比如它可以咨询包含授权数据的数据库，也可以联系另一个服务。

Java 平台提供了一个默认的 Policy 实现，它从安全属性文件中配置的一个或多个 ASCII（UTF-8）文件中读取授权数据。这些策略文件中包含授予代码的确切权限集：具体的说，授予从特定位置加载、由特定实体签名并作为特定用户执行的代码的确切权限集。每个文件中的策略条目必须符合文档中的专有语法，并且可以通过简单的文本编辑器或于图形化的工具程序结合使用。

### Access Control Enforcement 执行访问控制

Java 运行时会保持跟踪程序执行时所进行的 Java calls 的顺序。当请求访问受保护的资源时，默认情况下，将评估整个调用堆栈，以确定所请求的访问是被允许的。

前面提到过，资源由 SecurityManager 保护。Java 平台和应用程序中的安全敏感代码通过如下代码对资源进行访问：

```java
SecurityManager sm = System.getSecurityManager();
if (sm != null) {
    sm.checkPermission(perm);
}
```

这里的 perm 是 Permission 类的实例对象，对应着当前的请求访问，比如说，如果试图读取 `/tmp/abc` 文件，则权限构造可能是这样的：

```
Permission perm = new Java.io.FilePermission("/tmp/abc", "read");
```

SecurityManager 的默认实现将其决策委托给 `java.seucurity.AccessController` 实现。AccessController 遍历调用堆栈，将堆栈中的每个代码元素以及请求的权限（例如，上面例子中的 FilePermission）传递给已经注册的 Policy 实例。Policy 根据管理员配置的权限决定是否授予所请求的访问。如果未授予访问权，AccessController 将抛出 `java.lang.SecurityException`。



## 9. XML Signature

Java XML 数字签名 API 是用于生成和验证 XML 签名的标准 Java API。

XML 签名可以应用于任何类型的数据，XML 或二进制（参考：http://www.w3.org/TR/xmldsig-core/）。生成的签名用 XML 表示，XML 签名可用于保护数据并提供数据完整性，消息身份验证和签名者身份验证。

该 API 旨在支持 W3C 推荐的 XML 签名语法和处理所有必须或推荐特性。该 API 也是可扩展和可插入的，并且基于 Java 加密服务 Provider 体系。

参考：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html

## 10. Java API for XML Processing（JAXP）

JAXP 是 Java 提供了用于处理 XML 数据的 API。它包括对简单 API（SAX）、文档对象模型（DOM）和 XML Stream API（StAX）解析器、XML 模式验证和可扩展样式表语言转换（XSLT）的支持。此外，JAXP 提供了安全处理特性，可以帮助您的应用程序和系统免受与 XML 相关的攻击，具体参考：https://docs.oracle.com/javase/8/docs/technotes/guides/security/jaxp/jaxp.html

**Note**: [Secure Coding Guidelines for Java SE](https://www.oracle.com/technetwork/java/seccodeguide-139067.html) contains additional recommendations that can help defend against XML-related attacks.

## 11. For More Information

Additional Java security documentation can be found online at

[Java SE Security](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136007.html)

and in the book [Inside Java 2 Platform Security, Second Edition: Architecture, API Design and Implementation](http://www.oracle.com/technetwork/java/javaee/gong-135902.html).

**Note**: Historically, as new types of security services were added to the Java platform (sometimes initially as extensions), various acronyms were used to refer to them. Since these acronyms are still in use in the Java security documentation, here is an explanation of what they represent: JSSE (Java™ Secure Socket Extension) refers to the SSL-related services described in Section 7, JCE (Java™ Cryptography Extension) refers to cryptographic services (Section 4), and JAAS (Java™ Authentication and Authorization Service) refers to the authentication and user-based access control services described in Sections 6 and 8, respectively.



# TODO

对 Java 安全体系有了一个大概的认知后，后面可以考虑看看：

- https://docs.oracle.com/javase/8/docs/technotes/guides/security/spec/security-specTOC.fm.html

简单了解 Java 中各种权限：文件、网络、序列化、类加载等等。

