

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
