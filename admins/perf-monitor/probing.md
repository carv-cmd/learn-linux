# Advanced Monitoring

## Tracing
* Statically placed logical points in the kernel to track.
* Provides key event details as a 'format' string.

### Subsystem Tracing
| ***TOOL*** | ***DESCRIPTION*** |
|:---|:---|
| `syscalls` | System calls interface |
| `ext4` | File systems |
| `block` | Block device interface |
| `sock` | Sockets |
| `sched` | Scheduler |
| `vmscan` | Virtual memory |
| `kmem` | Virtual memory |
| `scsi` | Device drivers |
| `irq` | Device drivers |

### Choosing a Tracer
* Study what Linux already has built in:
   * `perf_events`
   * `ftrace`
   * `eBPF`
   * etc

*Use this small python sample to help pick tracer from:*
* `ftrace`, `perf_events`, `eBPF`, `SystemTap`, `LTTng`, `ktap`, `dtrace4linux`, `OEL DTrace`, `sysdig`
```python
if builtins == not_sufficient:
   return (SystemTap, LTTng)
else
   if uneed == [live_tracing, counting]:
      return (ftrace, perf_events)
      
   if uneed == [PMCs, stack_profiling, trace_dump_analyze]:
      return perf_events
      
   if uneed == [in_kernel_summaries]:
      return eBPF
```

### Tracing Frameworks +probes
* For dynamic:
  * Kernel tracing (function_calls, returns, numbers): `krpobes`
  * UserLevel tracing: `uprobes`

### Tracer Tools
| ***[Perf-Tools](https://github.com/brendangregg/perf-tools)*** | ***DESCRIPTION*** |
|:---|:---|
||
| ***single-purpose*** |
| `iosnoop` | Block I/O (disk) events with latency |
| `iolatency` | Block I/O (disk) latency distributions |
| `opensnoop` | Trace `open()` syscalls showing filenames |
| `tpoint` | Who's creating disk I/O, and what type? (Trace tracepoint) |
||
| ***multi-purpose*** |
| `funccount` | Count a kernel function call rate |
| `funcgraph` | Trace a graph of kernel code flow |
| `kprobe` | Dynamically trace a kernel function call or return, with variables, and in-kernel filtering |

---
## perf_events: Workflow

| **COMMAND LINE** | **DESCRIPTION** |
|:---|:---|
| ***cmds-to:*** | ***gather-perf-data*** |
| `perf list` > `perf.data` | List events |
| `perf stat` > `perf.data` | Count events |
| `perf record` > `perf.data` | Capture stacks |
||
| ***cmds-to:*** | ***get-quick-view*** |
| `perf report perf.data` | Text UI |
||
| ***cmds-to:*** | ***make-flamegraphs*** |
| `perf script perf.data` > `dump.profile` | Preparse perf.data for collapse |
| `stackcollapse-perf.pl dump.profile` > `collapsed.profile` | Format for flamegraph input |
| `flamegraph.pl collapsed.profile` > `flamegraph.svg` | Generate flamgraph in svg format|
