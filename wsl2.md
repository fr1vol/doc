## WSL2 Linux环境相关

[安装桌面](###安装桌面)
[安装使用python3 venv](###安装使用venv)
### 安装桌面

环境: **WSL2 ubuntu18**

1. 安装xrdp
```
sudo apt-get update 
sudo apt-get install xrdp 
```

2. 安装MATE桌面环境
```
sudo apt-get install mate-core mate-desktop-environment mate-notification-daemon
```
3. 配置xrdp默认启动环境
```
 sudo sed -i.bak '/fi/a #xrdp multiple users configuration \n mate-session \n' /etc/xrdp/startwm.sh
```
4. 开启端口，这里是13333
```
sudo ufw allow 13333/tcp
```
5. 重启xrdp服务
```
sudo /etc/init.d/xrdp restart
```
6. win10打开远程桌面连接

---


### 安装使用venv

环境: **wsl2 ubuntu18**
python版本：**3.6.9**

1. 安装venv
```
sudo apt install venv
```
2. 创建虚拟环境，env文件夹
```
python -m venv env
```
3. 激活环境
```
source env/bin/activate
```
4. 关闭环境
```
deactive
```