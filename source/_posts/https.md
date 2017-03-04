---
title: https笔记
---

# https

Https是就是套在SSL/TLS内的http，也就是安全的http。何为安全？一个安全的网络通信环境要解决3个问题：

```
1. 通信内容的保密
2. 通信双方身份的真实
3. 通信内容的完整
```
<!--more-->

## **Https 是利用非对对称加密算法**

> 对称加密算法在加密和解密时使用的是同一个秘钥；而非对称加密算法需要两个密钥来进行加密和解密，这两个秘钥是公开密钥（public key，简称公钥）和私有密钥（private key，简称私钥）。你可能比较好奇非对称加密算法的原理，但是我这里不展开讲算法，有兴趣的同学可以自行搜索。  

如果小红用她的私钥加密的话，班上所有人都知道公钥，而公钥可以解私钥的加密，也意味着所有人都能解密小红的回应消息。聪明的你一定想到了解决方案：利用非对称加密算法加密出一个对称密钥给小红，小红用她的私钥读取对称密钥，然后你们就用这个对称密钥来做对称加密，然后就可以愉快地约约约了。

当然，https也是这么干的。

上面的例子的意思是：
私钥可以解读公钥要加的数据，私钥加密的数据可以由公钥来解读

小红自己有一个私钥，其余每个人都有一串公钥，所以我们利用对称密钥加密出一个密钥给小红，让小红用这个给进行消息的加密并反馈。

## **通信双方身份的真实**
为了防止掉包事件的发生

所有加密通信都要带上一本证，用来证明自己的身份。这本证是凤姐特意为班上所有单身狗做的，公钥就放在证书里面返回给纸条的发送者，证书里面除了公钥还有学号、人名、甚至星座身高三围等各种信息。证书上盖了一个大大的鉴定章，这是凤姐独有的章，表示证上的信息真实性由凤姐保证，看到这个章就可以认为对方是个真·单身狗。

通过这些信息你就可以知道对方是小红还是如花了，这就是证书机制。

证书上的公章也是非对称加密过的，加密方式跟上面提到的刚好相反：用凤姐的私钥加密，用凤姐公钥就可以解密，这样就可以鉴定证书的真伪。这个公章就是证书的数字签名，具体来说就是先将证书用哈希算法提取摘要，然后对摘要进行加密的过程。另外你也可以直接拿着证书去找凤姐，凤姐就会帮你验证证书的有效性。（证书是有期限的，所以即使是真证书也会可能过期，需要注意）

这个机制看起来相当完善，但是我们要以怀疑一切的态度去做安全机制，凤姐保证的东西是可信任的了。

## **通信内容的完整**

为了防止通信内容的完整性与一致性

这种篡改通信内容的场景相信大家都深有体会，我们访问一些站点的时候无缘无故就出现了运营商的广告，这都是运营商给加的！！所以内容的完整性也需要保证，这比较简单：先用哈希算法提取内容摘要，然后对摘要进行加密生成数字签名，验证数字签名就可以判断出通信内容的完整性了。

以上就是https用到技术的简化版，一个http通信流程如下：

![https流程](https://github.com/ws456999/ws456999.github.io/blob/master/https.png?raw=true)

大体步骤：
```
1. 客户端发送Client Hello报文开始SSL通信，报文中包含SSL版本、可用算法列表、密钥长度等。
2. 服务器支持SSL通信时，会以Server Hello报文作为应答，报文中同样包括SSL版本以及加密算法配置，也就是协商加解密算法。
3. 然后服务器会发送Certificate报文，也就是将证书发送给客户端。
4. 客户端发送Client Key Exchange报文，使用3中的证书公钥加密Pre-master secret随机密码串，后续就以这个密码来做对称加密进行通信。
5. 服务器使用私钥解密成功后返回一个响应提示SSL通信环境已经搭建好了。
6. 然后就是常规的http c/s通信。
```
根据前文所述，在步骤3和步骤6都会使用摘要和签名算法来保证传递的证书和通信内容不被篡改。通过这个流程可以看出，https的核心在于加密，尤其是非对称加密算法被多次使用来传送关键信息。

理解了加密，认识到网络的透明性，抱着怀疑一切的态度，理解https这套体系就变得简单了。
