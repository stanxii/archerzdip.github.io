---
title: 计算机网络协议系列 -   HTTP 协议篇：HTTPS 协议之底层原理篇

layout: post

category: blog

tags: |-

  PHP

  计算机网络协议系列
  
  HTTP 协议篇

  HTTPS
---



# 计算机网络协议系列（五十一）



通过上篇分享的介绍，非对称加密在性能上不如对称加密，但是安全性上要更好，因此 HTTPS 综合运用了这两种加密方式的优势，使用非对称加密传输对称加密需要用到的密钥，而真正的双方大数据量的通信都是通过对称加密进行的，结合数字证书（包含公钥信息）验证服务端公钥的真实性，HTTPS 的底层原理如下：

![img](/assets/post/e6a0fc431c6fdc7d1dd595a29ab77e20e42e6890a5636179e148c825710863c2.png)

1、当我们访问一个 HTTPS 网站的时候，客户端通过发送 Client Hello 报文开始建立与服务器的 SSL 通信，报文中包含了 SSL 协议版本、加密组件、压缩算法等信息，另外，还有一个随机数，用于后续对称加密密钥的协商。

2、服务器可以进行 SSL 通信时，会以 Server Hello 报文作为应答，和客户端一样，在报文中包含 SSL 协议版本、加密组件、压缩算法等信息，同时还有一个随机数，用于后续对称加密密钥的协商。

3、接下来，服务器会以 Certificate 报文的形式给客户端发送服务端的数字证书，其中包含了非对称加密用到的公钥信息。最后，服务器还会发送 Server Hello Done 报文告知客户端，最初阶段的 SSL 握手协商部分结束。

4、客户端当然不相信这个证书，于是从自己信任的 CA 仓库中，拿 CA 证书里面的公钥去解密 HTTPS 网站的数字证书（证书是通过 CA 私钥加密的，所以要用公钥解密），如果能够成功，则说明 HTTPS 网站是可信的。

5、证书验证完毕之后，觉得这个 HTTPS 网站可信，于是客户端计算产生随机数字 Pre-master，用服务器返回的数字证书中的公钥加密该随机数字，再通过 Client Key Exchange 报文发送给服务器，服务器可以通过对应的私钥解密出 Pre-master。

到目前为止，无论是客户端还是服务器，都有了三个随机数，分别是：自己的、对端的，以及刚生成的 Pre-Master 随机数。通过这三个随机数，可以在客户端和服务器生成相同的对称加密密钥。

6、有了对称加密密钥，客户端就可以通过 Change Cipher Spec 报文告知服务器以后都采用该密钥和协商的加密算法进行加密通信了。

7、然后客户端还会发送一个 Encrypted Handshake Message 报文，将已经商定好的参数，采用对称加密密钥进行加密，发送给服务器用于数据与握手验证。

8、同样，服务器也可以发送 Change Cipher Spec 报文，告知客户端以后都采用协商的对称加密密钥和加密算法进行加密通信了，并且也发送 Encrypted Handshake Message 报文进行测试。当双方握手结束之后，就可以通过对称加密密钥进行加密传输了。

这个过程除了加密解密之外，其他的过程和 HTTP 是一样的，过程也非常复杂。

上面的过程只包含了 HTTPS 的单向认证，也即客户端验证服务端的证书，大部分场景下使用的都是单向认证，也可以在更加严格安全要求的情况下，启用双向认证，双方互相验证证书，比如银行的网银系统就是这样，需要客户端安装数字证书后才能登录，其实就要服务端要要验证客户端的合法性。

**SSL 与 TLS**

TLS（Transport Layer Security，传输层安全）是 SSL 的升级版（Secure Socket Layer，安全套接层），建立在 SSL 3.0 协议规范之上，由于 SSL 这一术语非常常用，所以通常我们把 SSL 和 TLS 统称为 SSL，从这个角度来说，不管是基于 TLS 还是 SSL 实现的 HTTPS，将其描述为 HTTP over SSL 并没有什么问题。