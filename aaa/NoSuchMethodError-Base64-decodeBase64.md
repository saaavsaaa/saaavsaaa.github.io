
有台服务器应用里报错，可稳定重现，但只能在这一台机器上重现：
-----
![Image](/ppp/NoSuchMethodError.png)
-----
先对比了一下，正常和异常的服务器，环境完全一样，其实都没必要对比，因为都是同样配置出来的虚机，连各种小版本都完全一样。
-----
Jhat查看了一下dump：
=====
这是正常的：
------
![Image](/ppp/213Base64.png)
------
![Image](/ppp/213staticdata.png)
-----
这是出错的：
-----
![Image](/ppp/214Base64.png)
-----
![Image](/ppp/214staticdata.png)
-----
这些静态成员有些意思，在正常的环境中是没有这些的，根据这些静态成员，从线上的代码中发现了一个类：
-----
```markdown
package com.xxx.xxx.xxx.xxx.reapel.client.utils;
import java.io.UnsupportedEncodingException;
public class Base64  {
    . . .
    public static byte[] encodeBase64(byte[] binaryData) {
            . . .
    }
    . . .
}
```
-----
这个类里的静态成员和上面截图的完全对的上，这个方法和报错也对的上，除了包不一样，其他的都更像是自定义的这个，回头再看报错处的代码：
-----
![Image](/ppp/base64invoke.png)
-----
我现在的想法是，我是不是可以怀疑这是ClassLoader或者不知道哪的bug？先找找证据
-----
我还在异常的服务器上javap -c RSA了一下，很正常：
-----
```markdown
  public static byte[] decryptBASE64(java.lang.String) throws java.lang.Exception;
    Code:
       0: aload_0
       1: invokestatic  #33                 // Method org/apache/commons/codec/binary/Base64.decodeBase64:(Ljava/lang/String;)[B
       4: areturn
```
-----
=====

把dump下载下来用Jvisualvm查看
=====
正常的：
-----
![Image](/ppp/213instance.png)
-----
有错的：
-----
![Image](/ppp/214instance.png)
=====

先怀疑包的问题，然而用有错服务器上的包直接部署到其他机器上没有问题，拉下来解压看了，都不缺，也都一样
