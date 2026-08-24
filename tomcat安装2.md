Tomcat的 `.tar.gz`压缩包和JDK类似：先复制到服务器的 `/tmp`，再用命令解压到 `/opt/tomcat`。不建议在 Windows解压后逐个复制，因为可能破坏 Linux文件权限和脚本执行权限。

假设文件名是：

```text
apache-tomcat-9.0.87.tar.gz
```

## 1. 从Windows上传

在 PowerShell执行：

```powershell
scp "D:\software\apache-tomcat-9.0.87.tar.gz" 用户名@服务器IP:/tmp/
```

## 2. 登录服务器并检查压缩包

```powershell
ssh 用户名@服务器IP
```

服务器中执行：

```bash
ls -lh /tmp/apache-tomcat-9.0.87.tar.gz
```

```bash
file /tmp/apache-tomcat-9.0.87.tar.gz
```

检查压缩包是否完整：

```bash
tar -tzf /tmp/apache-tomcat-9.0.87.tar.gz | head
```

没有报错再继续。

## 3. 检查目标目录

```bash
sudo ls -la /opt/tomcat
```

如果提示目录不存在，可以继续；如果目录已经存在且里面有文件，先停止，不要覆盖，可能已经安装过 Tomcat。

创建目录：

```bash
sudo mkdir -p /opt/tomcat
```

## 4. 解压Tomcat

```bash
sudo tar -xzf /tmp/apache-tomcat-9.0.87.tar.gz -C /opt/tomcat --strip-components=1
```

检查：

```bash
ls -l /opt/tomcat
```

正常会看到：

```text
bin
conf
lib
logs
temp
webapps
work
```

给脚本增加执行权限：

```bash
sudo chmod +x /opt/tomcat/bin/*.sh
```

## 5. 验证Java和Tomcat兼容

假设JDK安装在：

```text
/opt/java/jdk1.8.0_352
```

执行：

```bash
JAVA_HOME=/opt/java/jdk1.8.0_352 /opt/tomcat/bin/version.sh
```

正常应该显示：

```text
Server version: Apache Tomcat/9.0.87
JVM Version: 1.8.0_352
```

这就说明 Tomcat文件安装完成了。

不过，`.tar.gz`解压后只是“程序文件就位”，还没有：

- 配置专用运行用户
- 配置系统服务
- 配置开机自动启动
- 开放8080端口
- 部署你的HTML

确认 `version.sh`输出正常后，再进行启动和系统服务配置。
