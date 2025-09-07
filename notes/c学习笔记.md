C 学习笔记

# 可变参数
可变参数（Variadic Arguments）允许函数接受不定数量的参数，是C语言处理变长参数的核心机制（如`printf`、`scanf`）。需包含头文件 `<stdarg.h>`。

`<stdarg.h>`头文件中定义了以下宏：
- `va_list`：用于存储参数列表的类型。
- `va_start`：初始化`va_list`变量。
- `va_arg`：获取下一个参数。
- `va_end`：清理`va_list`变量。

1. **必须至少有一个固定参数**（用于`va_start`定位）
2. **需明确参数类型和数量**（通过固定参数传递信息）
3. **类型安全由程序员负责**（错误类型导致未定义行为）

## 可变参数函数的声明
可变参数函数的声明方式如下：
```c
void function_name(type fixed_arg1, ..., type fixed_argN, ...);
```
- 固定参数是必须的，用于确定参数列表的起始位置。
- 可变参数的数量和类型由函数内部逻辑决定。

## 使用可变参数的步骤
1. **声明`va_list`变量**：用于存储参数列表。
2. **初始化`va_list`变量**：使用`va_start`宏。
3. **访问参数**：使用`va_arg`宏。
4. **清理`va_list`变量**：使用`va_end`宏。

```
栈内存布局：
| 固定参数 | 可变参数1 | 可变参数2 | ... | 可变参数N |
            ↑
            va_start 初始化后 ap 指向这里
```

### 1. `va_list` 类型
- **作用**：声明一个指针变量，用于遍历可变参数列表
- **本质**：通常是 `char*` 或结构体指针（编译器实现相关）
- **特点**：不透明类型（用户不关心内部结构）

```c
va_list ap;  // 声明参数指针变量
```

---

### 2. `va_start` 宏
- **作用**：初始化 `va_list`，使其指向**第一个可变参数**
- **参数**：
  - `ap`：要初始化的 `va_list` 变量
  - `last`：函数声明中**最后一个固定参数**
- **原理**：通过最后一个固定参数的地址计算第一个可变参数位置

```c
void func(int fixed1, char fixed2, ...) {
    va_list ap;
    va_start(ap, fixed2);  // fixed2是最后一个固定参数
    // 现在ap指向第一个可变参数
}
```

---

### 3. `va_arg` 宏
- **作用**：获取当前参数并将指针移动到下一个参数
- **参数**：
  - `ap`：已初始化的 `va_list`
  - `type`：要获取的参数类型
- **返回值**：当前参数的值
- **关键点**：
  - 自动执行指针算术（根据类型大小移动）
  - 必须按参数顺序和声明类型获取

```c
int next_int = va_arg(ap, int);       // 获取int类型
double next_double = va_arg(ap, double); // 获取double类型
```

---

### 4. `va_end` 宏
- **作用**：清理 `va_list` 使用的资源
- **参数**：要清理的 `va_list` 变量
- **重要性**：必须调用，否则可能导致资源泄漏

```c
va_end(ap);  // 清理工作
```

## 参数类型和数量的确定
在可变参数函数中，参数的数量和类型需要通过某种方式确定。通常有以下几种方法：
- **固定参数**：通过固定参数来确定可变参数的数量或类型。
- **哨兵值**：在参数列表的末尾添加一个特殊的值（如`NULL`或`-1`）来标记参数的结束。

## 示例：打印多种类型参数
```c
#include <stdio.h>
#include <stdarg.h>

// 类型标记枚举
typedef enum { INT, DOUBLE, STR } DataType;

void print_args(int count, ...) {
    va_list ap;
    va_start(ap, count);  // 初始化ap，count是最后一个固定参数
    
    for (int i = 0; i < count; i++) {
        // 获取类型标记
        DataType type = va_arg(ap, DataType);
        
        switch (type) {
            case INT: {
                int val = va_arg(ap, int);
                printf("Integer: %d\n", val);
                break;
            }
            case DOUBLE: {
                double val = va_arg(ap, double);
                printf("Double: %.2f\n", val);
                break;
            }
            case STR: {
                char* val = va_arg(ap, char*);
                printf("String: %s\n", val);
                break;
            }
        }
    }
    
    va_end(ap);  // 清理资源
}

int main() {
    // 参数格式：(数量, 类型1, 值1, 类型2, 值2, ...)
    print_args(3, 
              INT, 42, 
              DOUBLE, 3.14159, 
              STR, "Hello World");
}
```

输出：
```
Integer: 42
Double: 3.14
String: Hello World
```

---

## 示例：打印参数地址
```c
void debug_args(int count, ...) {
    va_list ap;
    va_start(ap, count);
    
    printf("Fixed param addr: %p\n", &count);
    printf("First vararg addr: %p\n", ap);
    
    for (int i = 0; i < count; i++) {
        int val = va_arg(ap, int);
        printf("Arg %d: value=%d, next addr=%p\n", 
               i, val, ap);
    }
    
    va_end(ap);
}

int main() {
    debug_args(3, 10, 20, 30);
}
```

典型输出：
```
Fixed param addr: 0x7ffd4357a8cc
First vararg addr: 0x7ffd4357a8d0
Arg 0: value=10, next addr=0x7ffd4357a8d4
Arg 1: value=20, next addr=0x7ffd4357a8d8
Arg 2: value=30, next addr=0x7ffd4357a8dc
```

