### 创建docker镜像
1. jenkins build 的时候会按照项目里写的dockerfile文件进行构建docker镜像
2. 镜像构建完成之后，会推送到远程镜像仓库，供k8s集群使用
3. dockerfile 的编写一定要规范，不建议编写低于17.0版本的dockerfile
4. 基础镜像一定要统一，避免重复下载基础镜像。
5. dockefile 打成的镜像要加上时间戳，以进行老镜像的区分
