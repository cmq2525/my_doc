
## FlameGraph
[github地址](https://github.com/brendangregg/FlameGraph)

功能：

将栈信息展开，并基于此绘制火焰图。

不止用于perf保存的信息，也可以用于`jstack (Java stack)`, `vsprof (Visual Studio Stack)`,甚至`gdb`等

使用：
1. perf record调用栈
```bash
# 首先使用perf record命令存储调用栈
perf record -F 99 -a -g -- sleep 60
perf script -i perf.data > perf.unfold
# 展开调用栈
./stackcollapse-perf.pl perf.unfold > perf.fold
./flamegraph.pl perf.fold > perf.svg
```

## gprof2dot
[github地址](https://github.com/jrfonseca/gprof2dot)

功能：

将其他profiling工具存储的二进制数据绘制成dot格式的调用图。

使用：

1.perf格式
```bash
perf script -i perf.data > perf.unfold
python3 -m gprof2dot -f perf perf.unfold | dot -Tpng -o output.png
```
2.pstats格式（Python）
```bash
python3 -m gprof2dot -f pstats output.pstats | dot -T png -o output.png
```
