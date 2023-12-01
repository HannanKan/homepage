# Maven

## `-P` 选项
-P 标志用于指定要激活的 Maven Profile（Maven 配置文件）。Maven 项目中可用的 Profile 在 pom.xml 文件中可以找到。例如下面一个名为 `hadoop-3.2` 的 Profile 。

```xml
    <profile>
      <id>hadoop-3.2</id>
      <properties>
        <hadoop-client-api.artifact>hadoop-client-api</hadoop-client-api.artifact>
        <hadoop-client-runtime.artifact>hadoop-client-runtime</hadoop-client-runtime.artifact>
        <hadoop-client-minicluster.artifact>hadoop-client-minicluster</hadoop-client-minicluster.artifact>
      </properties>
    </profile>
```

可以使用以下命令将项目部署到远程仓库：
```shell
mvn deploy -Phadoop-3.2 
```
这将使用 hadoop-3.2 profile 中定义的配置信息来执行 maven 命令。