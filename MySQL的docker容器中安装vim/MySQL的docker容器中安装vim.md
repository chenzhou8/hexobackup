如何在MySQL的docker容器中安装vim以及其他的工具？

先进入MySQL容器：

```bash
docker exec -it <mysql容器id> bash
```

配置网易的镜像源

```bash
mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" > /etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
apt-get update
```

安装vim编辑器

```bash
apt-get install vim
```

