# Serial Communication Library (Linux)

一个用于 Linux 平台的 C++ RS-232 串口通信库。提供了简洁的现代 C++ 接口，用于串口的打开、关闭、读写等操作。

## 特性

- ✅ 简单易用的 C++ 接口
- ✅ 支持共享库和静态库
- ✅ 超时控制和流控制
- ✅ 无额外依赖
- ✅ CMake 集成支持

## 系统要求

- **操作系统**: Linux (Ubuntu/Debian 等)
- **编译器**: GCC 或 Clang，支持 C++11
- **构建工具**: CMake 3.5+
- **依赖库**: pthread, rt (系统自带)

## 构建与安装

### 1. 构建共享库（推荐）

```bash
mkdir build && cd build
cmake -DBUILD_SHARED_LIBS=ON ..
make
sudo make install
```

安装位置：
- 库文件: `/usr/local/lib/libserial.so`
- 头文件: `/usr/local/include/serial/`
- CMake 配置: `/usr/local/lib/cmake/serial/`

### 2. 构建静态库

```bash
mkdir build && cd build
cmake -DBUILD_SHARED_LIBS=OFF ..
make
sudo make install
```

### 3. 构建选项

```bash
# 禁用示例程序
cmake -DBUILD_EXAMPLES=OFF ..

# 指定安装路径
cmake -DCMAKE_INSTALL_PREFIX=/usr ..

# Debug 模式
cmake -DCMAKE_BUILD_TYPE=Debug ..
```

### 4. 卸载

```bash
cd build
sudo make uninstall
```

## 使用方法

### CMake 项目集成

在你的 `CMakeLists.txt` 中：

```cmake
cmake_minimum_required(VERSION 3.5)
project(my_project)

# 查找 serial 库
find_package(serial REQUIRED)

# 添加可执行文件
add_executable(my_app main.cpp)

# 链接 serial 库
target_link_libraries(my_app serial::serial)
```

### 基本示例代码

```cpp
#include <serial/serial.h>
#include <iostream>

int main() {
    // 创建串口对象
    serial::Serial my_serial("/dev/ttyUSB0", 115200, 
                             serial::Timeout::simpleTimeout(1000));
    
    // 检查串口是否打开
    if(my_serial.isOpen()) {
        std::cout << "串口已打开" << std::endl;
    }
    
    // 写入数据
    std::string data = "Hello Serial!";
    my_serial.write(data);
    
    // 读取数据
    std::string result = my_serial.read(100);
    std::cout << "接收: " << result << std::endl;
    
    // 关闭串口
    my_serial.close();
    
    return 0;
}
```

### 编译示例

使用 CMake：
```bash
cmake ..
make
```

手动编译：
```bash
g++ -std=c++11 main.cpp -lserial -o my_app
```

### 常用 API

```cpp
// 构造函数
Serial(const std::string &port, 
       uint32_t baudrate,
       Timeout timeout = Timeout());

// 基本操作
void open();                          // 打开串口
void close();                         // 关闭串口
bool isOpen();                        // 检查是否打开
size_t available();                   // 可读字节数

// 读写操作
size_t write(const std::string &data);     // 写入数据
size_t write(const uint8_t *data, size_t size);
std::string read(size_t size);             // 读取指定字节
std::string readline();                    // 读取一行
void flush();                              // 刷新缓冲区

// 配置
void setBaudrate(uint32_t baudrate);       // 设置波特率
void setTimeout(Timeout &timeout);         // 设置超时
void setPort(const std::string &port);     // 设置端口
```

### 波特率支持

常用波特率：9600, 19200, 38400, 57600, 115200, 230400, 460800, 921600

## 示例程序

查看 `examples/serial_example.cc` 获取完整示例：

```bash
cd build
./serial_example
```

## 常见问题

### 1. 权限问题

如果遇到权限错误，将用户添加到 `dialout` 组：

```bash
sudo usermod -a -G dialout $USER
# 注销后重新登录生效
```

### 2. 找不到串口设备

查看可用串口：
```bash
ls /dev/tty*
# 或
dmesg | grep tty
```

### 3. CMake 找不到 serial 包

确保已正确安装，或手动指定路径：
```bash
cmake -Dserial_DIR=/usr/local/lib/cmake/serial ..
```

### 4. 动态库链接问题

更新动态库缓存：
```bash
sudo ldconfig
```

## 许可证

[MIT License](LICENSE)

## 原作者

- William Woodall <wjwwood@gmail.com>
- John Harrison <ash.gti@gmail.com>

---

**注**: 此版本已针对 Linux 平台精简优化，移除了 Windows 和 macOS 支持。
