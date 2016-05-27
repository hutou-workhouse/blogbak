title: JS编码和Java编码区别
date: 2016-01-27 19:01:33
categories:
-	技术随笔

tags:
-	编码
-	杂项

---

Javascript编码与Java编码区别

<!--more-->

最近一个朋友问他一个问他，他们在前端使用JS进行签名，后端验签名。签名的方式是前端使用***encodeURIComponent***函数；后端使用***java.net.URLEncoder.encode***，比较两种方法形成的字符串是否相等来判断。但是，有些特殊字符结果不一致

java.net.URLEncoder.encode方法，查看文档有如下的说明

>1. The alphanumeric characters "a" through "z", "A" through "Z" and "0" through "9" remain the same.
>2. **The special characters ".", "-", "*", and "_" remain the same. **
>3. The space character " " is converted into a plus sign "+".
>4. All other characters are unsafe and are first converted into one or more bytes using some encoding scheme. Then each byte is represented by the 3-character string "%xy", where xy is the two-digit hexadecimal representation of the byte. The recommended encoding scheme to use is UTF-8. However, for compatibility reasons, if an encoding is not specified, then the default encoding of the platform is used.

结论：上述描述2的一些特殊字符不转意

javsacript函数***encodeURIComponent***：
> 不会进行编码的字符有71个：!， '，(，)，*，-，.，_，~，0-9，a-z，A-Z

javsacript函数***encodeURI***：
>不会进行编码的字符有82个 ：!，#，$，&，'，(，)，*，+，,，-，.，/，:，;，=，?，@，_，~，0-9，a-z，A-Z

综上所述，最好还是使用标准的sha1，md5等hash算法比较稳妥