### 参考
1.[C++Profile的大杀器_gperftools的使用](https://xusenqi.site/2020/12/06/C++Profile%E7%9A%84%E5%A4%A7%E6%9D%80%E5%99%A8_gperftools%E7%9A%84%E4%BD%BF%E7%94%A8/)

## 使用
### 1.简单程序示例
```c++
// test.cpp
#include <gperftools/profiler.h>
#include <iostream>
using namespace std;
void func1() {
   int i = 0;
   while (i < 100000) {
       ++i;
   }
}
void func2() {
   int i = 0;
   while (i < 200000) {
       ++i;
   }
}
void func3() {
   for (int i = 0; i < 1000; ++i) {
       func1();
       func2();
   }
}
int main(){
   ProfilerStart("test.prof"); // 指定所生成的profile文件名
   func3();
   ProfilerStop(); // 结束profiling，将数据写入文件
   return 0;
}
```
编译：
```bash
g++ test.cpp -lprofiler -o main
```
运行：
```bash
# 生成test.prof文件
./main
```
生成pdf：
```bash
pprof --pdf main test.prof > test.pdf
```

### 2.在bazel程序中使用pprof
由于代码中需要
`#include <gperftools/profiler.h>`，而bazel默认不会从`/usr/local/include/`目录下读取头文件，且BUILD中无法使用全局路径。

因此，需要使用[new_local_repository](https://docs.bazel.build/versions/main/be/workspace.html#new_local_repository)

在`WORKSPACE`中写入如下字段：
```WORKSPACE
new_local_repository(
    name = "system_libs",
    path = "/usr/local/",
    build_file_content =
 
"""
cc_library(
    name = "profiler",
    srcs = glob(["lib/libprofiler.so"]),
    hdrs = glob(["include/gperftools/*.h"]),
    # 声明include路径
    includes = ["include"],
    visibility = ["//visibility:public"],
)
""",
)
```
解释：

将/usr/local目录看成一个第三方repo，增加target：profiler。

在`BUILD`中以如下方式使用profiler依赖：
```
cc_megvii_binary(
    name = "test-memcpy-perf",
    srcs = glob([
        'test-memcpy-perf.cpp',
    ]),
    internal_deps = [
        "@system_libs//:profiler"
    ],
    internal_linkopts = if_cuda(["-lcudart"]),
    copts = common_options + ["-Icmq-test/"] + ["-lprofiler"]
)
```