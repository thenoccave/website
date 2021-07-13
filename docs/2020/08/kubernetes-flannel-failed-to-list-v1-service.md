# Kubernetes / Flannel – Failed to list *v1.Service
I was recently building a new HA Kubernetes cluster and was having issues with CoreDNS failing to do it’s thing.
### The Problem
The Kubernetes nodes were reporting as ready however the CoreDNS pods were refusing to be healthy
```
[root@km01 ~]# kubectl get pods --all-namespaces
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-kxvnf   0/1     Running   2          18h
kube-system   coredns-6955765f44-wgh6c   0/1     Running   2          18h
```
All the nodes were reporting as Ready
```
[root@km01 ~]# kubectl get nodes
NAME                        STATUS   ROLES    AGE     VERSION
km01                        Ready    master   13d     v1.17.3
km02                        Ready    master   3d22h   v1.17.3
km03                        Ready    master   3d20h   v1.17.3
kw01                        Ready    <none>   2d19h   v1.17.3
kw02                        Ready    <none>   21h     v1.17.3
kw03                        Ready    <none>   21h     v1.17.3
```
The 2 CoreDNS pods logs were full of
```
root@km01 ~]# kubectl logs coredns-6955765f44-kxvnf -n kube-system'
I0806 23:36:54.132644       1 trace.go:82] Trace[1939897294]: "Reflector pkg/mod/k8s.io/client-go@v0.0.0-20190620085101-78d2af792bab/tools/cache/reflector.go:98 ListAndWatch" (started: 2020-08-06 23:36:24.131764014 +0000 UTC m=+63460.404444423) (total time: 30.000829888s):
Trace[1939897294]: [30.000829888s] [30.000829888s] END
E0806 23:36:54.132664       1 reflector.go:125] pkg/mod/k8s.io/client-go@v0.0.0-20190620085101-78d2af792bab/tools/cache/reflector.go:98: Failed to list *v1.Service: Get https://10.96.0.1:443/api/v1/services?limit=500&amp;resourceVersion=0: dial tcp 10.96.0.1:443: i/o timeout
E0806 23:36:54.132664       1 reflector.go:125] pkg/mod/k8s.io/client-go@v0.0.0-20190620085101-78d2af792bab/tools/cache/reflector.go:98: Failed to list *v1.Service: Get https://10.96.0.1:443/api/v1/services?limit=500&amp;resourceVersion=0: dial tcp 10.96.0.1:443: i/o timeout
E0806 23:36:54.132664       1 reflector.go:125] pkg/mod/k8s.io/client-go@v0.0.0-20190620085101-78d2af792bab/tools/cache/reflector.go:98: Failed to list *v1.Service: Get https://10.96.0.1:443/api/v1/services?limit=500&amp;resourceVersion=0: dial tcp 10.96.0.1:443: i/o timeout
E0806 23:36:54.132664       1 reflector.go:125] pkg/mod/k8s.io/client-go@v0.0.0-20190620085101-78d2af792bab/tools/cache/reflector.go:98: Failed to list *v1.Service: Get https://10.96.0.1:443/api/v1/services?limit=500&amp;resourceVersion=0: dial tcp 10.96.0.1:443: i/o timeout
[INFO] plugin/ready: Still waiting on: "kubernetes"
[INFO] plugin/ready: Still waiting on: "kubernetes"
[INFO] plugin/ready: Still waiting on: "kubernetes"
```
After checking the following:

- Flannel interfaces existed on each node
- The routes on each table were checked to ensure each other servers subnet was present and I could ping flannel interfaces on other servers
- Kublet was running with the parameter –network-plugin=cni
- kube-controller-manager was running with the parameters –allocate-node-cidrs=true and –cluster-cidr

### The Solution
After looking at the Flannel subnet.env file way to many times it finally clicked. The FLANNEL_SUBNET doesn’t exist in the FLANNEL_NETWORK and looking at the other nodes they also had FLANNEL_SUBNETS outside of FLANNEL_NETWORK
```
[root@km01 ~]# cat /run/flannel/subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.243.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```
This is caused because we boot strapped the cluster to use the cidr 10.243.0.0/16. The default flannel deployment has the network set as 10.244.0.0/16 conflicting with what we bootstrapped the cluster with.
```
net-conf.json: |
   {
     "Network": "10.244.0.0/16",
     "Backend": {
       "Type": "vxlan"
     }
   }
```
The fix is to edit the flannel config map to update the network subnet to match.
```
[root@km01 ~]# kubectl edit configmap kube-flannel-cfg -n kube-system
apiVersion: v1
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.243.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
```
Restart the flannel pods
```
[root@km01 ~]# kubectl delete pods -l app=flannel -n kube-system
pod "kube-flannel-ds-amd64-9tmwn" deleted
pod "kube-flannel-ds-amd64-b4htd" deleted
pod "kube-flannel-ds-amd64-b6kmn" deleted
pod "kube-flannel-ds-amd64-nlmqk" deleted
pod "kube-flannel-ds-amd64-qpqsv" deleted
pod "kube-flannel-ds-amd64-zhh95" deleted
```
After flannel restarted the subnet.env showed the correct values.
```
[root@km01 ~]# cat /var/run/flannel/subnet.env
FLANNEL_NETWORK=10.243.0.0/16
FLANNEL_SUBNET=10.243.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
```
And CoreDNS started reporting as healthy.
```
[root@km01 ~]# kubectl get pods -l k8s-app=kube-dns -n kube-system
NAME                       READY   STATUS    RESTARTS   AGE
coredns-6955765f44-sxctb   1/1     Running   0          66m
coredns-6955765f44-xhhcp   1/1     Running   1          66m
```