## 示例: 三个固定参数
```c
#include <stdarg.h>
#include <stdio.h>

// 固定参数：type-操作类型, count-参数个数, base-基础值
void process_data(char type, int count, double base, ...) {
    va_list ap;
    // 关键：base 是最后一个固定参数（...前的参数）
    va_start(ap, base);  // 以 base 为锚点定位可变参数
    
    printf("Operation: %c\nCount: %d\nBase: %.2f\n", type, count, base);
    
    for (int i = 0; i < count; i++) {
        double value = va_arg(ap, double);  // 获取可变参数
        printf("Arg %d: %.2f → ", i+1, value);
        
        switch (type) {
            case '+': printf("Sum: %.2f\n", base + value); break;
            case '*': printf("Product: %.2f\n", base * value); break;
        }
        base = value;  // 更新基础值
    }
    
    va_end(ap);
}

int main() {
    // 固定参数: type='+', count=3, base=10.0
    // 可变参数: 20.0, 30.0, 40.0
    process_data('+', 3, 10.0, 20.0, 30.0, 40.0);
}
```

输出：
```
Operation: +
Count: 3
Base: 10.00
Arg 1: 20.00 → Sum: 30.00
Arg 2: 30.00 → Sum: 50.00
Arg 3: 40.00 → Sum: 70.00
```

## 示例：带元数据的日志函数
```c
#include <stdarg.h>
#include <stdio.h>
#include <time.h>

// 固定参数：level-日志级别, file-文件名, line-行号
void log_message(int level, const char* file, int line, const char* format, ...) {
    // 获取当前时间
    time_t now = time(NULL);
    struct tm *tm = localtime(&now);
    
    // 日志级别标签
    const char* level_str = "INFO";
    if (level == 1) level_str = "WARN";
    if (level == 2) level_str = "ERROR";
    
    // 打印日志头
    printf("[%02d:%02d:%02d][%s] %s:%d - ",
           tm->tm_hour, tm->tm_min, tm->tm_sec,
           level_str, file, line);
    
    // 处理格式化消息
    va_list ap;
    va_start(ap, format);  // format 是最后一个固定参数
    vprintf(format, ap);   // 使用vprintf处理可变参数
    va_end(ap);
    
    printf("\n");
}

int main() {
    // 固定参数：level=0, file=__FILE__, line=__LINE__
    // 可变参数：格式化字符串和参数
    log_message(0, __FILE__, __LINE__, "System started. Version: %d.%d", 2, 3);
    
    int err_code = 404;
    log_message(2, __FILE__, __LINE__, "Connection failed! Error: %d (%s)", 
                err_code, "Not Found");
}
```

输出示例：
```
[14:30:45][INFO] example.c:45 - System started. Version: 2.3
[14:30:45][ERROR] example.c:48 - Connection failed! Error: 404 (Not Found)
```

## 示例：使用哨兵值
假设想实现一个函数，计算所有传入参数的总和，直到遇到一个特殊的值（如`-1`）。

```c
#include <stdio.h>
#include <stdarg.h>

// 函数声明
int sum_with_sentinel(int sentinel, ...);

int main() {
    int result = sum_with_sentinel(-1, 1, 2, 3, 4, 5, -1);
    printf("Sum: %d\n", result);

    result = sum_with_sentinel(-1, 10, 20, 30, -1);
    printf("Sum: %d\n", result);

    return 0;
}

// 函数定义
int sum_with_sentinel(int sentinel, ...) {
    va_list args; // 声明va_list变量
    va_start(args, sentinel); // 初始化va_list变量，sentinel是最后一个固定参数
    int total = 0;
    int value;

    while ((value = va_arg(args, int)) != sentinel) { // 使用哨兵值
        total += value;
    }

    va_end(args); // 清理va_list变量
    return total;
}
```

输出
```
Sum: 15
Sum: 60
```

## 常见错误及避免方法
1. **类型不匹配**：
   ```c
   // 调用：func(3, 1.5, 2.5, 3.5)
   double sum = va_arg(ap, int); // 错误！应为double
   ```
   **解决方法**：使用类型标记或格式字符串

2. **参数数量错误**：
   ```c
   // 声明取3个参数，实际只传2个
   va_arg(ap, int); // 第三次调用时越界
   ```
   **解决方法**：在固定参数中传递准确数量

3. **忘记调用 va_end**：
   ```c
   va_start(ap, count);
   // ...使用ap...
   return; // 忘记va_end!
   ```
   **解决方法**：始终在函数返回前调用 `va_end`

4. **错误使用指针类型**：
   ```c
   char* str = "test";
   // 错误！应传char*，但传了char
   print_args(1, STR, *str); 
   ```
   **正确做法**：
   ```c
   print_args(1, STR, str); // 传递指针而非值
   ```

## 最佳实践总结
| 操作        | 正确用法                           | 错误用法                         |
| ----------- | ---------------------------------- | -------------------------------- |
| 声明        | `va_list ap;`                      | `void* ap;`                      |
| 初始化      | `va_start(ap, last_fixed_param);`  | `va_start(ap, first_param);`     |
| 取int参数   | `int val = va_arg(ap, int);`       | `int val = va_arg(ap, float);`   |
| 取float参数 | `double val = va_arg(ap, double);` | `float val = va_arg(ap, float);` |
| 取字符串    | `char* s = va_arg(ap, char*);`     | `char s = va_arg(ap, char);`     |
| 清理        | `va_end(ap);`                      | 省略清理                         |

