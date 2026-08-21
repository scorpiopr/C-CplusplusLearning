std::format 是 C++20 引入的现代化文本格式化库（并在 C++23 中进一步增强），旨在提供像 Python 的 print(f"...") 一样安全、好用的格式化体验，彻底终结了 printf 的类型不安全和 std::cout 的语法冗长。
------------------------------
## 🚀 核心语法与基础用法
std::format 位于 <format> 头文件中。它使用大括号 {} 作为占位符，默认支持自动类型推导。

```cpp
#include <iostream>
#include <format> // 必须引入该头文件
#include <string>
int main() {
    std::string name = "Alice";
    int age = 25;

    // 1. 基础自动推导
    std::string s1 = std::format("Hello, {}! You are {} years old.", name, age);
    
    // 2. 指定位置索引（从 0 开始，可复用参数）
    std::string s2 = std::format("{0} 的年龄是 {1}，请记住 {0} 的名字。", name, age);

    std::cout << s1 << "\n" << s2 << "\n";
    return 0;
}
```
------------------------------
## 🎨 进阶：格式化说明符（Width, Precision, Alignment）
大括号内部可以通过 {:说明符} 的形式控制输出样式。其核心语法格式为：{[索引]:[填充字符][对齐方式][宽度][.精度][类型]}
## 1. 文本对齐与填充

* < 左对齐，> 右对齐，^ 居中对齐。
* 对齐符号左侧可以紧跟一个自定义填充字符（默认是空格）。

```cpp
// 宽度为 10，默认左对齐，空格填充
std::format("{:10}", "Hi");      // "Hi        "
// 宽度为 10，右对齐，用 '*' 填充
std::format("{:*>&10}", "Hi");   // "********Hi"
// 宽度为 10，居中对齐，用 '-' 填充
std::format("{:-^10}", "Hi");    // "----Hi----"
```

## 2. 数字、进制与浮点数精度

* .精度：控制浮点数的小数位数或字符串的最大截取长度。
* b/B（二进制）、o（八进制）、x/X（十六进制）。
* #：显示进制前缀（如 0x）。

// 浮点数保留 2 位小数
std::format("{:.2f}", 3.14159);  // "3.14"
// 进制转换（# 自动添加 0x 前缀，X 表示大写）
std::format("{:#X}", 255);       // "0xFF"
std::format("{:b}", 5);          // "101" (二进制)

------------------------------
## ⚡ C++23 重磅升级：直接支持打印容器
在 C++20 中，std::format 无法直接打印容器（如 std::vector），需要自己写循环。
C++23 彻底解决了这一痛点，让标准容器、原生数组甚至 std::pair 和 std::tuple 均可直接放入 std::format 中打印。

```cpp
#include <vector>
#include <map>
#include <tuple>
#include <format>
#include <iostream>
int main() {
    // C++23 支持直接格式化容器
    std::vector<int> v = {1, 2, 3, 4};
    std::cout << std::format("Vector: {}\n", v); 
    // 输出: Vector: [1, 2, 3, 4]

    std::map<std::string, int> m = {{"Alice", 20}, {"Bob", 22}};
    std::cout << std::format("Map: {}\n", m);
    // 输出: Map: {"Alice": 20, "Bob": 22}

    std::pair<int, std::string> p = {1, "One"};
    std::cout << std::format("Pair: {}\n", p);
    // 输出: Pair: (1, "One")
}
```

------------------------------
## 🎯 配合 std::print（C++23 终极简化）
C++20 的 std::format 只返回 std::string，要打印到屏幕依然需要写 std::cout << std::format(...)。
C++23 引入了 std::print 和 std::println，它们内置了 std::format 的所有功能，且直接输出到标准输出流，性能比 cout 更高。

```cpp
#include <print> // C++23 新增头文件
int main() {
    // C++23 写法，自动换行，再也不需要 std::cout 和 std::endl
    std::println("Hello {}! Today is {:.2f} degrees.", "World", 26.55);
}
```

------------------------------
## 📊 格式化方案技术对比

| 特性 | printf (C) | std::cout (C++98) | std::format (C++20/23) |
|---|---|---|---|
| 类型安全 | ❌ 极差（类型写错直接崩溃） | 安全 | 绝对安全（编译期/运行期检查） |
| 代码可读性 | 好（格式与变量分离） | ❌ 极差（大量的 << 嵌套） | 极好（类似 Python/Rust 的优雅语法） |
| 性能表现 | 快 | 较慢（受本地化 I/O 影响） | 极快（几乎与 printf 齐平） |
| 支持自定义类型 | ❌ 不支持 | 通过重载 << 支持 | 通过特化 std::formatter 支持 |


