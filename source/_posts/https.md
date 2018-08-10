---
title: https运行原理解析笔记
date: 2018-08-06 17:59:24
tags: [https]
categories:
- 技术博客
- 笔记
---

# HTTPS运行原理解析笔记

HTTPS可以看作是安全的HTTP，逆可能听说过关于HTTPS的一些问题，比如什么握手，什么证书，加密之类的等等。

HTTPS为何能保障web的安全，其运行原理是怎样的，当我们深入了解下去，其设计的思路对我们其他安全方面的设计也有一定的启发作用。

<!-- more -->

## 与HTTP的区别
简单来说，HTTPS就是在HTTP的基础上加了一层安全协议，用以保障数据安全。

```

      HTTP            HTTPS

    |--------|     |-----------|
    |HTTP    |     |HTTP       |
    |--------|     |-----------|
    |TCP     |     |SSL or TLS |
    |--------|     |-----------|
    |IP      |     |TCP        |
    |--------|     |-----------|
                   |IP         |
                   |-----------|

```
如上图，HTTPS相比HTTP多了一层SSL/TLS。

## HTTPS流程

![https流程图](/pic/https/https.svg)

HTTPS流程包含握手和后续的数据传输，握手的目的是为了客户端与服务端协商加密算法等参数。

SSL/TLS基本过程是这样的：

1. 客户端向服务器端所要并验证证书
2. 双方协定加密算法以及“对话密钥”
3. 双方采用协商后的“对话密钥”进行加密通信

前两步，被称作“握手阶段”。

握手流程如上面图中红色部分。

1. 客户端首次请求服务器，告诉服务器自己支持的协议版本，支持的加密算法及压缩算法，并生成一个随机数（client random）告知服务器。
  > 客户端需要提供的信息：支持的协议版本，如TSL1.0版本；客户端生成的随机数，用以稍后生成“对话密钥”；支持的加密算法；支持的压缩方法

2. 服务器确认双方使用的加密方法，并返回给客户端证书以及一个服务器生成的随机数（server random）
  > 服务器需要提供的信息：协议的版本；加密的算法；服务器生成的随机数；服务器证书；

3. 客户端收到证书后，首先验证证书的有效性，然后生成一个新的随机数（premaster secret），并使用数字证书中的公钥，加密这个随机数，发送给服务器。
  > 客户端会对服务器下发的证书进行验证，验证通过后，客户端会再次生成一个随机数（premaster secret），然后使用服务器证书中的公钥进行加密，以及放一个ChangeCipherSpec消息即编码改变的消息，还有整个前面所有消息的hash值，进行服务器验证，然后用新秘钥加密一段数据一并发送到服务器，确保正式通信前无误。

4. 服务器接收到加密后的随机数后，使用私钥进行解密，获取这个随机数（premaster secret）
5. 服务器和客户端根据约定的加密方法，使用前面的三个随机数（client random, server random, premaster secret），生成“对话密钥”（session key），用来加密接下来的整个对话过程。


从上面的流程来看，需要注意一下几点：

* 生成对话密钥需要三个随机数，但整个通话的安全，只取决于第三个随机数能不能被破解。因为前两个随机数通信过程中并未加密，明文通信，存在被拦截的可能。
* 握手之后的对话，使用“对话密钥”进行加密，采用对称加密。服务器的公钥和私钥只用于加密和解密“对话密钥”

## 几个问题
### 为什么握手过程需要三个随机数，而且安全性只取决于第三个随机数？
前两个随机数采用明文传输，存在被拦截的风险，最终对话密钥安全性只和第三个随机数有关，那么前两个随机数有没有必要？
"不管是客户端还是服务器，都需要随机数，这样生成的密钥才不会每次都一样。由于SSL协议中证书是静态的，因此十分有必要引入一种随机因素来保证协商出来的密钥的随机性。

对于RSA密钥交换算法来说，pre-master-key本身就是一个随机数，再加上hello消息中的随机，三个随机数通过一个密钥导出器最终导出一个对称密钥。

