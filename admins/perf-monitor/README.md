# Performance Monitoring 101
> * ***Credits to [Brendan Gregg](https://brendangregg.com) for his amazing work in this area.***
> * ***Please watch his videos, [Linux Performance Tools(1-2)](https://www.youtube.com/watch?v=D53T1Ejig1Q)
> for an in-depth description of these tools and best practices.***
> * ***These are just my reference points.***

---
## Perf Mon Methodologies
* Problem Statement Method
* Workload Characterization Method
* USE Method
* Off-CPU Analysis
* CPU Profile Method
* Active Benchmarking
* Static Performance

---
### Problem Statement Method
1. What makes you **think** there is a perfomance problem?
2. Hs the system **ever** performed well?
3. What **changed** recently? (software? hardware? load?)
4. Can the performance degradation be expressed in terms of **laency** or runtime?
5. Does the problem effect **other** people or applications (or just ticket submitter)?
6. What is the **environment**? Software, hardware, instance types? Versions? Configuration?

---
### Workload Characterization Method
1. ***Who*** is causing the load? *PID, UID, IP, addr, ...*
2. ***Why*** is the load called? *code path, stack trace*
3. ***What*** is the load? *IOPS, tput, type, r/w*
4. ***How*** is the load changing over time?

---
### The USE Method
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
| *BASIC* | DESCRIPTION |
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

---
| *INTERMEDIATE* | DESCRIPTION |
|:---:|:---|
| `strace` | Trace system calls and signals (lot of overhead) |
| `tcpdump` | Dump traffic on a network (poor scalability) |
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
> | `netstat` | -s=protocol_stats, -i=iface_stat, -r=route_table, -p=proc_details, -c=interval |
> | `pidstat -(t\|d)` | by_thread, disk_io |
> | `swapon -s` | show_swap_dev_usage *(if enabled)* |
> | `lsof -iTCP -sTCP:ESTABLISHED` | for some apps, this will return all active network connections |
> | `sar -n DEV` | archive: network_stats_on_dev |
> | `sar -n TCP,ETCP,DEV 1` | live_poll: network_stats_on_tcp/etcp/dev |

---
| *ADVANCED* | DESCRIPTION |
|:---:|:---|
| ***misc*** |
| `ltrace` | A library call tracer |
| `ss` | Another utility to investigate sockets |
| `iptraf-ng` | Histogram packet sizes gathered from iface network traffic |
| `ethtool` | Query or control network driver and hardware settings (mostly tuning) |
| `slabtop` | Display kernel slab cache information in real time |
| `pcstat` | Show page cache residency by file. (dbs monitoring) |
| `iotop` | Block device I/O (disk) by process |
| `/proc` | Many raw kernel counters |
| `blktrace` | Block I/O event tracer (apt install) |
| `snmpget` | SNMP network host stats (compile) |
| `lldptool` | Can get LLDP broadcast stats (compile) |
| ***CPU Performance Counters*** |
| `perf_events` | Performance analysis tools for Linux i.e. `perf` |
| `tiptop` | IPC by process %MISS, %BUS |
| `rdmsr` | Use `rdmsr` from the *msr-tools* package to read MSRs |
| `pmu-tools` | On-and-Off core CPU counter tools |
| ***Advanced Tracers*** |
| `ftrace` | # WHATIS? |
| `eBPF` | # WHATIS? |
| `SystemTap` | # WHATIS? |
| `ktap` | # WHATIS? |
| `LTTng` | # WHATIS? |
| `dtrace4linux` | # WHATIS? |
> | **--examples** | ***see man $cmd*** |
> |:---|:---|
> | `ss -mop` | sock_mem_usage, timer_info, proc_using_sock |
> | `ss -i` | internal_tcp_info |
> | `iptraf-ng` | Install from apt sources |
> | `iotop` | Needs kernel support enabled: CONFIG_TASK_IO_ACCOUNTING |
> | `pcstat` | Download/clone from github |
> | `perf_events` | Usually added by 'linux-tools-common' (includes `perf`) |
> | `tiptop` | May be a wild ride compiling |
> | `rdmsr` | Model Specific Registers(MSRs), unlike PMCs, can be read by default in Xen guests |

---
## Benchmarking Tools
>
> * The conclusions gathered from benchmarking must be used with caution.
> * Poor benchmarking methods result in arbitrary results interpretted arbitrarily.
> * Proper scientific methodologies must be applied in both trial procedures and pre/post analysis.
> 
> * This still doesnt account for load surges, latency, crashes, etc, etc, etc
>   * Benchmarking works for comparing new hardware
>   * Dont consider this an umbrella for software
>   * Certainly dont try this in production if you value your job
>   * See Brendans videos linked above for detailed instructions on how to apply these tests.
>
| Tool | Description |
|:---:|:---|
|***Multi***|
| `UnixBench` | get source |
| `lmbench` | apt upstream |
| `sysbench` | apt upstream |
| `perf bench` | General framework for benchmark suites |
|***FS/Disk***|
| `dd` | disk destroy? |
| `hdparm` | get/set SATA/IDE device parameters |
| `fio` | apt upstream |
|***App/lib***|
| `jmeter` | apt upstream |
| `openssl` | OpenSSL command line tool |
| `ab` | # WHATIS? |
| `wrk` | # WHATIS? |
|***Networking***|
| `ping` | Send ICMP ECHO_REQUEST to network hosts
| `hping3` | apt upstream |
| `iperf` | apt upstream |
| `ttcp` | # WHATIS? |
| `traceroute` | apt upstream |
| `mtr` | A network diagnsostic tool |
| `pchar` | apt upstream |

---
## Tuning Tools
| Tool | Description |
|:---:|:---|
|***Generic Interfaces***|
| `sysctl` | Configure kernel parameters at runtime |
| `/sys` |
|***Applications***|
| `app.conf` | use the applications own configurations |
|***CPU/Scheduler***|
| `nice` | Run a program with modified scheduling priority |
| `renice` | Alter priority of running processes |
| `taskset` | Set or retrieve a process's CPU activity |
| `ulimit` | Get and Set user limits |
| `chcpu` | Configre CPUs |
|***Storage***|
| `tune2fs` | Adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems |
| `ionice` | Set or Get process I/O scheduling class and priority |
| `hdparm` | get/set SATA/IDE device parameters |
| `blockdev` | Call block device ioctls from the command line |
|***Network***|
| `ethtool` | Query or control network driver and hardware settings |
| `tc` | Show/manipulate traffic control settings |
| `ip` | Show/manipulate routing, network devices, interfaces and tunnels |
| `route` | Show/manipulate the IP routing table |
|***Dynamic***|
| `patching` | Appl a diff file to an original? |
| `stap` | # WHATIS? |
| `kpatch` | # WHATIS? |

---
## Static Performance Tuning
*"Ask what can be checked on the system without load?"*
* Methodology by Richard Eiling (2000)

* Check the static state and configuration of the system:

| **Parameter** | Command |
|:---|:---:|
| `cat /proc/cpuinfo` | CPU types & flags |
| `cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_{available_frequencies,governor}` | CPU frequency scaling config |
| `cat proc/scsi/scsi` | Storage devices |
| `cat proc/scsi/scsi` | File system capacity |
| `isscsi` | File system and volume configuration |
| `netstat -rn` or `ip route get ip_addr` | Route table |
| `?` | State of hardware |
| `dmesg` | System messages |
| `ifconfig -a; ip link` | Network interface config |
| `df -h` | File system capacity |
| `mdadm --misc -D /dev/md0` | Volume config |
| `smartctl` | Storage device info |
| `numactl -s; numactl -H` | NUMA Config |
| `ispci` | PCI info | 
| `lsmod` | Installed kernel modules |
| `crontab -l` | Root crontab config |
| `services --status-all` | Services |
