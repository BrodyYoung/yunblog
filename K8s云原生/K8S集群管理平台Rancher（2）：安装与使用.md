
Rancher 是一个容器管理平台。Rancher 简化了使用 Kubernetes 的流程。
下面记录一下手动安装Rancher的步骤
![在这里插入图片描述](https://img-blog.csdnimg.cn/a9230fec83a54d1f91911318fc87de29.png)

## 一、docker安装rancher
拉取rancher镜像
```bash
docker  pull  rancher/rancher
```
运行rancher容器
```bash
sudo docker run -d --restart=always  \
-v /mydata/docker/rancher_data:/var/lib/rancher/ \
-p 80:80 -p 443:443 --privileged \
--name=rancher rancher/rancher
```


访问rancher页面
http://服务器IP地址:80
80端口可以省略不写


## 二、使用rancher
访问rancher地址，会有安全提示，点击“高级”--->“继续前往”
![在这里插入图片描述](https://img-blog.csdnimg.cn/01c7cbd8ed884eb19ec4b74f589e0e4e.png)
进入到欢迎界面，提示在ssh中输入

```bash
查看Rancher容器ID
docker  ps    
获得登录密码，需要把container-id修改为Rancher容器id
docker logs  container-id  2>&1 | grep "Bootstrap Password:"
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/43a4a7c2a87f4c62be92674105304a16.png)
按照提示，查看一下初始密码是多少
![在这里插入图片描述](https://img-blog.csdnimg.cn/5e0083c7ab4842c89a87770f39d26200.png)


复制随机密码到输入框，登录进去。
到这一步可以设置自定义密码。
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4c7ea7e32654befb5d8027f81b426ef.png)
进入Rancher首页，至此Rancher就安装成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/3172b4b80d1340edadbd0279fcf5cfff.png)

可以切换中英文
![在这里插入图片描述](https://img-blog.csdnimg.cn/985d15bf27ec49c0965c093b5177f433.png)
导入集群或通过Rancher创建集群
![在这里插入图片描述](https://img-blog.csdnimg.cn/a7966417409444dca6f88b90b64cde13.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/be6691f4c8a24296bd0e2f711c080af8.png)

Rancher会提供一个local集群，使用的是k3s技术。

K3s是由Rancher开发的轻量级 Kubernetes。安装简单，内存只有一半，k8s(kubernetes)有10个字母，10的一半为5，所以叫做k3s。
k3s 旨在成为完全兼容的 Kubernetes 发行版，相比 k8s 主要更改如下：

 - 旧的、Alpha 版本的、非默认功能都已经删除。
 - 删除了大多数内部云提供商和存储插件，可以用插件替换。
 - 新增 SQLite3 作为默认存储机制，etcd3 仍然有效，但是不再是默认项。
 - 封装在简单的启动器中，可以处理大量 LTS 复杂性和选项。
 - 最小化到没有操作系统依赖，只需要一个内核和 cgroup 挂载。
 
 k3s 工作原理：
![在这里插入图片描述](https://img-blog.csdnimg.cn/43e51f19940b4ebf9388164d95afed13.png)


不想自定义创建k8s集群的可以使用local进行学习。
![在这里插入图片描述](https://img-blog.csdnimg.cn/3cd7df70fe45489697a1c142cb9919f2.png)



查看Pod容器集列表。
点击右上角命令行图标，在底部会出现k8s命令行工具，可以输入命令行进行操作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4739cf7bec854bfdb6cfdfaf6812d40a.png)

