# 使用K8s安装istio
一，下载istio 1.2.2版本
``` 
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.2.2 sh -
安装目录中包含：

在 install/ 目录中包含了 Kubernetes 安装所需的 .yaml 文件
samples/ 目录中是示例应用
istioctl 客户端文件保存在 bin/ 目录之中。istioctl 的功能是手工进行 Envoy Sidecar 的注入。
istio.VERSION 配置文件
```
二， 安装
```
cd istio/
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```
三，验证
```
验证svc

$ kubectl get svc -n istio-system
NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                                                                                                                   AGE
grafana                  ClusterIP      172.21.211.123   <none>          3000/TCP                                                                                                                  2m
istio-citadel            ClusterIP      172.21.177.222   <none>          8060/TCP,9093/TCP                                                                                                         2m
istio-egressgateway      ClusterIP      172.21.113.24    <none>          80/TCP,443/TCP                                                                                                            2m
istio-galley             ClusterIP      172.21.132.247   <none>          443/TCP,9093/TCP                                                                                                          2m
istio-ingressgateway     LoadBalancer   172.21.144.254   52.116.22.242   80:31380/TCP,443:31390/TCP,31400:31400/TCP,15011:32081/TCP,8060:31695/TCP,853:31235/TCP,15030:32717/TCP,15031:32054/TCP   2m
istio-pilot              ClusterIP      172.21.105.205   <none>          15010/TCP,15011/TCP,8080/TCP,9093/TCP                                                                                     2m
istio-policy             ClusterIP      172.21.14.236    <none>          9091/TCP,15004/TCP,9093/TCP                                                                                               2m
istio-sidecar-injector   ClusterIP      172.21.155.47    <none>          443/TCP                                                                                                                   2m
istio-telemetry          ClusterIP      172.21.196.79    <none>          9091/TCP,15004/TCP,9093/TCP,42422/TCP                                                                                     2m
jaeger-agent             ClusterIP      None             <none>          5775/UDP,6831/UDP,6832/UDP                                                                                                2m
jaeger-collector         ClusterIP      172.21.135.51    <none>          14267/TCP,14268/TCP                                                                                                       2m
jaeger-query             ClusterIP      172.21.26.187    <none>          16686/TCP                                                                                                                 2m
kiali                    ClusterIP      172.21.155.201   <none>          20001/TCP                                                                                                                 2m
prometheus               ClusterIP      172.21.63.159    <none>          9090/TCP                                                                                                                  2m
tracing                  ClusterIP      172.21.2.245     <none>          80/TCP                                                                                                                    2m
zipkin                   ClusterIP      172.21.182.245   <none>          9411/TCP               

验证pod
$ kubectl get pods -n istio-system
NAME                                                           READY   STATUS      RESTARTS   AGE
grafana-f8467cc6-rbjlg                                         1/1     Running     0          1m
istio-citadel-78df5b548f-g5cpw                                 1/1     Running     0          1m
istio-cleanup-secrets-release-1.1-20190308-09-16-8s2mp         0/1     Completed   0          2m
istio-egressgateway-78569df5c4-zwtb5                           1/1     Running     0          1m
istio-galley-74d5f764fc-q7nrk                                  1/1     Running     0          1m
istio-grafana-post-install-release-1.1-20190308-09-16-2p7m5    0/1     Completed   0          2m
istio-ingressgateway-7ddcfd665c-dmtqz                          1/1     Running     0          1m
istio-pilot-f479bbf5c-qwr28                                    2/2     Running     0          1m
istio-policy-6fccc5c868-xhblv                                  2/2     Running     2          1m
istio-security-post-install-release-1.1-20190308-09-16-bmfs4   0/1     Completed   0          2m
istio-sidecar-injector-78499d85b8-x44m6                        1/1     Running     0          1m
istio-telemetry-78b96c6cb6-ldm9q                               2/2     Running     2          1m
istio-tracing-69b5f778b7-s2zvw                                 1/1     Running     0          1m
kiali-99f7467dc-6rvwp                                          1/1     Running     0          1m
prometheus-67cdb66cbb-9w2hm                                    1/1     Running     0          1m
```
备注
```
开启istio的自动注入，即给对应的ns打上istio-injection=enabled标签即可
kubectl label ns default istio-injection=enabled   
```