- 向量，类似于数组，但可以动态增长。头文件 `<vector>`
- 是一个类模板，实例化产生一个类，如 `vector<int>` 产生一个数据元素是 `int` 的 `vector<int>` 类（向量）。
- 同样，可以通过 `vector<int>` 类对象去访问其成员，如成员函数。
- 同样可以用运算符进行一些运算。

vector是动态空间，可以动态扩展
动态扩展：并不是在原空间后接新空间，而是会创建一个更大的内存空间，然后将原有的数据拷贝放入新的内存空间，释放原空间(==扩展后的所有地址会变==)

```C++
#include<iostream>  
#include<vector>  
using namespace std;  
  
int main(){  
    vector<int> arr ={1, 2, 3 ,4 ,5};  
    arr.push_back(6);  
    for(int i =0;i<arr.size();i++){  
        cout << arr[i] << " ";  
    }  
    cout << endl;  
    arr.pop_back();  
    for(int i =0;i<arr.size();i++){  
        cout << arr[i] << " ";  
    }  
}
```
