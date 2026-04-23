
# 1.基础代码 

C++的环境可以完美兼容C语言的代码，只不过一些头文件要改一改(不改在一些编译器也能通过)

==cin完全支持 中间有空格或者换行符 的输入==
```c++
#include<iostream>  // input output 数据流 

using namespace std;  // 使用std这个名称空间

int main(){
	int n;
	cin >> n;  // cin其实就是scanf，在std这个名称空间里 >>指的输入到n中，如果像输入两个，就通过链式输入cin >> a >> b; 
	cout << "111" << ++n << endl  // cout就是printf，<<表示输出，多个<<是多个输出，其中的endl是"\n"，改成"\n"的效果是完全一样的
	return 0;
}
```

如果你没有`using namespace std` ，你可以如下操作
```c++
#include<iostream>  // input output 数据流 

int main(){
	int n;
	std::cin >> n;  // cin其实就是scanf，在std这个名称空间里 >>指的输入到n中
	std::cout << "111" << ++n << endl  // cout就是printf，<<表示输出，多个<<是多个输出，其中的endl是"\n"，改成"\n"的效果是完全一样的
	return 0;
}
```

==注意，cin 和 cout 的速度远远不如 scanf 和 printf==

## cout 对 char* 类型的特殊重载

```C++
#include<iostream>  
using namespace std;  
  
int main()  
{ char a[10]={'1','2', '3', '4', '5', '6', '7', '8', '9',0} ,*p;  
  
    int i;  
    i=8;  
    p=a+i;  
    cout<<(void*)p-3;  
    return 0;  
}
```
如上，输出结果并不是‘6’的地址，而是一个字符串，"6789"，
# 2. 头文件的调用

在C++中，头文件可以像C那样去调用，即`#include<stdio.h>` ，但是对于标准的C++，我们只需要把后缀删除加上一个c就可以了，如下 `#include<cstdio>` ，这样你就可以用`printf` 和 `scanf` 了

所有C的头文件在C++都能调用

# 3. string类

C++相较于C增加了字符串类型，可以用string类来直接定义字符串，而不用像C一样创建数组了
并且这个字符串类可以直接做到拼接和处理
在某一些编译器中，有的不用string类，因为iostream中可能包含，但是最好是显式表示string类，因为C++中含有string这个C++标准库，不用cstring

```c++
#include<iostream>  
#include<string>

using namespace std;  
  
int main(){
	//定义  
    string s1 = "Hello";   
    string s2 = "World!";  
	//拼接
    string s = s1 + " " + s2;  
	//输出与输入（cin可以直接输出字符串）
    cout << s << endl;  
	
    return 0;  
}
```

注意：cin无法输入一个句子（带空格的字符串）， 这时候可以用getline来输入

```c++
#include<iostream>  
#include<string>

using namespace std;  
  
int main(){  
    string s;  
    getline(cin , s);  // s就是定义的string 
    cout << s << endl;  
  
    return 0;  
}
```

我们将类中的函数定义为方法，通过`类.函数()`去调用，如`string.length()`

```c++
#include<iostream>  
#include<string>  
  
using namespace std;  
  
int main(){  
    string s;  
    getline(cin , s);  
    cout << s.length() << endl;  // 这里s表示的是字符串的类里面的length方法
  
    return 0;  
}
```

并且字符串中的值可以通过索引去修改，`s[2] = 'F'`是合法的
# 4.结构体

C++的结构体和C大致相同，但是还是有一些区别的，即在创建结构体的时候不用加上struct了

```c++
#include<iostream>  
#include<string>  
  
using namespace std;  
  
struct stu {  
    int age;  
    string name;  
};  
  
int main(){  
    stu a[1000];  //！！！
    return 0;  
}
```

# 5. for的新写法以及新的类型

C++中添加了**基于范围的 for 循环**，`auto`类型可以自己判断类型
```C++
int arr[] = {10, 20, 30, 40, 50};

for (auto x : arr) { //最基本的用法（遍历并打印）
    cout << x << " ";
}
// 输出：10 20 30 40 50 , x是arr的副本

for (auto &x : arr) {
    x = x * 2;  // 修改x会直接修改原数组元素
}
// arr 变为 {20, 40, 60, 80, 100}
```

## 迭代器循环

```C++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 50};
    
    // 基本迭代器循环
    for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    // 输出：10 20 30 40 50
    
    return 0;
}
```
it是迭代器的名字 
在更高的版本，迭代器的生成可以用auto去生成
```C++
    // auto 自动推导迭代器类型
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
```
迭代器的标准声明是`std::STL声明::iterator` ，也可以用`auto`
# 6. C++中的迭代器(**iterator**)

C++的**迭代器是一个可以‘一个一个遍历容器元素’的对象，它像==指针==一样指向容器中的元素，通过递增操作移动到下一个元素。"** 指针的操作完全适用于迭代器

所有的STL容器都是可迭代对象，而可迭代对象可以产生迭代器

## 迭代器的方法

一般来说，所有的迭代器都有一些固定的方法，如：begin，end方法等

![[C转C++基础知识.png]]

我们主要拿这个图去关注begin end rbegin rend的位置关系，如图前文写的那些函数都是一个迭代器，注意end的位置是在所有空间的后面的一个空间。

# 7.类型转换运算符

```C++
// 把一种类型"变成"另一种类型
//static_cast<目标类型>(表达式)

int num = 2;
Weekday day = static_cast<Weekday>(num);  // 把int转成Weekday
// day 现在是 WEDNESDAY（因为2对应周三）
```