## 注意
- **类型安全**：`va_arg`宏需要明确指定参数的类型，否则可能导致未定义行为。
- **参数数量**：必须通过某种方式确定参数的数量，否则可能导致访问非法内存。
- **哨兵值**：如果使用哨兵值，需要确保哨兵值不会与有效参数冲突。

## 应用场景
- **格式化输出**：如`printf`函数。
- **数学计算**：如`sum`函数。
- **日志记录**：记录可变数量的日志信息。

# sprintf
`sprintf` 用于将格式化的数据写入字符串。与 `snprintf` 不同，`sprintf` 不会检查目标缓冲区的大小，因此不会自动截断字符串或添加空字符（`\0`）。这使得 `sprintf` 在使用时需要特别小心，以避免缓冲区溢出的问题。

```c
int sprintf(char *str, const char *format, ...);
```

- **`str`**：目标字符串，`sprintf` 会将格式化后的数据写入这个字符串。
- **`format`**：格式化字符串，描述如何格式化后续参数。
- **`...`**：可变参数列表，根据格式化字符串的要求提供相应的参数。

`sprintf` 的返回值是写入目标字符串的字符数（不包括末尾的空字符 `\0`）。如果发生错误，返回值为负数。

## 示例 1：基本用法

```c
#include <stdio.h>

int main() {
    char buffer[100];
    int num = 42;
    float pi = 3.14159;

    int count = sprintf(buffer, "Number: %d, Pi: %.2f", num, pi);

    printf("Formatted string: %s\n", buffer);
    printf("Number of characters written (excluding '\\0'): %d\n", count);

    return 0;
}
```

**输出**:
```
Formatted string: Number: 42, Pi: 3.14
Number of characters written (excluding '\0'): 21
```

## 示例 2：缓冲区溢出的风险

```c
#include <stdio.h>

int main() {
    char buffer[20];
    int num = 42;
    float pi = 3.14159;

    int count = sprintf(buffer, "This is a very long string that will exceed the buffer size: %d, %.2f", num, pi);

    printf("Formatted string: %s\n", buffer);
    printf("Number of characters written (excluding '\\0'): %d\n", count);

    return 0;
}
```

**输出**:
```
Formatted string: This is a very long string that will exceed the buffer size: 42, 3.14
Number of characters written (excluding '\0'): 60
```

## 注意事项

1. **缓冲区溢出**：
   - `sprintf` 不会检查目标缓冲区的大小，因此如果格式化后的字符串长度超过缓冲区大小，会导致缓冲区溢出。
   - 缓冲区溢出可能导致程序崩溃、数据损坏甚至安全漏洞。

2. **安全替代品**：
   - 为了避免缓冲区溢出，建议使用 `snprintf`，它会检查缓冲区大小并自动截断字符串。
   - `snprintf` 的语法与 `sprintf` 类似，但多了一个参数指定缓冲区大小。

# snprintf
`snprintf` 是 C 标准库中的一个函数，用于将格式化的数据写入字符串缓冲区，并且可以限制写入的长度，从而避免缓冲区溢出。

```c
int snprintf(char *str, size_t size, const char *format, ...);
```

1. **`char *str`**:
   - 指向目标缓冲区的指针，格式化后的字符串将存储在这个缓冲区中。
   - 如果 `str` 为 `NULL`，则 `snprintf` 只计算格式化后的字符串长度，而不实际写入任何数据。

2. **`size_t size`**:
   - 指定目标缓冲区的大小（以字节为单位）。
   - 这是 `snprintf` 的一个重要参数，用于防止缓冲区溢出。如果格式化后的字符串长度超过 `size-1`，`snprintf` 会截断字符串，并在末尾添加空字符 `\0`。

3. **`const char *format`**:
   - 格式字符串，用于指定如何格式化后续的可变参数。
   - 格式字符串中可以包含普通字符和格式说明符（如 `%d`、`%s` 等）。

4. **`...`**:
   - 可变参数列表，根据格式字符串中的格式说明符提供相应的参数。

`snprintf` 返回格式化后的字符串所占用的字节数（不包括结尾的空字符 `\0`）。如果返回值小于 `size`，表示格式化成功且字符串完全写入缓冲区；如果返回值大于或等于 `size`，表示字符串被截断，实际写入的字符串长度为 `size-1`，并且在末尾添加了空字符 `\0`。

## 示例 1：基本用法
```c
#include <stdio.h>

int main() {
    char buffer[50];
    int num = snprintf(buffer, sizeof(buffer), "Hello, %s! You have %d new messages.", "Alice", 5);

    printf("Number of characters written (excluding '\\0'): %d\n", num);
    printf("Formatted string: %s\n", buffer);

    return 0;
}
```

**输出**:
```
Number of characters written (excluding '\0'): 39
Formatted string: Hello, Alice! You have 5 new messages.
```

