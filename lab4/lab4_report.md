University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)  
Year: 2023/2024  
Group: K4110c  
Author: Trapezin Andrey  
Lab: Lab4  
Date of create: 28.09.2023  
Date of finished: 31.09.2023
---

```bash
➜  ~ minikube start --network-plugin=cni --cni=calico -p=multinode --nodes 2
```

```bash
➜  ~ k get no
NAME                 STATUS   ROLES           AGE     VERSION
multinode       Ready    control-plane   4m35s   v1.27.4
multinode-m02   Ready    <none>          3m48s   v1.27.4
```

```bash
➜  ~ k get po -l k8s-app=calico-node -A
NAMESPACE     NAME                READY   STATUS    RESTARTS   AGE
kube-system   calico-node-4d7r9   1/1     Running   0          3m38s
kube-system   calico-node-qjlxj   1/1     Running   0          4m8s
```

```bash
➜  ~ k get ippools.crd.projectcalico.org default-ipv4-ippool -o yaml
...
cidr: 10.244.0.0/16
...
```

```bash
➜  ~ k delete ippools.crd.projectcalico.org default-ipv4-ippool
ippool.crd.projectcalico.org "default-ipv4-ippool" deleted
```

```bash
➜  ~ k label no multinode rack=1
node/multinode labeled
➜  ~ k label no multinode-m02 rack=2
node/multinode-m02 labeled
```

```bash
➜  ~ k describe no multinode
Name:               multinode
Roles:              control-plane
Labels:             ...
                    rack=1
...
```

```bash
➜  ~ k describe no multinode-m02
Name:               multinode-m02
Roles:              <none>
Labels:             ...
                    rack=2
...
```

ippool.yml
```yaml
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-1-ippool
spec:
  cidr: 10.244.0.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "1"
  vxlanMode: Never
---
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: rack-2-ippool
spec:
  cidr: 10.244.1.0/24
  ipipMode: Always
  natOutgoing: true
  nodeSelector: rack == "2"
  vxlanMode: Never
```

```bash
➜  lab4 git:(main) ✗ k apply -f ippool.yml
ippool.crd.projectcalico.org/rack-1-ippool created
ippool.crd.projectcalico.org/rack-2-ippool created
```

```bash
➜  ~ k create ns labs
namespace/labs created
➜  ~ helm install react-app -n labs react-app
```

values.yaml
```yaml
...
image:
  repository: ifilyaninitmo/itdt-contained-frontend
  tag: "master"
service:
  type: ClusterIP
  port: 3000
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 2
...
```

```bash
➜  lab4 git:(main) ✗ helm install react-app -n labs react-app
```

```bash
➜  lab4 git:(main) ✗ k get po -n labs -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP             NODE            NOMINATED NODE   READINESS GATES
react-app-f66d69d6b-6jdg5   1/1     Running   0          9m27s   10.244.1.129   multinode-m02   <none>           <none>
react-app-f66d69d6b-n9t2k   1/1     Running   0          9m12s   10.244.0.193   multinode       <none>           <none>
```

```bash
➜  lab4 git:(main) ✗ k exec -ti -n labs pods/react-app-f66d69d6b-6jdg5 -- sh
/frontend # ping 10.244.0.193
PING 10.244.0.193 (10.244.0.193): 56 data bytes
64 bytes from 10.244.0.193: seq=0 ttl=62 time=6.562 ms
64 bytes from 10.244.0.193: seq=1 ttl=62 time=0.257 ms
64 bytes from 10.244.0.193: seq=2 ttl=62 time=0.164 ms
64 bytes from 10.244.0.193: seq=3 ttl=62 time=0.238 ms
64 bytes from 10.244.0.193: seq=4 ttl=62 time=0.261 ms
^C
--- 10.244.0.193 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
```

```bash
➜  lab4 git:(main) ✗ k -n labs port-forward pods/react-app-f66d69d6b-6jdg5 3001:3000
Forwarding from 127.0.0.1:3001 -> 3000
Forwarding from [::1]:3001 -> 3000
```

![rack-1.png](screenshots%2Frack-1.png)

```bash
➜  lab4 git:(main) ✗ k -n labs port-forward pods/react-app-f66d69d6b-n9t2k 3002:3000
Forwarding from 127.0.0.1:3002 -> 3000
Forwarding from [::1]:3002 -> 3000
```

![rack-2.png](screenshots%2Frack-2.png)
