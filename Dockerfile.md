
### FROM 命令
    用于指定基础镜像，必须有，一般，java 基于jre,python 的就选择对应版本的镜像
### WORKDIR
    指定工作目录，其实后面再写命令如COPY命名 后面的/代表的就是工作目录
### COPY
    需要复制可执行的程序文件，配置文件，打入我们的镜像中
### CMD
     镜像启动命令，类似于tomcat start 这样的操作。
		 以上所有的命令，每一行代表一层，层越多，相应的镜像越大。
### 示例如下：
```
FROM openjdk:8u171-jre-alpine
WORKDIR /gateway
COPY ./build/libs/*gateway-*.jar /gateway
CMD java \
	-Ddiscovery.hostname=${DISCOVERY_HOSTNAME} \
	-Dserver.port=${server_port}  \
    -Duser.timezone=GMT+08 \
    -Xms512m -Xmx512m -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintFlagsFinal -XX:+PrintCommandLineFlags -XX:+PrintGC \
    -jar *gateway-*.jar


```