## 示例 2：缓冲区不足时的行为
```c
#include <stdio.h>

int main() {
    char buffer[7]; // 增加缓冲区大小
    int num = snprintf(buffer, sizeof(buffer), "This is a very long string that will exceed the buffer size.");

    printf("Number of characters written (excluding '\\0'): %d\n", num);
    printf("Formatted string: %s\n", buffer);

    return 0;
}
```

**输出**:
```
Number of characters written (excluding '\0'): 60
Formatted string: This i
```

实际写到缓冲区的为 6 字符，最后添加 \0。

## 示例 3：使用 `snprintf` 计算所需缓冲区大小
```c
#include <stdio.h>

int main() {
    const char *name = "Alice";
    int messages = 5;

    // 计算所需缓冲区大小
    int required_size = snprintf(NULL, 0, "Hello, %s! You have %d new messages.", name, messages);

    // 分配足够大的缓冲区
    char *buffer = (char *)malloc(required_size + 1); // +1 用于空字符

    if (buffer != NULL) {
        snprintf(buffer, required_size + 1, "Hello, %s! You have %d new messages.", name, messages);
        printf("Formatted string: %s\n", buffer);
        free(buffer);
    }

    return 0;
}
```

**输出**:
```
Formatted string: Hello, Alice! You have 5 new messages.
```

## 注意事项

1. **避免缓冲区溢出**:
   - 总是检查 `snprintf` 的返回值，确保目标缓冲区足够大。
   - 如果不确定缓冲区大小，可以先调用 `snprintf` 传入 `NULL` 和 `0` 来计算所需缓冲区大小，然后分配足够大的缓冲区。

2. **空字符的处理**:
   - `snprintf` 总是在格式化后的字符串末尾添加空字符 `\0`，即使字符串被截断。

3. **线程安全性**:
   - `snprintf` 是线程安全的，因为它不依赖于全局变量。

通过合理使用 `snprintf`，可以有效避免缓冲区溢出问题，同时灵活地格式化字符串。

在 C 和 C++ 中，当使用 `snprintf` 函数格式化字符串时，`%s` 格式说明符会读取对应的字符串，直到遇到空字符 `\0` 为止。因此，如果字符串存储在一个字符数组中，`%s` 只会读取字符串的实际长度，而不会读取整个数组的大小。

# vsnprintf
`vsnprintf` 是 C 标准库中的一个函数，用于格式化字符串并将其存储到一个缓冲区中。它是一个变长参数的版本的 `snprintf`，允许你使用 `va_list` 来处理可变参数列表。

`vsnprintf` 的原型定义在 `<cstdio>`（C++）或 `<stdio.h>`（C）中：

```c
int vsnprintf(char *str, size_t size, const char *format, va_list ap);
```

- **`char *str`**：目标缓冲区，用于存储格式化后的字符串。
- **`size_t size`**：目标缓冲区的大小（以字节为单位）。
- **`const char *format`**：格式化字符串，类似于 `printf` 中的格式化字符串。
- **`va_list ap`**：一个变量参数列表，包含要格式化的参数。

**返回值**
- **成功**：返回写入的字符数（不包括末尾的空字符 `\0`）。如果返回值小于 `size`，表示成功写入。
- **失败**：如果返回值大于或等于 `size`，表示缓冲区不足，部分或全部字符串未写入。

**作用**
`vsnprintf` 的主要作用是将格式化后的字符串安全地存储到一个指定大小的缓冲区中，避免缓冲区溢出。它特别适用于处理可变参数列表，例如在实现类似于 `printf` 的函数时。

## 示例

```c
#include <stdio.h>
#include <stdarg.h>
#include <string.h>

void my_printf(const char *format, ...) {
    va_list args;
    char buffer[1000];

    va_start(args, format);
    vsnprintf(buffer, sizeof(buffer), format, args);
    va_end(args);

    printf("%s\n", buffer);
}

int main() {
    my_printf("Hello, my name is %s and I am %d years old.", "John", 30);
    return 0;
}
```

```cpp
#include <iostream>
#include <cstdarg>
#include <cstdio>
#include <cstring>

std::string my_printf(const char *format, ...) {
    va_list args;
    char buffer[1000];

    va_start(args, format);
    vsnprintf(buffer, sizeof(buffer), format, args);
    va_end(args);

    return std::string(buffer);
}

int main() {
    std::string message = my_printf("Hello, my name is %s and I am %d years old.", "John", 30);
    std::cout << message << std::endl;
    return 0;
}
```


如果需要处理更长的字符串，可以使用动态内存分配。例如：
```cpp
#include <iostream>
#include <cstdarg>
#include <cstdio>
#include <cstring>

std::string my_printf(const char *format, ...) {
    va_list args;
    va_start(args, format);

    // 获取所需缓冲区大小
    int size = vsnprintf(nullptr, 0, format, args) + 1; // +1 用于空字符
    va_end(args);

    if (size <= 0) {
        return "";
    }

    std::string buffer(size, '\0');
    va_start(args, format);
    vsnprintf(&buffer[0], size, format, args);
    va_end(args);

    return buffer;
}

int main() {
    std::string message = my_printf("Hello, my name is %s and I am %d years old.", "John", 30);
    std::cout << message << std::endl;
    return 0;
}
```

# `memcpy` - 内存拷贝

