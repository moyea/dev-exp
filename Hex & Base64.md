# Hex & Base64

Base64和Hex之间的区别实际上只是字节的表示方式。Hex也可以称之为"Base16"。

Hex将一个字节编码为两个十六进制字符。eg: 0000 0000 => 00

Base64将三个字节编码为四个Base64字符。

> 为什么base64要特别指定为3个字节？
>
> 3个字节 = 3 * 8位 =24位
>
> base64代表数字 0~63，用二进制表示就是 000000<sub>二进制</sub> (0) ~ 111111<sub>二进制</sub>（63），也就是每个base64字符代表6位二进制数
>
> 因此，24位(3个字节) / 6位(base64字符) = 4个base64字符 
>
> 也就是每个Base64字符(1个字节8位)编码6位实际数据。通过计算可知Base64相比于原数据增加了33%(有时候可能在尾部有一些=号之类的输出)

可以看到，base64相比于Hex更加高效。假设使用UTF-8编码XML文档，则100k文件编码为Hex字符则需要200K，而Base64中只需要133K。

为什么不用只增加了14%大小 base128(一个字节8位代表实际的7位数据)来编码？因为base64是我们可以找到最好的，ASCII码中前128个字符并不是都是可以打印的，还有许多控制字符比如 `NULL` 之类的。

当然也可以创建其他的编码方式，比如base81，然后实现自己的解码算法。但是base64的优点在于编解码很清晰、简单。base64编码: 读3个字节，输出4个字节; base64解码: 读4个字节输出3个字节。所以base64编码很流行。

以下Demo取自：[维基百科 Ascii85](https://en.wikipedia.org/wiki/Ascii85) 

```
// utf-8 原文
Man is distinguished, not only by his reason, but by this singular passion from other animals, which is a lust of the mind, that by a perseverance of delight in the continued and indefatigable generation of knowledge, exceeds the short vehemence of any carnal pleasure.

// base64
TWFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5IGhpcyByZWFzb24sIGJ1dCBieSB0aGlzIHNpbmd1bGFyIHBhc3Npb24gZnJvbSBvdGhlciBhbmltYWxzLCB3aGljaCBpcyBhIGx1c3Qgb2YgdGhlIG1pbmQsIHRoYXQgYnkgYSBwZXJzZXZlcmFuY2Ugb2YgZGVsaWdodCBpbiB0aGUgY29udGludWVkIGFuZCBpbmRlZmF0aWdhYmxlIGdlbmVyYXRpb24gb2Yga25vd2xlZGdlLCBleGNlZWRzIHRoZSBzaG9ydCB2ZWhlbWVuY2Ugb2YgYW55IGNhcm5hbCBwbGVhc3VyZS4=

// base85
<~9jqo^BlbD-BleB1DJ+*+F(f,q/0JhKF<GL>Cj@.4Gp$d7F!,L7@<6@)/0JDEF<G%<+EV:2F!,
O<DJ+*.@<*K0@<6L(Df-\0Ec5e;DffZ(EZee.Bl.9pF"AGXBPCsi+DGm>@3BB/F*&OCAfu2/AKY
i(DIb:@FD,*)+C]U=@3BN#EcYf8ATD3s@q?d$AftVqCh[NqF<G:8+EV:.+Cf>-FD5W8ARlolDIa
l(DId<j@<?3r@:F%a+D58'ATD4$Bl@l3De:,-DJs`8ARoFb/0JMK@qB4^F!,R<AKZ&-DfTqBG%G
>uD.RTpAKYo'+CT/5+Cei#DII?(E,9)oF*2M7/c~>
```

由于计算机本质上是大规模集成电路(IC, Integrated Circuit), 电路只有开/关两种状态，也就决定了计算机信息处理只能用0或1来表示，也就是二进制。

计算机的最小集成单位是`位`，也就是`比特(bit)`。为了方便使用，我们将8位二进制数称为一个字节。所以一个字节能够表示 00000000<sub>二进制</sub> ~ 1111111<sub>二进制</sub>

> 为什么1个字节等于8位呢？因为8位能够涵盖所有ASCII字符编码。

字节是最基本的计量单位，位是最小单位。

用字节处理数据时，如果数字小于存储数据的字节数 ( = 二进制的位数)，那么高位就用 0 填补，高位和数学的数字表示是一样的，**左侧表示高位，右侧表示低位。**比如 这个六位数用二进制数来表示就是 `100111`，只有6位，高位需要用 0 填充，填充完后是 `00100111`，占一个字节，如果用 16 位表示 就是 `0000 0000 0010 0111`占用两个字节。

我们一般口述的 32 位和 64位的计算机一般就指的是处理位数，32 位一次可以表示 4个字节，64位一次可以表示8个字节的二进制数。



参考链接：

- [关于二进制世界的秘密](https://mp.weixin.qq.com/s?__biz=MzkwMDE1MzkwNQ==&mid=2247496074&idx=1&sn=3ddc367ac21ddf3156ef643e2f70356c&source=41#wechat_redirect)

- [Base64 vs Hex](https://stackoverflow.com/questions/3183841/base64-vs-hex-for-sending-binary-content-over-the-internet-in-xml-doc)

