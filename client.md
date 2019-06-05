### 获取登录凭证

登录[阿里云容器服务](<https://cs.console.aliyun.com/?spm=5176.2020520001.aliyun_sidebar.10.69864bd3N5GRIT#/k8s/cluster/c4082eec4463a4912b3f0df1c27929063/info?eci=false>)，基本信息内，通过 kubectl 连接 Kubernetes 集群 获取认证信息

* 1. 从 [Kubernetes 版本页面](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md) 下载最新的 kubectl 客户端。

  2. 安装和设置 kubectl 客户端。有关详细信息，参见 [安装和设置 kubectl](https://kubernetes.io/docs/tasks/kubectl/install/)

  3. 配置集群凭据:

     将凭证 复制到 客户端  $HOME/.kube/config   #\$HOME 为登录到阿里云k8s server的用户home目录 



### 依赖环境

| 环境    | 版本                                     |
| ------- | ---------------------------------------- |
| Ubuntu  | 16.04   -- 系统非依赖                    |
| kubectl | 1.13.2                                   |
| git     | any                                      |
| docker  | Docker version 18.06.3-ce, build d7080c1 |
| Gradle  |                                          |
| Jenkins |                                          |

### 环境部署

- Cent OS

```shell
$ sudo yum -y install net-tools   #centos7 网络工具
$ sudo yum -y install git
$ sudo yum -y install kubernetes-client

$ sudo yum install docker  # docker 环境
$ sudo reboot # 如果安装docker 需更新内核信息
```

> \# yum search kubectl
>
> 已加载插件：fastestmirror
> Loading mirror speeds from cached hostfile
>
> - base: mirrors.aliyun.com
> - extras: mirrors.163.com
> - updates: mirrors.aliyun.com
>   ============ 匹配：kubectl =========
>   kubernetes-client.x86_64 : Kubernetes client tools

- Ubuntu

```shell
# docker
$ sudo apt-get install docker

# k8s client 源 需要科学上网 故采用二进制方法
$ mkdir ~/soft
$ cd ~/soft
$ wget https://dl.k8s.io/v1.13.2/kubernetes-client-linux-amd64.tar.gz
$ tar xzf kubernetes-client-linux-amd64.tar.gz
$ cd ~/soft/kubernetes/client/bin
$ sudo ln -s ~/soft/kubernetes/client/bin/kubectl /usr/bin
```



### Docker 登录阿里云

```
docker login --username=cheche_k8s@1512522691951489 registry.cn-beijing.aliyuncs.com --password xxxxxx   # 登录阿里云docker镜像仓库
Login Succeeded
```



### 配置 k8s client 认证文件(阿里云)

> 参考文档

```shell
$ cd ~/.kube    # if not exists mkdir .kube
$ vim config    # 编辑config文件 将凭证粘贴到文件。

$ kubectl get nodes  # 测试是否成功  类似返回为成功
NAME                                STATUS   ROLES    AGE    VERSION
cn-beijing.i-2ze16wo7jtythx5o7hv6   Ready    <none>   127d   v1.11.5
```



### 获取项目配置信息

> 此处用例 测试环境 git地址：http://192.168.1.213/devops/k8s-cd.git

```shell
$ git clone http://192.168.1.213/devops/k8s-cd.git
$ cd k8s-cd
```



### yaml模板替换

* 需要替换内容

```
deployment.yaml:  namespace: ${namespace}
deployment.yaml:  replicas: ${replicas}
deployment.yaml:          image: registry-vpc.cn-beijing.aliyuncs.com/cheche365/cheche:${imageName}

ingress.yaml:  - host: ${domin}
ingress.yaml:  namespace: ${namespace}

service.yaml:  namespace: ${namespace}
```

* 已有内容

```shell
# grep ':' uat/config.properties
domain: opc.bc.com
namespace: uat
replicas : 2
```

* shell 

```shell
$ config=uat/config.propertie

# service.yaml 
$ sed "s/\(namespace:\).*/\1 `awk -F':' '/namespace/{print $2}' ${config}`/" service.yaml  > uat/service.yaml

#ingress.yaml
$ sed -e "s/\(host:\).*/\1 `awk -F':' '/domain/{print $2}' ${config}`/" -e "s/\(namespace:\).*/\1 `awk -F':' '/namespace/{print $2}' ${config}`/" ingress.yaml > uat/ingress.yaml

# deployment.yaml
$ sed -e "s/\(namespace:\).*/\1 `awk -F':' '/namespace/{print $2}' ${config}`/" -e "s/\(replicas:\).*/\1 `awk -F':' '/replicas/{print $2}' ${config}` /" -e "s/\(image\: registry-vpc.cn-beijing.aliyuncs.com\/cheche365\/cheche\:\).*/\1$imageName/" deployment.yaml >uat/deployment.yaml

```



### Gradle 测试   (Jenkins 集成)

```
$ cd /path_of_project/
$ gradle -Dorg.gradle.daemon=false
-Dorg.gradle.jvmargs="-Xms4g
-Xmx4g
-XX:+UseG1GC
-Xverify:none
-XX:ReservedCodeCacheSize=256m
-XX:SoftRefLRUPolicyMSPerMB=256"
clean
-x test
admin:build  # 测试
```



### 构建docker 镜像

```shell
# docker build -t admin:1.01 -f ./admin/Dockerfile .  # 测试语句
$ docker build -f ./admin/Dockerfile -t registry.cn-beijing.aliyuncs.com/cheche365/cheche:admin_Ali --force-rm .
```

> ### 命名 为阿里格式 / 上传
>
> ```
> $ docker tag f32c1c082c9f registry.cn-beijing.aliyuncs.com/cheche365/cheche:admin1.01
> $ docker tag [ImageId] registry.cn-beijing.aliyuncs.com/cheche365/cheche:[镜像版本号]
> 
> $ docker push registry.cn-beijing.aliyuncs.com/cheche365/cheche:admin1.0
> $ docker push registry.cn-beijing.aliyuncs.com/cheche365/cheche:[镜像版本号]
> ```



_____

### 提交配置

 *kubectl 执行用户 需要和 配置.kube/config 是同一用户*

```shell
$ kubectl apply -f path_obj/common-configmap.yaml    # 数据配置文件
$ kubectl apply -f uat/
```