**功能**：将一块内存区域（源）的数据复制到另一块内存区域（目标）。它**不处理**源和目标内存区域重叠的情况。如果重叠，其行为是未定义的（可能会出错）。

```c
void *memcpy(void *dest, const void *src, size_t n);
```
-   `dest`: 目标内存地址的指针。
-   `src`: 源内存地址的指针。
-   `n`: 要拷贝的**字节数**。
-   **返回值**：返回目标内存地址的指针 `dest`。

**关键点**：
-   效率极高，是大量数据拷贝的首选。
-   绝对不要用于重叠内存的拷贝。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char src[] = "Hello, World!";
    char dest[20];

    // 将 src 中的前 13 个字节 (包括字符串结束符 '\0') 拷贝到 dest
    memcpy(dest, src, 13);

    printf("Copied string: %s\n", dest); // 输出: Hello, World!

    // 拷贝整型数组
    int arr_src[5] = {1, 2, 3, 4, 5};
    int arr_dest[5];

    memcpy(arr_dest, arr_src, 5 * sizeof(int));

    printf("Copied array: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr_dest[i]); // 输出: 1 2 3 4 5
    }

    return 0;
}
```

# `memmove` - 安全的内存移动（拷贝）

**功能**：与 `memcpy` 类似，也是拷贝内存数据。但它的关键优势在于它能**正确处理**源和目标内存区域重叠的情况。

```c
void *memmove(void *dest, const void *src, size_t n);
```
-   参数和返回值与 `memcpy` 完全相同。

**工作原理**：
它会先判断源和目标内存的位置关系。如果目标地址在源地址之前，或者两者不重叠，它会从前往后拷贝（和 `memcpy` 一样）。如果目标地址在源地址之后（存在重叠风险），它会选择从后往前拷贝，从而避免覆盖尚未被拷贝的源数据。

**何时使用**：
当你不确定两块内存是否重叠时，为了安全起见，总是使用 `memmove`。虽然它可能比 `memcpy` 稍微慢一点点（因为需要做判断），但安全性更重要。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char str[] = "ABCDEFGH"; // 创建一个数组

    // 尝试将 "ABCDEFGH" 从开头向右移动 2 个字节
    // 源地址: str, 目标地址: str + 2
    // 它们明显重叠了 [A B C D E F G H] -> [A B A B C D E F] (如果从前往后拷就会覆盖)
    memmove(str + 2, str, 5); // 拷贝 5 个字节 'A','B','C','D','E'

    // 结果是：原数组的前5个字节被拷贝到了从第3个字节开始的位置
    printf("Result: %s\n", str); // 输出: ABABCDE

    // 对比一下错误的 memcpy 用法（未定义行为）
    // memcpy(str + 2, str, 5); // 可能导致错误输出，如 "ABABABA"

    return 0;
}
```

# `memset` - 内存设置（初始化）

**功能**：将一块内存的每个字节都设置为特定的值。

```c
void *memset(void *ptr, int value, size_t n);
```
-   `ptr`: 要设置的内存起始地址。
-   `value`: 要设置的值（虽然类型是 `int`，但实际使用时会被转换为 `unsigned char`）。
-   `n`: 要设置的字节数。
-   **返回值**：返回原始指针 `ptr`。

**常见用途**：
-   将数组或结构体初始化为 `0`。`memset(arr, 0, sizeof(arr));`
-   将内存区域初始化为一个特定的字符。`memset(buffer, '-', 100);`

```c
#include <stdio.h>
#include <string.h>

int main() {
    char buffer[20];

    // 将 buffer 的所有字节都设置为 'A'
    memset(buffer, 'A', 19);
    buffer[19] = '\0'; // 手动添加字符串结束符

    printf("Buffer: %s\n", buffer); // 输出: AAAAAAAAAAAAAAAAAAA

    // 将整型数组清零
    int arr[5];
    memset(arr, 0, 5 * sizeof(int)); // 每个字节都设为0，整个int也就是0

    printf("Array after memset: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]); // 输出: 0 0 0 0 0
    }

    return 0;
}
```
**注意**：`memset` 按字节操作。如果想将 `int` 数组初始化为 `1`，使用 `memset(arr, 1, sizeof(arr))` 并不会得到每个元素为 `1` 的数组，而是得到每个元素值为 `0x01010101`（取决于系统字节顺序）的数组。

# `memcmp` - 内存比较

**功能**：比较两块内存区域的前 `n` 个字节的内容。

**原型**：
```c
int memcmp(const void *ptr1, const void *ptr2, size_t n);
```
-   `ptr1`: 第一块内存的指针。
-   `ptr2`: 第二块内存的指针。
-   `n`: 要比较的字节数。
-   **返回值**：
    -   `< 0`: 第一个不匹配的字节在 `ptr1` 中的值**低于**在 `ptr2` 中的值。
    -   `0`: 两块内存的内容完全相同。
    -   `> 0`: 第一个不匹配的字节在 `ptr1` 中的值**高于**在 `ptr2` 中的值。

