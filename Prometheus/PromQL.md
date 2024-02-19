## Node
### CPU 使用率
```
(1- sum(rate(node_cpu_seconds_total{mode="idle"}[1m])) by (instance) / sum(rate(node_cpu_seconds_total[1m])) by (instance)) *100
```
### 内存使用率
```
(1-(node_memory_Buffers_bytes + node_memory_Cached_bytes + node_memory_MemFree_bytes )/node_memory_MemTotal_bytes)*100
```
### 磁盘容量
### 磁盘 IO
## Pod