pre master的存在在于SSL协议不信任每个主机都能产生完全随机的随机数，如果随机数不随机，那么pre master secret就有可能被猜出来，那么仅适用pre master secret作为密钥就不合适了，因此必须引入新的随机因素，那么客户端和服务器加上pre master secret三个随机数一同生成的密钥就不容易被猜出了，一个伪随机可能完全不随机，可是是三个伪随机就十分接近随机了，每增加一个自由度，随机性增加的可不是一。"

所以简单来说，采用三个随机数是为了是最终的对话密钥更“随机”。

### 为什么HTTPS的流程设计这么复杂，只用对称加密或非对称加密加密信息不可么？
不可。

对于对称加密，存在最大的问题就是，密钥的存储与分发。由于对称加密算法加密，解密过程使用同一个密钥，因此，密钥在传输过程中如果被拦截，那么等同于明文传输。

如果只是简单的使用非对称加密，由于公钥是公开的，如果服务器返回的加密内容被黑客拦截，那么黑客可以直接使用公钥进行解密，获取其中的内容。

因此HTTPS将对称加密与非对称加密结合起来，先在握手阶段采用非对称加密算法传输对话密钥，确定对话密钥后，采用对称加密算法加密会话内容。

而且，由于对称加密算法比非对称加密要快， 所以，采用 '非对称加密+对称加密' 这种混合模式比 '非对称加密+非对称加密' 这种模式效率也高。可以说是在安全性以及性能方面的综合考虑。

### 对于证书的验证，客户端是如何做的？
![证书验证流程](/pic/https/证书验证流程.svg)

首先，服务器端需要先向CA机构申请证书，申请证书的时候，服务器向CA机构提供服务器的公钥，CA机构用自己的CA私钥对服务器的公钥进行签名，生成数字摘要，然后将服务器公钥和数字签名打进证书。

客户端从服务器拿到证书后，根据证书上的CA签发机构，从内置的根证书里找到对应的CA机构公钥，用此公钥解开数字签名，得到摘要，根据此验证证书的合法性。

### fiddler/charles等可实现对https请求的拦截，怎么实现的？

首先，fiddler/charles要实现对https的拦截，首先要做的就是，在客户端安装fiddler/charles自己导出的证书，并信任该证书。

原理就是，fiddler/charles在客户端和服务器间充当了中间人的角色，在客户端面前假装是https服务器，在真正的https服务器前假装是客户端。

流程如下图：

![fiddler拦截https示意图](/pic/https/fiddler拦截https示意图.svg)

1. fiddler/charles拦截客户端发送给https服务器的握手请求，并伪装成客户端向服务器发送请求进行握手
2. 服务器发回响应，fiddler/charles获取到服务器的CA证书，用根证书公钥进行解密，验证服务器数字签名，获取到服务器CA证书的公钥。然后fiddler/charles伪造自己的CA证书，冒充服务器证书传递给客户端浏览器。
3. 与普通过程中客户端操作相同，客户端根据返回的数据进行证书校验，校验时由于已提前导入了fiddler/charles中导出的根证书，所以这里证书校验会通过。通过校验后，客户端生成 pre_master，用fiddler/charles伪造的证书公钥加密，并生成https通信用的对称密钥 enc_key。
4. fiddler/charles拦截到密文，用自己伪造证书的私钥解开，得到enc_key，并将该密钥使用服务器证书中的公钥加密并传递给真正的https服务器。
5. https服务器收到密文后，利用自己的私钥解开该对称加密密钥，建立信任，握手完成，并使用对称密钥加密消息，开始通信。
6. fiddler/charles拦截到服务器发送的密文，用对称密钥解开，获得密文，再次加密，发送给客户端。
7. 同样，客户端向服务器发送消息时，用对称密钥加密，被fiddler/charles拦截，解密获得明文，再次加密，发送给服务器。

之后的整个过程，fiddler/charles都将会持有对称加密的密钥，因此对于客户端和服务器来说，它们相对于是透明的，整个过程是无感知的。
整个过程的关键在于，客户端安装并信任了fiddler/charles导出的根证书，使得fiddler/charles可以在客户端面前伪装成服务器，在服务器面前伪装成客户端，所以才能拦截https加密数据。