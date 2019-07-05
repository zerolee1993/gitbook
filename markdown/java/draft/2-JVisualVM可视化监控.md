[JVisualVM官方文档](https://htmlpreview.github.io/?https://raw.githubusercontent.com/visualvm/visualvm.java.net.backup/master/www/zh_CN/gettingstarted.html)



- 监控远程 Tomcat 的方式

修改 Catalina.sh 添加如下参数：

```shell
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.serve.hostname=39.105.86.150"
```

- 监控远程普通 Java 进程

添加启动参数

```shell
nohup java -Dcom.sum.management.jmxremote -Dcom.sun.management.jmxremote.port=9004 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.net.preferIPv4Stack=true -Djava.rmi.serve.hostname=39.105.86.150 -jar test.jar &
```

