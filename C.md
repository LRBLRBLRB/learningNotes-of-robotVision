# CH0 其他补充知识
## C语言补充
#### 1、C的输入命令
#### 2、C的输出命令
###### (1)printf()
​		首先，printf()函数打印数据的指令要与待打印数据的类型相匹配。
![printf()转换说明](E:\z-lab\openCV_tutorials\opencv_notes\printf()转换说明.jpg)
​		函数的格式是——`printf(格式字符串, 待打印项1, 待打印项2,...);`其中，待打印项可以是变量常量表达式等等。例如：

```c
int number = 7;
float pies = 12.75;
printf("The %d contestants ate %f berry pies.\n", number,
pies);
//output: The 7 contestants ate 12.750000 berry pies.

printf("The value of pi is %f.\n", PI);
//putput: The value of pi is 3.141593.

printf("Farewell! thou art too dear for my possessing,\n");
//output: Farewell! thou art too dear for my possessing,

int cost = 7800;
printf("%c%d\n", '$', 2 * cost);
//output: $15600
```

​		printf()转换说明的修饰符![printf()转换说明的修饰符](E:\z-lab\openCV_tutorials\opencv_notes\printf()转换说明的修饰符.jpg)
​		printf()的标记![printf()的标记](E:\z-lab\openCV_tutorials\opencv_notes\printf()的标记.jpg)

###### (2)sprintf()

​		该函数和printf()类似，但是它是把数据写入字符串，而不是打印在显示器上。

​		因此，sprintf()的第1个参数是目标字符串的地址。其余参数和printf()相同，即格式字符串和待写入项的列表。










