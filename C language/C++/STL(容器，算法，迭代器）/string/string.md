C++相较于C增加了字符串类型，可以用string类来直接定义字符串，而不用像C一样创建数组了
并且这个字符串类可以直接做到拼接和处理

C++提供了直接的字符串方法，并且提供了很多方法，方便我们去使用字符串，具体例子如下

```C++
#include<iostream>  
#include<string>  
using namespace std;  
  
int main(){  
    // 两种赋值方法  
    string s = "hello";  
    string s2("World");  
    
    //string类的方法  
    cout << s.size() << endl;  
    string s3 = s.substr(1,3);  
    cout << s3 << endl;  
    
    //字符串的拼接，和python差不多  
    string s4 = s + " " + s2;  
    cout << s4 << endl;  
    
    //字符串的替换  
    s4[0] = 'H';  
    s4[6] = 'X';  
    cout << s4 << endl;  
    
    //子串的查找(会返回第一个字符的位置)
    int pos = s4.find("orl");  
	cout << pos << endl; 
	
	//字符串的插入(例子是3，会在index为2的后面插入这个字符串)
	s4.insert(3, "ABCD");  
	cout << s4 <<endl;
	
	//间隔“-”的方法
	for(int i=0;i<s4.size();i++){  
    cout << s4[i] << "-";  
	}  
	cout << endl;
	
	
    return 0;  
}
```


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
