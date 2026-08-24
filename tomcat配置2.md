下一步把 Tomcat配置成系统服务，确保服务器重启后自动运行。因为现在是手动启动的，需要先停止，再交给 systemd管理。

## 1. 停止手动启动的Tomcat

```bash
sudo -u tomcat /opt/tomcat/bin/shutdown.sh
```

等待几秒后检查：

```bash
ss -lntp | grep 8080
```

没有输出表示已经停止。

## 2. 创建系统服务

```bash
sudo vi /etc/systemd/system/tomcat.service
```

写入：

```ini
[Unit]
Description=Apache Tomcat 9
After=network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment="JAVA_HOME=/opt/java/jdk1.8.0_352"
Environment="JRE_HOME=/opt/java/jdk1.8.0_352/jre"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

保存后执行：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now tomcat
```

## 3. 验证系统服务

```bash
sudo systemctl status tomcat
```

应该显示：

```text
active (running)
```

检查开机启动：

```bash
sudo systemctl is-enabled tomcat
```

应该显示：

```text
enabled
```

内部访问测试：

```bash
curl http://127.0.0.1:8080
```

以后管理 Tomcat统一使用：

```bash
sudo systemctl start tomcat
sudo systemctl stop tomcat
sudo systemctl restart tomcat
sudo systemctl status tomcat
```

不要再混用 `startup.sh`手动启动，否则可能出现两个启动方式互相冲突。

确认 systemd状态为 `active (running)`且 `curl`正常后，下一步就是开放内网8080端口，并把负反馈看板部署到 `/opt/tomcat/webapps/ROOT/`。
