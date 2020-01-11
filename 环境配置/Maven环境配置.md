# Maven对JDK要求

Maven 3.3 要求 JDK 1.7 或以上

Maven 3.2 要求 JDK 1.6 或以上

Maven 3.0/3.1 要求 JDK 1.5 或以上

链接：[JAVA安装配置](https://github.com/MrSayyes/Grocery-store/blob/master/环境配置/Java开发环境配置（windows版）.md)

# Maven下载

Maven 下载地址：http://maven.apache.org/download.cgi

Windows平台包：Binary zip archive

![image](https://user-images.githubusercontent.com/19297162/72205834-63bf4280-34c2-11ea-874d-458d235108cc.png)

# Maven环境变量

### 1、新增系统变量：MAVEN_HOME

![image](https://user-images.githubusercontent.com/19297162/72205792-f9a69d80-34c1-11ea-8e91-a0911bff7038.png)

### 2、编辑Path变量，添加%MAVEN_HOME%\bin

![image](https://user-images.githubusercontent.com/19297162/72205916-3a52e680-34c3-11ea-9efa-b18e21f74057.png)

### 3、检查Maven配置

cmd进入打开控制台，进入Maven安装目录，使用mvn -v来检查，如果出现Maven版本与配置版本一致则表示环境配置完成。

![image](https://user-images.githubusercontent.com/19297162/72205991-1a6ff280-34c4-11ea-9e52-3d46470a7be6.png)