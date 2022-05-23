# 把远程服务器代理到本地

众所周知的原因，国内的服务器无法/难以访问一些外网资源，比如说 ~~Github~~ 等。

实验室的服务器需要挂VPN才能访问，通过VPN，服务器和我的笔记本处于同一个内网，可以通过ip直接连接。

正好我的笔记本上安装有能够访问外网的商业VPN软件，而且很可惜其限制了最多两台设备访问（分配给了手机和电脑），而且没有Linux版本可用。

所以我选择将服务器的网络代理到我的笔记本的端口，并且笔记本上使用[Proxyman](https://github.com/ProxymanApp/Proxyman)来代理这一个端口，这样网络的拓扑结构就如下：

```text
lab server => client => proxyman => bussiness VPN => Github
```

可以愉快的使用 GitHub 了。

```shell
#!/bin/sh
port=9909
ip=`echo $SSH_CLIENT | awk '{print $1}'`

export http_proxy=http://$ip:$port
export https_proxy=http://$ip:$port

curl http://ip-api.com/json/\?lang\=zh-CN

echo
echo export http_proxy=http://$ip:$port
echo export https_proxy=http://$ip:$port
echo
```