**与 `strcmp` 的区别**：`memcmp` 比较精确的 `n` 个字节，不会因为遇到 `'\0'` 而停止。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char str1[] = "ABCDEF";
    char str2[] = "ABCXYZ";

    // 比较前 3 个字节 ("ABC" vs "ABC")
    int result1 = memcmp(str1, str2, 3);
    printf("Compare first 3 bytes: %d\n", result1); // 输出: 0 (相等)

    // 比较前 4 个字节 ('D' vs 'X', D < X)
    int result2 = memcmp(str1, str2, 4);
    printf("Compare first 4 bytes: %d\n", result2); // 输出: 一个负数

    return 0;
}
```

# `memchr` - 内存字符搜索

**功能**：在指向的内存块的前 `n` 个字节中搜索第一次出现的特定字符（转换为 `unsigned char`）。

```c
void *memchr(const void *ptr, int value, size_t n);
```
-   `ptr`: 要搜索的内存起始地址。
-   `value`: 要搜索的字符值（转换为 `unsigned char`）。
-   `n`: 搜索的字节范围。
-   **返回值**：如果找到，返回指向该字符的指针；否则返回 `NULL`。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char data[] = "Hello, this is a test string.";
    char search_char = 't';

    // 在 data 中前 20 个字节里搜索字符 't'
    char *result = memchr(data, search_char, 20);

    if (result != NULL) {
        printf("Found '%c' at position: %ld\n", search_char, result - data);
        printf("The rest of string: %s\n", result);
    } else {
        printf("Character '%c' not found.\n", search_char);
    }
    // 输出:
    // Found 't' at position: 8
    // The rest of string: this is a test string.

    return 0;
}
```

# `strcpy` - 字符串拷贝

**功能**：将源字符串（包括结束符 `'\0'`）复制到目标字符数组中。

```c
char *strcpy(char *dest, const char *src);
```

**工作方式**：
1.  从 `src` 地址开始逐个字符拷贝到 `dest`。
2.  一旦遇到源字符串中的 `'\0'` 结束符，**会将其拷贝过去**，然后停止。

**关键特性与风险**：
*   **不检查目标数组大小**：这是它最危险的地方。如果源字符串长度大于目标数组容量，会导致**缓冲区溢出**，这是非常严重的安全漏洞，可能会覆盖相邻的内存数据，导致程序崩溃或被利用执行恶意代码。
*   **保证目标字符串以 `'\0'` 结尾**：因为它会拷贝源字符串的结束符。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char src[20] = "Hello, World!";
    char dest[20];

    strcpy(dest, src); // 安全，因为 src 长度小于 dest
    printf("Dest: %s\n", dest); // 输出: Hello, World!

    // 危险示例！
    char small_buffer[5];
    // strcpy(small_buffer, "This is a long string"); // 缓冲区溢出！未定义行为，极其危险！

    return 0;
}
```
**结论**：在现代编程中，**应避免使用 `strcpy`**，除非你能 100% 确定源字符串绝不会长于目标缓冲区。

# `strncpy` - 带长度限制的字符串拷贝

**功能**：尝试从源字符串拷贝最多 `n` 个字符到目标数组。

**原型**：
```c
char *strncpy(char *dest, const char *src, size_t n);
```

**工作方式与 `'\0'` 处理（这是最关键的细节）**：
1.  **拷贝字符**：从 `src` 开始拷贝，直到以下两种情况之一发生：
    *   拷贝了 `n` 个字符。
    *   遇到了源字符串的 `'\0'` 结束符。
2.  **`'\0'` 处理规则**：
    *   **规则一**：如果在拷贝完 `n` 个字符之前遇到了 `src` 的 `'\0'` 结束符，`strncpy` 会**用 `'\0'` 填充**目标数组中剩余的空间，直到写满 `n` 个字符。
    *   **规则二**：如果拷贝了 `n` 个字符都**没有**遇到 `src` 的 `'\0'`（即源字符串长度 >= `n`），那么 `strncpy` **不会在目标数组的末尾添加 `'\0'` 结束符**！

**关键特性与风险**：
*   **设计初衷**：它的原始设计并非为了创建安全的字符串，而是为了处理 UNIX 文件系统中的一个古老数据结构，该结构需要完全填满固定长度的字段。
*   **主要风险**：由于规则二，目标数组可能**不是一个有效的、以 `'\0'` 结尾的 C 字符串**。如果你后续像使用普通字符串一样使用它（例如用 `printf("%s", dest)`），会导致未定义行为（因为会一直读取内存直到碰巧遇到一个 `'\0'`）。

```c
#include <stdio.h>
#include <string.h>

