如果文件完整名称是：

```text
jdk-8u352-linux-x64.tar.gz
```

它不是传统安装程序，而是解压后配置环境变量。下面按这个格式操作。如果文件后缀是 `.rpm`，不要执行这些步骤，把完整文件名发给我。

## 1. 从Windows上传到服务器

在自己电脑的 PowerShell 中执行：

```powershell
scp "你的文件目录\jdk-8u352-linux-x64.tar.gz" 用户名@服务器IP:/tmp/
```

例如：

```powershell
scp "D:\software\jdk-8u352-linux-x64.tar.gz" user01@10.20.30.40:/tmp/
```

然后登录服务器：

```powershell
ssh user01@10.20.30.40
```

## 2. 确认文件完整

服务器执行：

```bash
ls -lh /tmp/jdk-8u352-linux-x64.tar.gz
```

检查文件类型：

```bash
file /tmp/jdk-8u352-linux-x64.tar.gz
```

应该包含类似：

```text
gzip compressed data
```

检查压缩包能否正常读取：

```bash
tar -tzf /tmp/jdk-8u352-linux-x64.tar.gz | head
```

如果没有报错再继续。

## 3. 创建Java安装目录

```bash
sudo mkdir -p /opt/java
```

解压：

```bash
sudo tar -xzf /tmp/jdk-8u352-linux-x64.tar.gz -C /opt/java
```

查看解压后的目录：

```bash
ls -l /opt/java
```

正常情况下会出现：

```text
jdk1.8.0_352
```

验证Java：

```bash
/opt/java/jdk1.8.0_352/bin/java -version
```

应该看到：

```text
java version "1.8.0_352"
```

## 4. 配置系统环境变量

创建配置文件：

```bash
sudo vi /etc/profile.d/java.sh
```

按 `i`进入编辑，写入：

```bash
export JAVA_HOME=/opt/java/jdk1.8.0_352
export JRE_HOME=/opt/java/jdk1.8.0_352/jre
export PATH=$JAVA_HOME/bin:$PATH
```

保存：

1. 按 `Esc`
2. 输入 `:wq`
3. 回车

设置文件权限：

```bash
sudo chmod 644 /etc/profile.d/java.sh
```

让当前终端立即生效：

```bash
source /etc/profile.d/java.sh
```

## 5. 验证安装

```bash
echo $JAVA_HOME
```

应该输出：

```text
/opt/java/jdk1.8.0_352
```

然后执行：

```bash
java -version
javac -version
```

应该分别显示 Java和编译器的 `1.8.0_352`版本。

## 6. 暂时不要删除安装包

先保留：

```text
/tmp/jdk-8u352-linux-x64.tar.gz
```

等 Tomcat成功启动后再决定是否清理。

完成以上步骤后，把下面三条命令的输出发给我：

```bash
echo $JAVA_HOME
java -version
javac -version
```

确认 Java正常后，我们再继续配置 Tomcat 9.0.87使用这个 JDK。



需要。`/tmp`只是临时存放安装包的位置，不适合长期运行 Java，因为系统可能定期清理 `/tmp`。

正确关系是：

```text
/tmp/jdk-8u352-linux-x64.tar.gz    临时安装包
/opt/java/jdk1.8.0_352/            正式安装目录
```

你不用手动进入其他目录，直接执行：

```bash
sudo mkdir -p /opt/java
```

然后把压缩包解压到正式目录：

```bash
sudo tar -xzf /tmp/jdk-8u352-linux-x64.tar.gz -C /opt/java
```

检查：

```bash
ls -l /opt/java
```

正常应该看到：

```text
jdk1.8.0_352
```

测试：

```bash
/opt/java/jdk1.8.0_352/bin/java -version
```

所以：

- 安装包暂时放在 `/tmp`。
- Java程序放在 `/opt/java`。
- 不需要使用 `cd`切换目录。
- 等 Java和 Tomcat都正常运行后，再删除 `/tmp`中的安装包。

另外注意：Linux里的 `/tmp`是从根目录开始的系统临时目录；`tmp`没有开头的 `/`时，表示当前目录下面名为 `tmp`的文件夹，两者不是一回事。
