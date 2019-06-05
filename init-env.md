###  新建命名空间namespace和授权
对于k8s集群来说，使用namespace进行资源的隔离和限制，故每一个namespace可以看做是一条环境。当然，集群需要进行权限的访问控制
，所以，主账号可以对于子账号进行授权特定空间的访问，参阅：[阿里云主账号授权子账号设置](https://help.aliyun.com/document_detail/87656.html?spm=a2c4g.11186623.6.580.6e7e678cb3AYrN)

### 挂载PVC持久化存储盘

如果需要数据的持久化存储，则需要手动创建pvc,请参阅[使用nas挂载存储](https://help.aliyun.com/document_detail/88940.html?spm=a2c4g.11186623.6.666.520f49feQc4fhs)

重要：因为我们实现的自动化部署的模板，默认的持久化存储申明（pvc）的名称和命名空间一致，所以，在创建的pvc申明中，必须
命名和namespace名称一致，否则，使用pvc的服务将无法正常启动。
### 准备必须的秘钥
目前，必须的必要只有一个，就是image_secret,这个是用来从镜像仓库拉取镜像认证权限使用的。有两种方式创建secret,一种是yaml方式
提交，一种是在控制台手动创建，注意的是，这个秘钥是属于某个命名空间的，不是全局的，yaml方式创建如下：
```
# 配置 docker镜像拉取的秘钥和
apiVersion: v1
data:
  .dockerconfigjson: eyJhdXRocyI6eyJyZWdpc3RyeS12cGMuY24tYmVpamluZy5hbGl5dW5jcy5jb20iOnsidXNlcm5hbWUiOiJjaGVjaGVfazhzQDE1MTI1MjI2OTE5NTE0ODkiLCJwYXNzd29yZCI6ImNoZWNoZWs4cyIsImF1dGgiOiJZMmhsWTJobFgyczRjMEF4TlRFeU5USXlOamt4T1RVeE5EZzVPbU5vWldOb1pXczRjdz09In19fQ==
kind: Secret
metadata:
  name: image-secret
  namespace: i-bedrock
type: kubernetes.io/dockerconfigjson

```
使用kubectl apply -f image-sercte.yaml -n 命名空间名称即可。注意name名称属性，结合我们的发布模板，固定为: image-secret

### 准备引入外部服务
1. 一般情况，mysql,redis，mongo等有状态的服务不需要经常变更，我们为了稳定性，一般把服务单独部署在k8s集群外面或者是直接够买了
   阿里云的对应的服务
2. 引入服务的方式有两种，一种是自己搭建，暴露IP端口的，一种是直接买的服务，使用域名访问的，下面介绍对于两种方式如何引入k8s
#### 配置ExternalName方式的service引入域名型

        apiVersion: v1
        kind: Service
        metadata:
          name: mysql-bedrock
          namespace: bedrock
        spec:
          externalName: rm-2zeh269b9w3e2odt9.mysql.rds.aliyuncs.com
          sessionAffinity: None
          type: ExternalName
注意：externalName 就是服务提供的域名
#### 配置Service和Endpoints 引入端口型

    ---
      apiVersion: v1
      kind: Endpoints
      metadata:
        name: redis-gow
        namespace: bedrock
      subsets:
      - addresses:
        - ip: 10.153.134.94
        ports:
        - port: 6379
          protocol: TCP
    ---
      //配置service
      apiVersion: v1
      kind: Service
      metadata:
        name: redis-gow
        namespace: bedrock
      spec:
        ports:
        - port: 6379
          protocol: TCP
          targetPort: 6379
        sessionAffinity: None
        type: ClusterIP
注意：请注意Endpoints的name和service里name属性一致。

### k8s内部使用引入的外部服务
```
例如，以前的jdbc连接是 ：
jdbc:mysql://192.168.1.18:3306/c_bedrock?characterEncoding=utf8
在k8s 内部可以配置为：
jdbc:mysql://mysql:3306/c_bedrock?characterEncoding=utf8；
因为IP端口在k8s里已经虚拟化为一个服务，使用服务名可以代替ip，这是通过k8s 内部dns解析实现的
```
