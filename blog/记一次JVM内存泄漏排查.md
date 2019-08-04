# 问题描述
* hbase_rpc_server程序，接收client请求读写数据库；
* 程序运行一段时间（3小时）后抛出java.lang.OutOfMemoryError:Java heap space；

# 解决方案
* `jps`命令定位程序进程：`jps -lvm | grep rpc`找到程序pid
* 服务端启动`jstatd`远程监控服务
* 客户端以`jvisualvm`工具连接jstatd端口，根据pid查看服务的运行情况；
* `jvisualvm`中安装visual gc插件，发现eden区每次回收后都有很多的survivor，survivor的1和2区交换几次满了后就都到old gen老年代去了，
导致每次回收后内存使用量一直在增长，内存使用曲线呈现45度锯齿状；
![i内存使用曲线](https://user-images.githubusercontent.com/3156608/62417899-70777900-b68e-11e9-8eb7-37d367b316fb.png)
![GC可视化](https://user-images.githubusercontent.com/3156608/62417901-78cfb400-b68e-11e9-9ec4-3b6b141757ef.png)
毫无疑问是内存泄漏了！！！
* 程序添加OOM时输出日dump志，java程序启动命令新增：-XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=/home/users/developer/service/log/mlp
* 下一次OOM发生后将生成的dump文件导入jvisualvm中分析；
* dump分析发现最多的是char[]，类实例中大多是insert语句语句中涉及的参数：清楚明白了，是数据库连接未释放
* 数据库连接是本地连接池管理的，所以基本不释放，但dao中生成的preparestatement和resultset需要手动释放；
* 在finally中添加statement.close方法释放资源；
* 重新运行程序，世界一片美好，回复了正常的非倾斜的锯齿状内存占用曲线；

