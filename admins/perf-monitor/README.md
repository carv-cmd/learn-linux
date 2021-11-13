# Performance Monitoring 101
---
> * ***Credits to [Brendan Gregg](https://brendangregg.com) for his amazing work in this area.***
> * ***Please watch his video, [Linux Performance Tools(1-2)](https://www.youtube.com/watch?v=D53T1Ejig1Q)
> for an in-depth description of these tools and best practices.***
> * ***These are just my reference points.***

---
## Workload Characterization Method
1. ***Who*** is causing the load? *PID, UID, IP, addr, ...*
2. ***Why*** is the load called? *code path, stack trace*
3. ***What*** is the load? *IOPS, tput, type, r/w*
4. ***How*** is the load changing over time?

---
## The USE Method
*"Start with questions, then find the tools"*

* If possible, create a block diagram detailing: 
  * Host **system/software/environment** with **all associated resources**.
 
* For these resources, check the following:
  1. ***Utilization*** - busy time
  2. ***Saturation*** - queue length or queued time
  3. ***Errors*** - easy to interpret (objective)

---
## Tool Types
| Type | Characteristic |
|---|---|
| Observability | Watch activity. Usually safe (dependent on resource overhead) |
| Benchmarking | Load test: Use caution w/ production tests (contention issues) |
| Tuning | Changes can hurt perf!!! (apparent now or later under load...) |
| Static | Check configuration: Should be safe. |

---
## Observability Tools
| *BASIC* | Description |
|:---:|:---|
| `uptime` | Tell how long the system has been running |
| `top` | Display Linux processes. (or `htop`) |
| `ps` | Report a snapshot of the current processes |
| `vmstat` | Report virtual memory statistics |
| `iostat` | Report CPU stats and I/O stats for devices and partitions |
| `mpstat` | Report processors related stats. Look for unbalanced workloads, hot CPU's. |
| `free` | Display amount of free and used memory in the system |
> | **--examples** | ***see man $cmd*** |
> |:---|:---|
> | `ps -ef f` | ASCII Art Forest. Show parent child relationships. |
> | `vmstat -Sm 1` | define_display_unit, slabinfo |
> | `iostat -xmdz 1` | extended_stats, mbps_format, dev_utilization, omit_sleeping_devs |
> | `mpstat -P ALL 1` | cpu_cores_to_monitor |
 
| *INTERMEDIATE* | Description |
|:---:|:---|
| `strace` | Trace system calls and signals |
| `tcpdump` | Dump traffic on a network |
| `netstat` | Print network conns, routing tables, interface stats, masquerade conns, and multicast memberships |
| `nicstat` | Print network traffic statistics (`enicstat`) |
| `pidstat` | Report statistics for Linux tasks |
| `swapon` | enable/disable(`swapoff`) devices and files for paging and swapping |
| `lsof` | List open files |
| `sar` | Collect, report, or save system activity information |
> | **--examples** | ***see man $cmd*** |
> |:---|:---|
> | `strace -tttT -p 313` | time_since_epoch, syscall_time(s), pid |
> | `TMPF=/tmp/out.tcpdump;` | `tcpdump -i eth0 -w $TMPF -Z root; tcpdump -nr $TMPF \| head; rm $TMPF` |
> | `sar -n DEV 1` | report_network_stats_on_keyword |


---
## Benchmarking Tools
| Tool | Description |
|---|---|

---
## Tuning Tools
| Tool | Description |
|---|---|

---
## Static Tools
| Tool | Description |
|---|---|






