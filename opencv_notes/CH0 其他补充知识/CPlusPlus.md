# CH0 其他补充知识

## 0.2 C++语言补充

#### 0.2.1 std::string类

#### 0.2.2 枚举类型enum

枚举类型可自动转换为int，但反过来不是自动的。

#### 0.2.3 main函数

​		main函数中的参数可以用来处理命令行选项。

```c++
int main(int argc, char *argv[]){}
```

​		如上所示，main函数有两个参数，其中，argc表示包含命令行选项的个数；argv则包含了argc个C风格字符串。例如，对于命令行`prog -d -o ofile data()`，如果argc设置为5，且

[c++ primer5 第305页](E:\学习资料\程序设计语言\C++\learing meterials\C++ primer中文版.pdf)





































vectorhttps://blog.csdn.net/weixin_40311211/article/details/81065786