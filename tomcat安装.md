下面按“openEuler虚拟机、没有指定旧版本、部署静态看板”说明。建议使用 Java 17 + Tomcat 10.1；Tomcat 10.1要求 Java 11或以上，Java 17较稳妥。[Tomcat版本兼容说明](https://tomcat.apache.org/whichversion.html)

所有 Linux 命令都在服务器终端执行。

## 一、登录服务器

在自己电脑的 PowerShell 中：

```powershell
ssh 用户名@服务器IP
```

登录后确认系统：

```bash
cat /etc/os-release
```

看到 `openEuler` 再继续。

## 二、安装Java

先查看是否已经安装：

```bash
java -version
```

如果没有 Java，安装 Java 17：

```bash
sudo dnf clean all
sudo dnf makecache
sudo dnf install java-17-openjdk-devel -y
```

验证：

```bash
java -version
javac -version
```

查看 Java 安装目录：

```bash
dirname "$(dirname "$(readlink -f "$(command -v java)")")"
```

记住输出结果，后面配置 `JAVA_HOME` 时需要使用。openEuler官方也建议通过 `dnf` 安装 OpenJDK。[openEuler JDK安装文档](https://docs.openeuler.org/zh/docs/25.03/server/development/application_dev/preparations-for-development-environment.html)

## 三、下载Tomcat

截至目前，Tomcat 10.1是受支持的稳定系列。正式服务器应从 [Apache Tomcat官方下载页面](https://tomcat.apache.org/download-10) 获取最新的 10.1.x 版本，不要从不明网站下载。

如果服务器可以联网，可以执行：

```bash
cd /tmp
curl -O https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.57/bin/apache-tomcat-10.1.57.tar.gz
curl -O https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.57/bin/apache-tomcat-10.1.57.tar.gz.sha512
sha512sum -c apache-tomcat-10.1.57.tar.gz.sha512
```

看到：

```text
apache-tomcat-10.1.57.tar.gz: OK
```

才能继续。

如果服务器不能联网，就在自己电脑下载压缩包，再上传：

```powershell
scp "apache-tomcat-10.1.57.tar.gz" 用户名@服务器IP:/tmp/
```

公司如果指定了其他 Tomcat 版本，应以公司要求为准，不要自行选择。

## 四、创建专用账号

不要直接使用 root 运行 Tomcat：

```bash
sudo useradd --system --home-dir /opt/tomcat --shell /sbin/nologin tomcat
sudo mkdir -p /opt/tomcat
```

解压：

```bash
sudo tar -xzf /tmp/apache-tomcat-10.1.57.tar.gz -C /opt/tomcat --strip-components=1
sudo chown -R tomcat:tomcat /opt/tomcat
sudo chmod +x /opt/tomcat/bin/*.sh
```

## 五、先手动测试启动

```bash
sudo -u tomcat /opt/tomcat/bin/startup.sh
```

查看日志：

```bash
tail -n 50 /opt/tomcat/logs/catalina.out
```

在服务器内部测试：

```bash
curl http://127.0.0.1:8080
```

如果返回 Tomcat 页面，说明安装成功。

停止测试服务：

```bash
sudo -u tomcat /opt/tomcat/bin/shutdown.sh
```

## 六、配置成系统服务

先执行：

```bash
dirname "$(dirname "$(readlink -f "$(command -v java)")")"
```

假设输出是：

```text
/usr/lib/jvm/java-17-openjdk-17.0.x
```

创建服务配置：

```bash
sudo vi /etc/systemd/system/tomcat.service
```

按 `i` 进入编辑，然后填写：

```ini
[Unit]
Description=Apache Tomcat
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-17.0.x"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms256M -Xmx512M"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

注意：`JAVA_HOME`必须换成你服务器实际查询到的路径。

保存方式：

1. 按 `Esc`
2. 输入 `:wq`
3. 回车

加载并启动：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
sudo systemctl status tomcat
```

看到：

```text
active (running)
```

表示成功。

## 七、开放8080端口

先让运维确认允许开放，不要直接关闭防火墙。

如果允许：

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

然后在同一内网的电脑访问：

```text
http://服务器IP:8080
```

## 八、部署你的静态看板

在自己电脑上传：

```powershell
scp "C:\Code\Remote_work\0814\negative-feedback-dashboard.html" 用户名@服务器IP:/tmp/index.html
```

服务器执行：

```bash
sudo cp /opt/tomcat/webapps/ROOT/index.html /opt/tomcat/webapps/ROOT/index.html.bak
sudo cp /tmp/index.html /opt/tomcat/webapps/ROOT/index.html
sudo chown tomcat:tomcat /opt/tomcat/webapps/ROOT/index.html
```

访问：

```text
http://服务器IP:8080
```

应该就能看到看板。

## 常见问题

检查服务：

```bash
sudo systemctl status tomcat
```

查看日志：

```bash
tail -n 100 /opt/tomcat/logs/catalina.out
```

检查端口：

```bash
ss -lntp | grep 8080
```

服务器内部能访问、其他电脑打不开，一般是防火墙或公司网络策略问题，需要找运维开放8080端口。

这一步只完成“静态看板发布”。后续管理员上传 JSON 并让所有人共享数据，还要开发 Java后端接口和数据库。