int main() {
    char src[] = "Hello"; // 长度 6 (5个字母 + 1个\0)
    char dest[10];

    // 案例 1: n > src 长度
    // 会拷贝 'H','e','l','l','o','\0'，然后用 '\0' 填充剩下的 4 个字节 (规则一)
    strncpy(dest, src, 10);
    printf("Case 1: %s (Null-terminated)\n", dest); // 安全输出: Hello

    // 案例 2: n == src 长度 (注意：这个长度不包含最后的 \0)
    // 会拷贝 'H','e','l','l','o'。不会拷贝 src 的 '\0'！(规则二)
    char dest2[5];
    strncpy(dest2, src, 5); // 只拷贝了5个字符，没有\0
    // dest2 现在不是合法的字符串！
    // printf("Case 2: %s\n", dest2); // 危险！未定义行为，会读取越界

    // 正确用法：手动添加终止符
    dest2[4] = '\0'; // 确保在最后一个位置添加 \0
    printf("Case 2 (fixed): %s\n", dest2); // 安全输出: Hell

    return 0;
}
```
**安全使用 `strncpy` 的范式**：
```c
char dest[BUF_SIZE] = {};
strncpy(dest, src, BUF_SIZE - 1); // 最多拷贝 BUF_SIZE-1 个字符
dest[BUF_SIZE - 1] = '\0';        // 手动确保最后一个位置是终止符
```

# `strcpy`/`strncpy` 与 `memcpy` 的核心区别总结

| 特性                         | `strcpy`                      | `strncpy`                           | `memcpy`                             |
| :--------------------------- | :---------------------------- | :---------------------------------- | :----------------------------------- |
| **操作对象**                 | 字符串                        | 字符串                              | **内存块**（任何数据）               |
| **终止符感知**               | **是** (遇到 `\0` 停止并拷贝) | **是** (行为复杂)                   | **否** (完全忽略内容)                |
| **长度参数**                 | 无                            | `n`: **最大字符数**                 | `n`: **精确字节数**                  |
| **目标是否会以 `\0` 结尾？** | **总是**                      | **不一定** (需手动保证)             | **不关心** (按字节拷贝)              |
| **主要风险**                 | 缓冲区溢出                    | 目标可能无终止符                    | 内存重叠问题 (用 `memmove` 解决)     |
| **性能**                     | 需要逐字节检查 `\0`           | 需要逐字节检查 `\0` 并可能填充 `\0` | **极高** (通常用硬件指令优化)        |
| **用途**                     | **已过时，不推荐使用**        | 拷贝字符串并希望限制长度            | 拷贝任何类型的数据（数组、结构体等） |

## 现代C编程的建议

1.  **永远不要使用 `strcpy`**。它的危险性远大于便利性。
2.  **谨慎使用 `strncpy`**。如果使用，必须牢记它不会自动添加终止符的特性，并遵循“拷贝 `n-1` 个字符后手动添加 `\0`”的模式。
3.  **优先使用更安全的替代函数**（如果编译器支持，如 GNU C 或 C11 的 Annex K）：
    *   `strlcpy` (非标准，但很常见)：行为更直观，总是保证目标字符串以 `\0` 结尾。
    *   `snprintf` (标准函数)：非常安全的选择。
        ```c
        char dest[BUF_SIZE];
        snprintf(dest, sizeof(dest), "%s", src); // 安全，自动处理终止符
        ```
4.  **明确的操作对象**：
    *   如果要操作的是**字符串**，目的是产生一个以 `\0` 结尾的字符串，使用字符串函数（并注意安全）。
    *   如果要操作的是**任意数据**（一个结构体、一个整型数组、或者字符串的一部分），使用 `memcpy` 或 `memmove`。例如，从一个字符串中拷贝中间的一部分数据，`memcpy` 是完美选择。
        ```c
        char source[] = "Hello World";
        char excerpt[6];
        memcpy(excerpt, &source[6], 5); // 拷贝 "World" 这5个字符
        excerpt[5] = '\0';              // 手动添加终止符，使其成为字符串
        `````

# 条件编译        

1. **条件编译优先级**：  
   将特定平台的条件放在前面，通用平台放在后面。
2. **互斥性检查**：  
   使用`#if`/`#elif`/`#else`链确保只有一个分支生效。
3. **宏定义管理**：  
   在项目配置文件中明确各平台宏的定义，避免冲突。
4. **调试技巧**：  
   使用`#error`指令验证条件编译路径。

# malloc

```c
void *malloc(size_t size);
```

- `size`：需要分配的内存块的大小，以字节为单位。类型是 `size_t`，通常定义在 `<stddef.h>` 中，是一个无符号整数类型，用于表示大小或数量。

返回值：
- 返回一个指向分配的内存块的指针，类型是 `void*`，即空类型指针。
- 如果内存分配成功，返回的指针指向分配的内存块的起始位置。
- 如果内存分配失败（例如请求的内存大小为0，或者内存不足等），返回 `NULL`。

可以通过检查 `malloc` 的返回值来判断内存分配是否成功。如果返回值不是 `NULL`，说明分配成功；如果返回值是 `NULL`，说明分配失败。

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *ptr = (int *)malloc(sizeof(int) * 10); // 分配足够存放10个整数的内存

    if (ptr == NULL) {
        // 分配失败
        printf("Memory allocation failed\n");
        return 1;
    } else {
        // 分配成功
        printf("Memory allocation succeeded\n");
        // 使用分配的内存...
        
        // 释放内存
        free(ptr);
    }

    return 0;
}
```

## 注意事项
- **类型转换**：在 C++ 中，`malloc` 返回的 `void*` 需要显式转换为目标指针类型（如上面的 `(int *)`）。在 C 中，类型转换不是必须的，但为了兼容性，有时也会加上。
- **内存初始化**：`malloc` 分配的内存内容是未初始化的，即其中的值是随机的。如果需要初始化为零，可以使用 `calloc` 函数。
- **释放内存**：使用 `malloc` 分配的内存必须用 `free` 函数释放，避免内存泄漏。
- **分配失败处理**：在嵌入式系统或内存紧张的环境中，`malloc` 可能会频繁失败，因此总是需要检查返回值。

# calloc
在C语言中，`calloc`函数用于动态内存分配，它会将分配的内存初始化为0。

```c
void* calloc(size_t num, size_t size);
```

- **num**：要分配的元素个数，类型是`size_t`（无符号整型）。
- **size**：每个元素的大小（以字节为单位），类型也是`size_t`。

- 返回一个指向已分配内存块的指针，类型是`void*`，该内存块已被初始化为0。
- 如果内存分配失败（例如内存不足），则返回`NULL`。

`calloc`函数为`num`个大小为`size`的元素分配内存空间，并将每个元素初始化为0。这与`malloc`函数不同，后者不会初始化内存。

## 示例
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *arr = (int*)calloc(5, sizeof(int)); // 分配5个整数的内存，并初始化为0

    if (arr == NULL) {
        // 分配失败
        printf("Memory allocation failed\n");
        return 1;
    } else {
        // 分配成功，打印数组内容（全为0）
        printf("Array elements after calloc:\n");
        for (int i = 0; i < 5; i++) {
            printf("%d ", arr[i]);
        }
        printf("\n");
    }

    // 使用分配的内存...
    // 修改数组内容
    for (int i = 0; i < 5; i++) {
        arr[i] = i + 1;
    }

    printf("Modified array elements:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");

    // 释放内存
    free(arr);
    arr = NULL;

    return 0;
}
```

