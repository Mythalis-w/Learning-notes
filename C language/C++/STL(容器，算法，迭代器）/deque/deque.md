**功能：**

- 双端数组，可以对头端进行插入删除操作

**deque与vector区别：**

- vector对于头部的插入删除效率低，数据量越大，效率越低(移动元素的方法)
- deque相对而言，对头部的插入删除速度会比vector快
- vector访问元素时的速度会比deque快，这和两者内部实现有关

![[depue.png]]

deque相较于vector多了头部的插入和删除

构造方法和vector类似`deque<int> arr`就可以，同样需要deque头文件
拷贝的方法依旧与vector类似`deque<int> temp(arr)`就可以了