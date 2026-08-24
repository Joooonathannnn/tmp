说明 JDK和 Tomcat已经兼容。下一步先用专用账号启动测试，不急着配置开机启动和防火墙。

## 1. 创建Tomcat运行账号

先检查是否已经存在：

```bash
getent passwd tomcat
```

如果没有任何输出，执行：

```bash
sudo useradd -r -M -d /opt/tomcat -s /sbin/nologin tomcat
```

设置Tomcat目录归属：

```bash
sudo chown -R tomcat:tomcat /opt/tomcat
```

## 2. 配置Tomcat使用这个JDK

创建配置文件：

```bash
sudo vi /opt/tomcat/bin/setenv.sh
```

按 `i`进入编辑，写入：

```bash
export JAVA_HOME=/opt/java/jdk1.8.0_352
export JRE_HOME=/opt/java/jdk1.8.0_352/jre
export CATALINA_HOME=/opt/tomcat
export CATALINA_BASE=/opt/tomcat
export CATALINA_OPTS="-Xms256m -Xmx512m"
```

保存：

1. 按 `Esc`
2. 输入 `:wq`
3. 回车

设置权限：

```bash
sudo chown tomcat:tomcat /opt/tomcat/bin/setenv.sh
sudo chmod 750 /opt/tomcat/bin/setenv.sh
```

## 3. 启动Tomcat

```bash
sudo -u tomcat /opt/tomcat/bin/startup.sh
```

正常会看到类似：

```text
Tomcat started.
```

## 4. 查看日志

```bash
tail -n 50 /opt/tomcat/logs/catalina.out
```

正常日志中会出现：

```text
Server startup in ... milliseconds
```

## 5. 在服务器内部测试

```bash
curl http://127.0.0.1:8080
```

如果返回 Tomcat的 HTML内容，说明服务已经成功运行。

再检查端口：

```bash
ss -lntp | grep 8080
```

先完成这一步。确认 `curl`能够返回内容后，再配置 systemd开机启动、内网访问端口和看板部署。
