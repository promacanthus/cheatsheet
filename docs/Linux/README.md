将 `sudo` 放在脚本首行，以 root 身份运行整个脚本

```bash
#!/usr/bin/sudo /bin/bash
```

1. 列出排名前10的内存占用进程

```bash
ps aux | sort -rk 4,4 | head -n 10
```

2. 查看网卡实时流量

```bash
#!/bin/bash
NIC=$1
echo -e " In ------ Out"
while true; do
    OLD_IN=$(awk '$0~"'$NIC'"{print $2}' /proc/net/dev)
    OLD_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    sleep 1
    NEW_IN=$(awk  '$0~"'$NIC'"{print $2}' /proc/net/dev)
    NEW_OUT=$(awk '$0~"'$NIC'"{print $10}' /proc/net/dev)
    IN=$(printf "%.1f%s" "$((($NEW_IN-$OLD_IN)/1024))" "KB/s")
    OUT=$(printf "%.1f%s" "$((($NEW_OUT-$OLD_OUT)/1024))" "KB/s")
    echo "$IN $OUT"
    sleep 1
done
```

3. 查看服务器利用率

```bash
#!/bin/bash  
function cpu(){  
  
 util=$(vmstat | awk '{if(NR==3)print $13+$14}')  
 iowait=$(vmstat | awk '{if(NR==3)print $16}')  
 echo "CPU -使用率：${util}% ,等待磁盘IO相应使用率：${iowait}:${iowait}%"  
  
}  
function memory (){  
  
 total=`free -m |awk '{if(NR==2)printf "%.1f",$2/1024}'`  
    used=`free -m |awk '{if(NR==2) printf "%.1f",($2-$NF)/1024}'`  
    available=`free -m |awk '{if(NR==2) printf "%.1f",$NF/1024}'`  
    echo "内存 - 总大小: ${total}G , 使用: ${used}G , 剩余: ${available}G"  
}  
disk(){  

  
 fs=$(df -h |awk '/^\/dev/{print $1}')  
    for p in $fs; do  
        mounted=$(df -h |awk '$1=="'$p'"{print $NF}')  
        size=$(df -h |awk '$1=="'$p'"{print $2}')  
        used=$(df -h |awk '$1=="'$p'"{print $3}')  
        used_percent=$(df -h |awk '$1=="'$p'"{print $5}')  
        echo "硬盘 - 挂载点: $mounted , 总大小: $size , 使用: $used , 使用率: $used_percent"  
    done  
  
}  
function tcp_status() {  
    summary=$(ss -antp |awk '{status[$1]++}END{for(i in status) printf i":"status[i]" "}')  
    echo "TCP连接状态 - $summary"  
}  
cpu  
memory  
disk  
tcp_status
```
