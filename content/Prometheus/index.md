+++
title = "Prometheus"
+++


- [kubelet 上报的指标](https://github.com/kubernetes/kube-state-metrics/blob/main/docs/pod-metrics.md)
- [cAdvisor 上报的指标](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md)
- <https://samber.github.io/awesome-prometheus-alerts/rules#host-and-hardware>

## PromQL

### Node

#### CPU 使用率

```sql
(1- sum(rate(node_cpu_seconds_total{mode="idle"}[1m])) by (instance) / sum(rate(node_cpu_seconds_total[1m])) by (instance)) *100
```

#### 内存使用率

```sql
(1-(node_memory_Buffers_bytes + node_memory_Cached_bytes + node_memory_MemFree_bytes )/node_memory_MemTotal_bytes)*100
```

#### 磁盘容量

#### 磁盘 IO

### Pod
