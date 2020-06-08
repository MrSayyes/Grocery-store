**1、git工具下载**

链接：https://gitforwindows.org/

**2、一路默认安装，安装完成后在桌面右击启动git bash**

![image](https://user-images.githubusercontent.com/19297162/68990366-3f156880-088d-11ea-9134-4468e6356a2a.png)

**3、设置用户名和邮箱**

```
$ git config --global user.name "your_name"
$ git config --global user.email "your_email@example.com"
```
**4、生成ssh公钥**

在git bash操作界面执行 ssh-keygen -t ras，一直回车即可

**5、通过git bash操作去c盘个人目录下面找到.ssh文件夹，里面有如下两个文件**

![image](https://user-images.githubusercontent.com/19297162/68990463-45581480-088e-11ea-8521-eb3834ea3d88.png)

**6、通过cat id_rsa.pub输出内容，并复制内容**

![image](https://user-images.githubusercontent.com/19297162/68990507-e646cf80-088e-11ea-81c8-d4f55523b4de.png)

**7、在gitHub设置里面找到ssh-key，并将第6点复制的内容粘贴进去保存即可**

![image](https://user-images.githubusercontent.com/19297162/68990544-6e2cd980-088f-11ea-9615-45ae6571c1b9.png)

**8、使用gitbash测试是否成功**

指令：$ ssh -T git@github.com

![image](https://user-images.githubusercontent.com/19297162/68990577-d8de1500-088f-11ea-9518-6c493c0c4e4a.png)

如果遇到yes/no的选项，输入yes，看最后结果是否是successfully

**9、补充乱码设置**

git bash指令设置：

git config --global core.quotepath false          # 显示 status 编码

git config --global gui.encoding utf-8            # 图形界面编码

git config --global i18n.commit.encoding utf-8    # 提交信息编码

git config --global i18n.logoutputencoding utf-8  # 输出 log 编码

在git安装目录的etc文件夹下找到vimrc文件，添加：

set fileencoding=gb18030 

set fileencodings=utf-8,gb18030,utf-16,big5

**10、git访问加速配置**

1）在https://www.ipaddress.com/ 上面搜索github.global.ssl.fastly.net和github.com和assets-cdn.github.com，找到对应的ip，然后配置hosts文件：

199.232.69.194 github.global.ssl.fastly.net
140.82.114.4 github.com
185.199.108.153 assets-cdn.github.com
185.199.109.153 assets-cdn.github.com
185.199.110.153 assets-cdn.github.com
185.199.111.153 assets-cdn.github.com

2）Winodws系统的做法：打开CMD，输入`ipconfig /flushdns`