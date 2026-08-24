`firewall-cmd --state`显示 `not running`，说明服务器没有启用 `firewalld`。这不代表网站有问题，也不需要马上启动防火墙。

不要自行执行：

```bash
sudo systemctl start firewalld
```

因为公司可能使用外部防火墙或统一网络策略，贸然启动可能影响现有服务。

先在服务器检查 Tomcat监听地址：

```bash
ss -lntp | grep 8080
```

正常应该看到类似：

```text
0.0.0.0:8080
```

或者：

```text
*:8080
```

这表示允许其他机器连接。

如果显示：

```text
127.0.0.1:8080
```

则表示只允许服务器自己访问，需要修改 Tomcat监听配置。

然后在另一台公司内网 Windows电脑的 PowerShell中测试：

```powershell
Test-NetConnection 服务器IP -Port 8080
```

重点看：

```text
TcpTestSucceeded : True
```

判断结果：

- `True`：网络已通，直接访问 `http://服务器IP:8080/feedback/`。
- `False`，但服务器监听 `0.0.0.0:8080`：公司网络策略、安全组或上层防火墙没有放行，需要找运维。
- 服务器内部 `curl`也失败：Tomcat或看板部署存在问题。

如果网络不通，可以告诉运维：

> Tomcat已经监听服务器TCP 8080端口，服务器本地访问正常，firewalld未运行，但其他内网PC连接8080失败，请检查虚拟机安全组或内网访问策略。
