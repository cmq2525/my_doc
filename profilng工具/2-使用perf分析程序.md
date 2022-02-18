### 参考：
1.[官网](https://perf.wiki.kernel.org/index.php/Main_Page)
2.[如何读懂火焰图？](https://www.ruanyifeng.com/blog/2017/09/flame-graph.html)

## 使用
### 1.使用perf record记录各函数执行时间
```bash
# 会生成perf.data
# -F指定采样周期，单位是HZ，
# 指定栈大小为dwarf mode，
sudo perf record -F 99 --call-graph dwarf [$CMD]
# 解析采样数据，转换为.unflod
perf script -i perf.data > perf.unfold
# 使用FlameGraph（里面的perl脚本）将结果转为svg格式
git clone https://github.com/brendangregg/FlameGraph
cd FlameGraph
# 将unflod的符号进行折叠
./stackcollapse-perf.pl ../perf.unfold > perf.fold

# 生成svg格式的火焰图
./flamegraph.pl perf.fold > perf.svg
```