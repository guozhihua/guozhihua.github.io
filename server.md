### 配置内部SLB
阿里云k8s 集群在初始化的时候，需要一个外网的slb做集群入口，这个slb绑定到了nginx-controler 这个负载控制器上；但是有个问题，如果绑定的服务端口徐还要新增80和443之外的端口是不行的，所以我们采用了nat转发弹性公网地址的端口到内网SLB的方式，取代了阿里云默认的slb.自定义nginx-controller控制器绑定内网SLB如下（管理员操作）：

```
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet
    service.beta.kubernetes.io/alicloud-loadbalancer-force-override-listeners: 'true'
    service.beta.kubernetes.io/alicloud-loadbalancer-id: lb-2ze9mxrew1ompynh9tqfc
  creationTimestamp: '2019-03-11T02:42:00Z'
  labels:
    app: nginx-ingress-lb
  name: nginx-ingress-lb
  namespace: kube-system

spec:
  clusterIP: 172.21.10.85
  externalTrafficPolicy: Local
  healthCheckNodePort: 31521
  ports:
    - name: http
      nodePort: 30736
      port: 80
      protocol: TCP
      targetPort: 80
    - name: https
      nodePort: 31844
      port: 443
      protocol: TCP
      targetPort: 443
  selector:
    app: ingress-nginx
  sessionAffinity: None
  type: LoadBalancer
```
注意上面的三个注解
```
      service.beta.kubernetes.io/alicloud-loadbalancer-address-type: intranet  //代表是内网类型的slb
      service.beta.kubernetes.io/alicloud-loadbalancer-force-override-listeners: 'true' //允许覆盖
      service.beta.kubernetes.io/alicloud-loadbalancer-id: lb-2ze9mxrew1ompynh9tqfc //在slb控制台新建服务得到的slb 的id
```

使用kubectl apply 提交了这个文件之后，内网slb服务就和k8s入口服务对接了起来。

### 配置域名
配置域名解析到集群的弹性公网IP即可，最好是域名泛解析，这样运维起来会更加灵活

### 新增Node节点
必须在同一个VPC网络才可以添加为集群节点，通过控制台删除或者新增节点即可
