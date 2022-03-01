### 参考：

## 使用
### 1. quickstart
```Python
import yappi
# clock_type = "cpu" or "wall"
yappi.set_clock_type("cpu")
yappi.start()
 
# run your code
 
# get stats for all threads
yappi.stop()
threads = yappi.get_thread_stats()
 
import pstats
total_pstats = pstats.Stats()
for thread in threads:
    print(
        "Function stats for (%s) (%d)" % (thread.name, thread.id)
    )  # it is the Thread.__class__.__name__
    # func stats   
    stats = yappi.get_func_stats(ctx_id=thread.id)
    stats.print_all()
    # convert to pstats.Stats Object
    pstats = yappi.convert2pstats(stats)
    total_pstats.add(pstats)
    print("====================")
total_pstats.dump_stats("all_threads.pstats")
```

### 2.分析
`yappi.set_clock_type`可以设置记录`cpu_time`还是`wall_time`
`yappi.get_thread_stats`会获取所有线程的统计信息
`yappi.get_func_stats`通过`ctx_id`参数，获取某个线程所有函数的统计信息。
