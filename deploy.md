###  准备服务配置文件configMap
ConfigMap 是一种以不加密的方式存储配置信息的资源类型，可以挂载到运行的pod中作为环境变量，我们结合Spring框架从环境变量获取程序运行配置的特点，采用这种类型的配置进行配置的抽取，定义，从而在不同环境做到不同的配置，而和程序解耦合。

一般情况，此配置文件的内容都是业务配置信息，需要开发人员自行配置即可，发布模板都是写好的，每次部署都会从新更新配置。

例如，开发人员修改了pc项目的配置属性，修改的配置文件为pc-config.yaml,内容如下：
```
apiVersion: v1
data:
  sso.client.admin_host: http://admin.i.bedrock.chetimes.com
  sso.server.host: https://i.bedrock.chetimes.com/cas
  sso.client.app_host:  https://i.bedrock.chetimes.com
kind: ConfigMap
metadata:
  name: pc-config
  namespace: i-bedrock
```
pc-cofig.yaml 文件中，sso.client.admin_host、sso.server.host、sso.client.app_host 原先是配置在程序里的application.yml文件中的属性，注意如果在k8s里面的configmap里进行了配置的属性，将不会加载applicaiton.yml里对应的属性，这是由spring的属性加载优先级决定的。

对于不适用spring的加载机制的，在比较复杂，需要使用到—D参数进行加载，还需要修改dockerfile的启动命令，这是非常不规范的方式。只有在dockerfile启动命令加入了 -Dtest=${test} 这种方式，在configmap中加入 test：123 才能生效，启动的时候-Dtest=123 。

对于隐私程度较高的信息，如数据库地址，密码等信息，我们抽取在了另一个configmap，common-configmap。也就是说发布的模板项目里引用了两个configmap，一个是common-congimap,引入的私密信息；一个是业务自己的configmap,如pc-config.yml;业务开发人员只能修改自己的业务配置文件，无法修改common-configmap .


### 初始化common-configmap
文件里的name名称固定为：common-config；内容如下:
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: common-config
  namespace: i-bedrock
data:
  spring.data.mongodb.database: cheche_k8s
  spring.data.mongodb.host: mongo
  spring.data.mongodb.password: xxxxxx
  spring.data.mongodb.port: "3717"
  spring.data.mongodb.username: i_bedrock
  spring.profiles.active: k8s
  k8s.ds_password: xxxxxxx
  k8s.ds_url: jdbc:mysql://mysql:3306/c_bedrock?characterEncoding=utf8
  k8s.ds_username: c_bedrock
  k8s.redis_host: redis
  k8s.redis_password: xxxxxxxx
  k8s.redis_port: "6379"
  _JAVA_OPTIONS: -Dspring.profiles.active=k8s -Dserver.port=$server.port
  PROFILES_ACTIVE: k8s
  conf.paths: /data/conf-dev/
  mail.ssl: true
  spring.zipkin.base-url: http://jaeger-service:9411
  spring.sleuth.enabled: true
  db_name: c_bedrock
  db_password: xxxxxxx
  jdbc_url: jdbc:mysql://mysql:3306/c_bedrock?characterEncoding=utf8
```

在确定好该环境的支撑服务如redis，mysql的地址之后，创建一次即可；当然，也是可以修改的，修改完之后使用到这些属性的服务只有重启才生效或者重新发布生效。

###  jenkins一键部署

上面多次提到了模板项目，目前其实在gitlab上就是k8s-cd项目。注意这个项目的第一层的文件名就是服务名，第二层代表的是每一个的环境namespace.对于开发人员来说，直接在jenkin 执行对应的服务build任务即可。如何实现请参阅项目[k8s-cd wiki](http://192.168.1.213/devops/k8s-cd/wikis/home)
