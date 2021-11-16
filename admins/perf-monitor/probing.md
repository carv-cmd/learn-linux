# Advanced Monitoring
> * ***See README for credits.***
> * ***This reference doc includes 
> [Brendan's](https://www.brendangregg.com/overview.html)
> visualization software
> [FlameGraphs](https://github.com/brendangregg/FlameGraph).***
> * ***Again please visit those links for detailed explanations of these tools and methodologies.***
---

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

---
### Choosing a Tracer

*Use this small python sample code to help pick a tracer:*
```python
# First study what your Linux distro already has builtin/upstream.
# Again see Brendans work. This is my reference.
tracers = {
    'builtins': ['ftrace', 'perf_events', 'eBPF', 'sysdig'], 
    'compiled': ['SystemTap', 'LTTng', 'ktap', 'dtrace4linux', 'OEL DTrace']
    }

if builtins not sufficient_for_needs:
   return SystemTap, LTTng
else
   if needs in ['Live-Tracing', 'Counting']:
      return ftrace, perf_events
      
   if needs in ['PMCs', 'Stack-Profiling', 'Trace-Dump-Analyze']:
      return perf_events
      
   if needs in ['In-Kernel-Summaries']:
      return eBPF
```
* `ftrace`, `perf_events`, `eBPF`, `SystemTap`, `LTTng`, `ktap`, `dtrace4linux`, `OEL DTrace`, `sysdig`

### Tracing Frameworks +probes
* For dynamic...
  * Kernel tracing (function_calls, returns, numbers): `krpobes`
  * UserLevel tracing: `uprobes`

---
### Tracer Tools (authored by Brendan Gregg)
| ***Perf-Tools*** | *Full details @ his [GitHub](https://github.com/brendangregg/perf-tools)* |
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

### [eBPF](https://ebpf.io/projects)
* Extended BPF: Programs on tracepoints
  * High-performance filtering: JIT
  * In-kernel summaries: maps
  * Developed by **Alexei Starovoltov**


---
## [perf_events](https://www.brendangregg.com/perf.html): Workflow

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

