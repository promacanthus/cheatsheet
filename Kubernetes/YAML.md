## sample

```
apiVersion: v1
kind: Pod
metadata:
  name: pods-simple-pod
spec:
  containers:
    image: busybox
    name: pods-simple-container
    - command:
        - sleep
        - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

## debug network

[https://github.com/Praqma/Network-MultiTool](https://github.com/Praqma/Network-MultiTool)

```
apiVersion: v1
kind: Pod
metadata:
  name: debug-network-pod
  namespace: default
spec:
  containers:
    image: praqma/network-multitool
    name: debug-network-container
    - command:
        - sleep
        - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

## debug DNS

[https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

`kubectl exec -i -t dnsutils -- nslookup kubernetes.default`

`kubectl exec -ti dnsutils -- cat /etc/resolv.conf`

```
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3
    command:
      - sleep
      - "infinity"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
  
```

### custome dns

```
apiVersion: v1
kind: Pod
metadata:
  name: dns-config-dns-config-pod
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 1.2.3.4
    searches:
      - ns1.svc.cluster-domain.example
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

### DNS policy

[https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

```
apiVersion: v1
kind: Pod
metadata:
  name: dns-config-policy-pod
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: dns-config-policy-container
  hostNetwork: true
  dnsPolicy: ClusterFirstWithHostNet
 
```

### hostAlias

[https://kubernetes.io/docs/tasks/network/customize-hosts-file-for-pods/#adding-additional-entries-with-hostaliases](https://kubernetes.io/docs/tasks/network/customize-hosts-file-for-pods/#adding-additional-entries-with-hostaliases)

```
apiVersion: v1
kind: Pod
metadata:
  name: pods-host-aliases-pod
spec:
  hostAliases:
    - ip: "127.0.0.1"
      hostnames:
        - "foo.local"
        - "bar.local"
    - ip: "10.1.2.3"
      hostnames:
        - "foo.remote"
        - "bar.remote"
  containers:
    - name: cat-hosts
      image: busybox
      command:
        - cat
      args:
        - "/etc/hosts"
 
```

## Affinity

[https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: Exists
  containers:
    - command: ["sleep", "3600"]
      name: pod-node-affinity-container
      image: busybox
```