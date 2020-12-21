# k8s-s3fs-aws-zhy
Mount s3fs as k8s persistent volume in AWS ZHY Region

### 1. 将本项目整体Clone到本地

```
git clone https://github.com/toreydai/k8s-s3fs-aws-zhy.git
```
### 2. 修改并创建Kubernetes Configmap

```
# S3_BUCKET：将要挂载的Amazon S3存储桶的名称
# AWS_KEY: 具有S3读写权限的AWS IAM User的Access Key
# AWS_SECRET_KEY：具有S3读写权限的AWS IAM User的Secret Key

data:
  S3_BUCKET: <YOUR-S3-BUCKET-NAME>
  AWS_KEY: <YOUR-AWS-TECH-USER-ACCESS-KEY>
  AWS_SECRET_KEY: <YOUR-AWS-TECH-USER-SECRET>
```
修改完成后，创建Configmap

```
kubectl create -f configmap_secrets_template.yaml
```
### 3. 创建Kubernetes Daemonset
```
kubectl create -f daemonset.yaml
```
查看daemonset是否运行正常

```
kubectl get ds
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
s3-provider   1         1         1       1            1           <none>          144m

kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
s3-provider-ckgd5             1/1     Running   0          144m
```

### 4. 创建示例Pod挂载S3FS进行测试
```
# 创建测试Pod
kubectl create -f example_pod.yaml

# 查看Pod状态
kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
NAME                          READY   STATUS    RESTARTS   AGE
s3-provider-ckgd5             1/1     Running   0          144m
test-pd                       1/1     Running   0          143m

# 登陆到Pod中查看挂载的s3fs目录
kubectl exec -it test-pd -- /bin/bash
root@test-pd:/# ls -la /var/s3
drwxrwxrwx 1 root root        0 Jan  1  1970 .
drwxr-xr-x 1 root root       29 Dec 21 04:28 ..
---------- 1 root root    13173 Jun 15  2020 bbr.sh
---------- 1 root root      739 Jun 15  2020 eks_role.yaml
---------- 1 root root      197 Jun 15  2020 index.html
d--------- 1 root root        0 Aug 17 03:26 openvpn

# 写入一个文件进行测试
root@test-pd:/# echo helloworld >> test.txt
root@test-pd:/# ls -la /var/s3
drwxrwxrwx 1 root root        0 Jan  1  1970 .
drwxr-xr-x 1 root root       29 Dec 21 04:28 ..
---------- 1 root root    13173 Jun 15  2020 bbr.sh
---------- 1 root root      739 Jun 15  2020 eks_role.yaml
---------- 1 root root      197 Jun 15  2020 index.html
d--------- 1 root root        0 Aug 17 03:26 openvpn
-rw-r--r-- 1 root root        9 Dec 21 04:07 test.txt

```

### 5. 在Amazon S3存储桶中查看
![avatar](https://github.com/toreydai/k8s-s3fs-aws-zhy/blob/main/s3fs-s3.png)

参考资料：
https://aws.amazon.com/cn/blogs/china/use-u3fs-as-shared-storage-to-kubernetes-pod/
