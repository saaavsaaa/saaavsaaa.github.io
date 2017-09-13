

![Image](/ppp/NoSuchMethodError.png)

Jhat查看
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
这些静态成员有些意思，在正常的环境中是没有这些的，根据这些静态长远，从线上的代码中发现了一个类：
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
这个类里的静态成员和上面截图的完全对的上，这个方法和报错也对的上，除了包不一样，其他的都更像是自定义的这个，回头再看报错处的代码：
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
=====

使用Jvisualvm查看
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
