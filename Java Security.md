# Java Security

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

本文提到的 Java Security 的某些方面，它们的默认的 provider 是可以通过配置 security properties 来定制的。可以在存有安全配置属性的文件中声明为静态属性，这个文件叫做 `java.security` 位于 JRE 的 `lib/security/` 目录下，也可以通过 `java.security.Security` 类中合适的方法来动态设置相关属性。

### Cryptography

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

进度：https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/jsoverview.html