## 注意事项
- **检查返回值**：在使用`calloc`时，必须检查返回值是否为`NULL`，以确定内存分配是否成功。
- **内存释放**：使用`calloc`分配的内存必须用`free`函数释放，以避免内存泄漏。
- **初始化为0**：`calloc`分配的内存会被自动初始化为0，这在处理数组、结构体等数据时非常有用。

# goto
`goto` 是 C 语言中的一种跳转语句，它允许程序无条件地跳转到指定的标签（label）。标签的定义方式是在代码中指定一个标识符，后面跟着一个冒号（`:`）。使用 `goto` 时，程序会跳转到该标签的位置，并从那里继续执行。

```c
goto label;
...
label: // 标签
    // 跳转后继续执行的代码
```

```c
#include <stdio.h>

int main() {
    int x = 10;

    if (x > 5) {
        goto end; // 跳转到标签 "end"
    }

    printf("This line will not be executed.\n");

end: // 标签
    printf("Reached the end.\n");

    return 0;
}
```
**输出：**
```
Reached the end.
```

- 当 `x > 5` 时，`goto end;` 会跳转到标签 `end`。
- 跳转后，`printf("This line will not be executed.\n");` 不会被执行，程序直接从标签 `end` 开始执行。

## goto 警告
在 C 语言中，`goto` 语句用于无条件跳转到代码中的指定标签位置。其核心规则是：**`goto` 只能在当前函数内跳转，且不能跳过变量的初始化**（C 标准要求局部变量的初始化必须在作用域内完成）。

## 示例

如果 `goto` 跳转时跳过了变量的初始化，编译器会报错。例如：
```c
void demo() {
    goto jump; // 错误：跳过变量初始化
    int x = 10;

jump:
    printf("%d\n", x); // x 未初始化
}
```
编译器会提示：
```
error: jump bypasses variable initialization
```

# strlen
`strlen` 是 C 标准库中一个非常常用的函数，用于计算字符串的长度。它定义在 `<string.h>` 头文件中。

```c
size_t strlen(const char *s);
```

- **`s`**：指向以空字符（`\0`）结尾的字符串的指针。
- **返回值**：返回字符串的长度，不包括末尾的空字符（`\0`）。

`strlen` 用于计算**以空字符结尾**的字符串的长度。它会从给定的字符串开始，逐个字符地计数，直到遇到空字符（`\0`）为止。

## 示例 1：基本用法

```c
#include <stdio.h>
#include <string.h>

int main() {
    const char *str = "Hello, World!";
    size_t length = strlen(str);

    printf("The length of the string \"%s\" is %zu.\n", str, length);

    return 0;
}
```

**输出**:
```
The length of the string "Hello, World!" is 13.
```

## 示例 2：处理空字符串

```c
#include <stdio.h>
#include <string.h>

int main() {
    const char *str = "";
    size_t length = strlen(str);

    printf("The length of the string \"%s\" is %zu.\n", str, length);

    return 0;
}
```

**输出**:
```
The length of the string "" is 0.
```

## 注意事项

1. **空字符（`\0`）**：
   - `strlen` 计算的是从字符串开始到第一个空字符（`\0`）之前的字符数。因此，它不包括末尾的空字符。

2. **字符串必须以空字符结尾**：
   - 如果传入的字符串没有以空字符结尾，`strlen` 会继续读取内存，直到遇到空字符为止。这可能导致未定义行为，甚至程序崩溃。

3. **返回值类型**：
   - `strlen` 的返回值类型是 `size_t`，这是一个无符号整数类型，通常用于表示大小。在使用时，确保将其存储在适当类型的变量中。

4. **性能考虑**：
   - `strlen` 需要逐个字符地遍历字符串，因此其时间复杂度为 O(n)，其中 n 是字符串的长度。对于非常长的字符串，这可能会影响性能。

## 安全性

使用 `strlen` 时，确保传入的字符串是有效的，并且以空字符结尾。如果字符串可能来自不可信的源，建议在使用前进行验证，以避免潜在的安全问题。