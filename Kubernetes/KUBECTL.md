# 命令速查

## API

1. `kubectl get <xxx> -v=9`

## 集群

1. 查看集群版本：`kubectl version`
2. 查看集群信息：`kubectl cluster-info`
3. 代理 API Server：`kubectl proxy -p 8080 --keepalive 3600s --reject-paths='' -v=9`

## Pod

1. 查看 Pod 状态：

1. `kubectl -n <namespace> get pod <pod-name> -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'`
2. `kubectl -n <namespace> get pod <pod-name> -o json | jq '.status.ccondtions[] | select(.type == "Ready") | .status'`

2. 查看 Pod 事件：`kubectl -n <namespace> get events --field-selector involvedObject.name=<pod-name>`
3. 查看 Pod 的 IP：

1. `kubectl -n <namespace> get pod <pod-name> -o custom-columns=POD:metadata.name,IP:status.podIP --no-headers`
2. `kubectl -n <namespace> get pods -o json | jq '[.items[] | {POD: .metadata.name, IP: .status.podIP}]'`

4. 查看 Pod 状态：

1. `kubectl -n <namespace> get pod <pod-name> -o jsonpath='{.status.conditions[?(@.type=="Ready")]}.status'`
2. `kubectl -n <namespace> get pod <pod-name> -o json | jq '.status.conditions[] | select(.type=="Ready") | .status'`

5. 验证 Pod 安全上下文：`kubectl auth can-i list pods --as=system:serviceacount:<namespace>:<serviceaccount-name>`
6. Pod 亲和性：

1. `kubectl -n <namespace> get pod <pod-name> -ojsonpath='{.spec.affinity}'`
2. `kubectl -n <namespace> get po <pod-name> -ojson | jq '.spec.affinity'`

7. 查看 Pod 开销：

1. `kubectl -n <namespace> get pod <pod-name> -ojsonpath='{.spec.overhead}'`
2. `kubectl -n <namespace> get pod <pod-name> -ojson | jq '.spec.overhead'`

8. 删除指定 Pod：

1. `kubectl get pods --filed-selector= status.phase=Pending -A -ojson | jq '.item[] | "kubectl -n \(.metadata.namespace) get pods \(.metadata.name)"' | xargs -n 1 bash -c`
2. `kubectl -n default get pods | grep Completed | awk '{print $1}' | xargs kubectl -n default delete pods`

## Event

1. 按时间排序：`kubectl get events --sort-by=.metadata.creationTimestamp`

## Deployment

1. 查看滚动发布状态：`kubectl -n <namespace> rollout status deployment <deployment-name>`
2. 查看滚动发布历史：`kubectl -n <namespace> rollout history deployment <deployment-name>`
3. 扩缩：`kubectl -n <namespace> scale deployment <deployment-name> --replicas=<replica-count>`
4. 自动扩缩：`kubectl -n <namespace> autoscale deployment <deployment-name> --min=<min-pods> --max=<max-pods> --cpu-percent=<cpu-percent>`

## Node

1. 运行在特定节点上的 Pod：`kubectl -n namespace get po --field-selector spec.nodeName=<node-name>`
2. 清空节点：`kubectl drain <node-name> --ignore-daemonsets`
3. 状态检查：`kubectl desribe node <node-name> | grep Conditions -A5`
4. 列出节点容量和可分配资源：`kubectl desribe node <node-name> | grep -E "Capacity|Allocatable"`
5. 查看节点的污点：`kubectl describe node <node-name> | grep Taints`
6. 查看所有节点状态：`kubectl get nodes -o custom-columns='NODE:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status'`

## PV

1. PV 按容量排序：`kubectl get pv --sort-by=.spec.capacity.storage`

## 资源使用

1. Pod：`kubectl -n <namespace> top pod <pod-name>`
2. Node：`kubectl top node`

## 网络

1. 启动临时容器（1.18+）：`kubectl -n <namespace> debug -it <pod-name> --image=<debug-image> --/bin/sh`
2. 启动`busybox`：`kubectl run -it --rm --restart=Never --image=busybox net-debug-pod --/bin/sh`
3. 测试 HTTP 请求：`kubectl -n <namespace> exec -it <pod-name> -- curl <endpoint-url>`
4. 跟踪网络路径：`kubectl -n <namespace> exec -it <pod-name> -- tracetroute <destination-pod-ip>`
5. DNS 解析：`kubectl -n <namespace> exec -it <pod-name> -- nslookup <domain-name>`
6. 查看 Pod DNS 配置：`kubectl -n <namespace> exec -it <pod-name> -- cat /etc/resolv.conf`