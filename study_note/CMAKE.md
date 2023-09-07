
[[CMake] cmake 入门: 调用多个目录下的源文件\_cmake 引用另外一个源码目录\_刘好念的博客-CSDN 博客](https://blog.csdn.net/Strengthennn/article/details/104540673)

[CMake学习(2)--添加头文件编译 - 知乎](https://zhuanlan.zhihu.com/p/402118882)
### include_directories 与 target_include_directories 区别
include_directories 会为当前 CMakeLists.txt 的所有目标，以及之后添加的所有子目录的目标添加头文件搜索路径。因此，慎用 target_include_directories，因为会影响全局 target。
target_include_directories 只会为指定目标包含头文件搜索路径。如果想为不同目标设置不同的搜索路径，那么用 target_include_directories 更合适。