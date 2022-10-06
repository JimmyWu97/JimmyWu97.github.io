# MR计算分析数据时处理小文件

使用FileInputFomat下的CombineTextInputFomat实现来处理

处理机制：

​	1.用户设置切片的大小

​		![1665059798800](C:\Users\m1316\AppData\Roaming\Typora\typora-user-images\1665059798800.png)

​	2.虚拟过程：和切片大小进行比较，如果当前文件>设置的大小且小于2倍的设置的大小，以最大值切割一块；如果当前文件>设置的大小且小于2倍的设置的大小就一分为2

​	3.切片过程：根据虚拟以后的结果，把每个虚拟文件和设置的片大小进行比较，如果大于等于设置的大小就单独形成一个切片，否则就和下一个虚拟文件合并。

<img src="C:\Users\m1316\AppData\Roaming\Typora\typora-user-images\1665060384892.png" alt="1665060384892" style="zoom:50%;" />