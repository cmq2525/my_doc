```bash
#!/bin/bash
set -x

## $1 ：c头文件路径
## $2 ： so路径
## $3 ：保存为.py文件路径
# example: scripts/gen_py_header.sh ./c_headers/megvideo_c.h ./lib/libmegvideo.so ./pymegvideo/megvideo_c.py
 
c_header_path=`realpath $1`
lib_path=`realpath $2`
output_path=`realpath $3`
base_name=`basename $output_path`
 
repo_path=$PWD
echo "c_header_path: ${c_header_path}\n lib_path:${lib_path}\n output_path:${output_path}\n repo_path:${repo_path}"
workspace=${repo_path}
 
# update ctypesgen
git submodule --update
 
cp $c_header_path ctypesgen/generate/
pushd $workspace/ctypesgen/generate
../run.py -o $base_name --output-language=py32 -l $lib_path $c_header_path
cp ./$base_name $output_path
popd
```

### 2.cffi
参考：
1.官方文档：https://cffi.readthedocs.io/en/latest/index.html
2.例子：https://stackoverflow.com/questions/53471742/python-build-cffi-in-api-mode-with-setuptools


1.简介
将所有依赖打成一个`_[name]_abi3.so`作为接口，把它放到源码目录下，即可调用c程序。

例子：
build_foo.py
```Python
#!/usr/bin/env python3
import cffi
 
ffi = cffi.FFI()
ffi.cdef("int get_one();")
ffi.set_source("_one_cffi",
"""
int get_one() {
    return 1;
}
 
"""
)
 
if __name__ == '__main__':
    ffi.compile(verbose=True)
```
直接`python3 build_foo.py`运行后，会生成`_one_cffi.cpython-37m-x86_64-linux-gnu.so`（因为没用setup.py指定包名）。

使用：
```Python
from _one_cffi import lib
lib.get_one()
```

2.打包
打包时，只需要在setup.py中的cffi_modules项指定build_xxx.py的路径即可：
```Python
from setuptools import setup
setup(
    name = 'Foo',
    version = '0.1',
    packages = ['foopkg'],
    cffi_modules=["foopkg/build_foo.py:ffi"],
    install_requires = ['cffi']
)
```