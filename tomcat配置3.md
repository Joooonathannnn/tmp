现在可以部署看板。建议使用独立的 `/feedback/`地址，不覆盖 Tomcat默认页面。

## 1. 从Windows上传HTML

在自己电脑的 PowerShell中执行：

```powershell
scp "C:\Code\Remote_work\0814\negative-feedback-dashboard.html" 用户名@服务器IP:/tmp/
```

## 2. 在服务器创建网站目录

登录服务器后执行：

```bash
sudo mkdir -p /opt/tomcat/webapps/feedback
```

复制并改名为 `index.html`：

```bash
sudo cp /tmp/negative-feedback-dashboard.html /opt/tomcat/webapps/feedback/index.html
```

设置权限：

```bash
sudo chown -R tomcat:tomcat /opt/tomcat/webapps/feedback
sudo chmod 640 /opt/tomcat/webapps/feedback/index.html
```

## 3. 在服务器内部验证

```bash
curl -I http://127.0.0.1:8080/feedback/
```

正常应看到：

```text
HTTP/1.1 200
```

一般不需要重启；如果没有加载，再执行：

```bash
sudo systemctl restart tomcat
```

然后重新测试。

## 4. 开放内网访问

先检查防火墙：

```bash
sudo firewall-cmd --state
```

如果显示 `running`，并且公司允许开放8080端口：

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

应该能看到：

```text
8080/tcp
```

如果公司网络权限由运维管理，则让运维只对公司内网开放服务器的TCP 8080端口，不要对公网开放。

## 5. 从其他电脑访问

浏览器打开：

```text
http://服务器IP:8080/feedback/
```

例如：

```text
http://10.20.30.40:8080/feedback/
```

如果服务器内部 `curl`返回200，但其他电脑打不开，说明是服务器防火墙或公司网络策略问题，需要找运维放通。

此时完成的是静态版本：每位同事都能打开网页，但每个人导入的 JSON只在自己的浏览器中生效。管理员上传一次、所有人共享数据的功能还需要继续开发 Java后端和服务器存储。
