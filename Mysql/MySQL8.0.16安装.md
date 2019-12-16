## MySQL 8.0.16 安装

参考：https://www.cnblogs.com/zyongzhi/p/10063149.html


### <font color="#159957">目录</font>

&emsp;**一、下载地址**

&emsp;**二、下载目录**

&emsp;**三、将安装包放入待解压文件夹**

&emsp;**四、解压**

&emsp;**五、安装 Common**

&emsp;**六、安装 libs**

&emsp;**七、安装 client**

&emsp;**八、额外需要注意的点**

---
**<font color="#159957">一、下载地址</font>**

官网

![](assets/markdown-img-paste-20190625143650817.png)

网易开源镜像站

![](assets/markdown-img-paste-20190625143605487.png)

**<font color="#159957">二、下载目录</font>**

![](assets/markdown-img-paste-20190625143930517.png)

**<font color="#159957">三、创建一个文件夹将此安装包放入其中，准备解压</font>**

![](assets/markdown-img-paste-2019062514424464.png)

**<font color="#159957">四、解压</font>**

![](assets/markdown-img-paste-20190625145130801.png)

**<font color="#159957">五、安装 Common（必须按照下面的顺序）</font>**

![](assets/markdown-img-paste-2019062515030379.png)

**<font color="#159957">六、安装 libs</font>**

![](assets/markdown-img-paste-20190625150521273.png)

**<font color="#159957">七、安装 client</font>**

![](assets/markdown-img-paste-20190625150740520.png)

**<font color="#159957">七、安装 server</font>**

![](assets/markdown-img-paste-20190625151154234.png)

**<font color="#159957">八、初始化数据库</font>**

**<font color="#159957">九、目录授权，否则启动失败</font>**

**<font color="#159957">十、启动服务</font>**

**<font color="#159957">十一、停止服务</font>**

![](assets/markdown-img-paste-20190625153425472.png)

**<font color="#159957">十二、找到初始密码，使用客户端登陆</font>**

![](assets/markdown-img-paste-2019062515510042.png)

**<font color="#159957">十三、set global validate_password_policy=0; 设置密码规则遇到错误</font>**

![](assets/markdown-img-paste-20190625185718508.png)

https://blog.csdn.net/zd147896325/article/details/82427107

设置完密码之后，使用 Navicat 进行连接报错：

![](assets/markdown-img-paste-2019062518302826.png)

解决方法:

![](assets/markdown-img-paste-20190625183526342.png)

![](assets/markdown-img-paste-20190625183947795.png)
