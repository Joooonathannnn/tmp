环境已经明确：

- 系统：Huawei Cloud EulerOS 2.0
- 架构：`x86_64`
- 包管理器：`yum`、`dnf`都有
- Java：尚未安装
- 软件源：`base`、`updates`正常

HCE 2.0同时支持 `yum` 和 `dnf`，下面统一使用 `dnf`。[华为云HCE软件管理说明](https://support.huaweicloud.com/intl/en-us/usermanual-hce/hce_repo.html)

先不要急着安装 Tomcat，第一步要安装 Java。

请依次执行：

```bash
sudo dnf makecache
```

然后查询 Java 17：

```bash
sudo dnf list --available "java-17-openjdk*"
```

再查询 Java 11：

```bash
sudo dnf list --available "java-11-openjdk*"
```

同时看看软件源有没有自带 Tomcat：

```bash
sudo dnf list --available "tomcat*"
```

把这三条查询命令的输出发给我。

我们优先选择：

```text
Java 17 + Tomcat 10.1
```

因为后续开发管理员上传功能时，可以使用现代版本的 Spring Boot。如果软件源没有 Java 17，再根据查询结果选择 Java 11。先查询再安装，可以避免包名或版本不匹配。
