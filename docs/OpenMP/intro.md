# 绪论

OpenMP:编写多线程应用程序的API

## 核心语法

- OpenMP中的大多数构造都是编译器指令。`#pragma omp construct [clause [clause]…]`
- 文件中的函数原型和类型在 `omp.h`
- 大多数OpenMP*构造适用于“结构化块”
    - 结构化块：一个或多个语句的块，顶部有一个入口点，底部有一个出口点。
    - 在结构化块中有一个exit()是可以的。