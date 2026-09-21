map作为一个STL容器，它可以动态扩充空间，比如你用数组开了10000个空间，但是只用了10个，剩下的全浪费了，但是map可以避免这个情况
`#include<map>`引入

用python类比，就是python的字典

**map是红黑树实现的字典**，但是vector是动态数组

```C++
map<int,int>mp; //第一个位置相当于放索引（信息），第二个位置放需要拿到的东西
mp[10001] = 82; //这个就点就创建成功了
cout<<mp[10001] //这个就可以输出82
/*
此时只开了10001这个空间，1-10000都是没有开的，很省空间啊
*/

if(mp.find(1000) != mp.end()){
	cout<<mp[10005]<<endl;
}
else{
	cout<<"no key"<<endl
}
```

map还能开一个不同的键值对`map<string,int> mp`也可以定义

map的遍历，map在遍历的时候会按照字典序去排序，就是按照key的大小从小到大去排序，string类型就是首字母（ASCII）排序了
```C++
for(auto it:mp){
	cout<<it.first<<' '<<it.second<<endl;
}//通过迭代器的形式就可以输出键和值

for(it=mp.begin();it!=mp.end();it++){
	cout<<(*it).first<<' '<<(*it).second<<endl
}
```