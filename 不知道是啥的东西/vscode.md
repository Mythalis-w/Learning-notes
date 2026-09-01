在 VS Code 里选编译器通常这样做：

1. 按 `Ctrl + Shift + P`
2. 输入并选择 **`C/C++: Select IntelliSense Configuration`**
3. 选择你的编译器，例如：
    - `gcc.exe`
    - `g++.exe`
    - 或对应的 MinGW 路径

这主要影响代码补全、头文件解析和报错提示。

真正“编译时用哪个”由构建任务决定：

1. 按 `Ctrl + Shift + P`
2. 选择 **`Tasks: Configure Default Build Task`**
3. 选择：
    - **`C/C++: gcc.exe build active file`**：编译 `.c`
    - **`C/C++: g++.exe build active file`**：编译 `.cpp`

如果你同时有 C 和 C++ 文件，建议分别创建两套任务，或用 CMake 管理；不要用 `g++` 直接编译 `.c` 文件。