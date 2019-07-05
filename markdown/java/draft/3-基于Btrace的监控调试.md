进入[Btrace官网](https://github.com/btraceio/btrace)下载安装



配置环境变量

```shell
vi ./.bash_profile
export BTRACE_HOME=/Users/lishaofei/Environment/btrace-bin-1.3.11.3
export PATH=$PATH:$BTRACE_HOME
```



运行脚本的方式

- 在 JVistualVM中添加 Btrace 插件，添加 classpath
- 使用命令行 btrace <pid> <trace_script>