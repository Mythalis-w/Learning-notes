`enum` 是 C++ 中的**枚举类型**，用来定义一组命名的常量。简单来说，就是给数字起个有意义的名字，让代码更易读、更安全。

这个类型本质就是转换，让代码更加容易读懂 : 用户输入时用**数字**，程序内部用**枚举**，显示时用**文字**。
```C++
enum Weekday {
    MONDAY,     // 0
    TUESDAY,    // 1
    WEDNESDAY,  // 2
    THURSDAY,   // 3
    FRIDAY,     // 4
    SATURDAY,   // 5
    SUNDAY      // 6
};

Weekday today = MONDAY;  // 清清楚楚是周一
if (today == MONDAY) {   // 代码一看就懂
    cout << "今天是周一" << endl;
}
```

```C++
// 1. 用户输入数字
int sugarChoice;
cin >> sugarChoice;  // 用户输入 2

// 2. 将数字转换成对应的 enum 类型
SugarLevel sugar = static_cast<SugarLevel>(sugarChoice);
// sugar 现在是 SugarLevel::HALF (半糖)

// 3. 存储到订单中
OrderItem item(product, quantity, sugar, temperature);

// 4. 显示时转换回文字
cout << EnumToString::sugarLevelToString(sugar);  // 输出"半糖"
```
数字是可以进行转化的，但是直接给枚举类型赋值为数字是错误的，比如第一个例子today赋值为0会报错，想要转化可以用函数，如下面的例子

```C++
#include <iostream>
using namespace std;
enum Weekday {
    MONDAY,     // 0
    TUESDAY,    // 1
    WEDNESDAY,  // 2
    THURSDAY,   // 3
    FRIDAY,     // 4
    SATURDAY,   // 5
    SUNDAY      // 6
};
// ===== 函数1：Weekday → string（显示用） =====
string weekdayToString(Weekday day) {
    switch(day) {
        case MONDAY:    return "星期一";
        case TUESDAY:   return "星期二";
        case WEDNESDAY: return "星期三";
        case THURSDAY:  return "星期四";
        case FRIDAY:    return "星期五";
        case SATURDAY:  return "星期六";
        case SUNDAY:    return "星期日";
        default:        return "未知";
    }
}
// ===== 函数2：int → Weekday（输入用） =====
Weekday intToWeekday(int num) {
    switch(num) {
        case 0: return MONDAY;
        case 1: return TUESDAY;
        case 2: return WEDNESDAY;
        case 3: return THURSDAY;
        case 4: return FRIDAY;
        case 5: return SATURDAY;
        case 6: return SUNDAY;
        default: return MONDAY;  // 输入错误时默认返回周一
    }
}
```


`enum`有两种写法，一般是分为`enum name` 和 `enum class name` 
```C++
enum Weekday {
    MONDAY,     // 0
    TUESDAY,    // 1
    // ...
};

Weekday today = MONDAY;  // 直接使用，不用加 Weekday::
int num = MONDAY;        // 甚至可以隐式转换成整数（危险！）
```

```C++
enum class Weekday {
    MONDAY,     // 0
    TUESDAY,    // 1
    // ...
};

Weekday today = Weekday::MONDAY;  // 必须加作用域
// int num = Weekday::MONDAY;     // 错误！不能隐式转换
int num = static_cast<int>(Weekday::MONDAY);  // 必须显式转换
```

写class就是规定作用域，如果不写class的话enum重复的话会报错的