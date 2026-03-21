- `sys.exit()` 在程序中途结束这个程序
	
- `sys.stdin.read()` ，他会返回一个包含所有输入内容的**字符串**
	有些题的输入是以EOF结束的，如果逐行读取可能会出错
```python
# 输入
Hello
World
Python
data = sys.stdin.read()
# 返回(data)，也就是说data可以用字符串的方法
"Hello\nWorld\nPython\n"
```





