# VrdsClient Qt 桌面客户端技术实现说明

> 说明：文档路径均以 `D:\VrdsClient_v1.2_new\VrdsClient_v1.2_new` 为仓库根目录。本文描述的是当前目录中源码可验证的实现范围，注释保留、未接入主流程的扩展能力已单独标注。

## 岗位 JD 对照标注

以下标注为在原有文档基础上新增的一层对照，不删除、不改写原文内容。标注口径：

- `【已实现】`：源码中有直接对应实现，可以在本文件相应章节找到代码证据。
- `【部分实现/相关】`：源码中有相关基础或间接能力，但不等同于 JD 中该项的完整标准实现。
- `【代码未体现】`：当前项目源码中未找到对应技术或用法，需要依靠其他项目经历补充。

### 任职要求逐条对照

| JD 条目                                     | 标注              | 代码证据                                                     |
| ------------------------------------------- | ----------------- | ------------------------------------------------------------ |
| 本科、计算机/软件工程、2年以上 C++ 开发经验 | 非代码项          | 需在简历中由个人经历体现                                     |
| 熟练掌握 C++（C++11/14/17）                 | 【部分实现/相关】 | 工程显式配置 `CONFIG += c++11`，见 `VrdsClient/VrdsClient.pro:5`；代码中未使用 C++14/17 专有特性 |
| 常用设计模式                                | 【部分实现/相关】 | 大量使用 Qt 信号槽与事件驱动模型，可归入观察者/事件驱动范式；源码未显式命名工厂、策略等设计模式 |
| STL                                         | 【已实现】        | `std::string`：`VrdsClient/myfunc.h:27`；`std::unordered_set`：`VrdsClient/Client/vrdsclient.cpp:17`；`std::unordered_map`：`VrdsClient/Client/vrdsclient.cpp:20`、`343` |
| 内存管理                                    | 【已实现】        | `new` 与 `deleteLater()` 配合释放网络回复、定时器、TCP 对象；FFmpeg 对象通过 `av_frame_free`、`avcodec_close`、`avformat_close_input` 释放，见 `VrdsClient/Client/mediaplayerthread.h:156` |
| 多线程编程                                  | 【已实现】        | `QThread`、`std::mutex`、`std::condition_variable`、`std::unique_lock`，见 `VrdsClient/Client/mediaplayerthread.h:19`、`58`、`170` |
| Qt Widgets / QML                            | 【部分实现/相关】 | 本项目为 Qt Widgets 实际落地；源码中无 QML/Qt Quick 业务代码 |
| 信号槽机制                                  | 【已实现】        | `connect`、`SIGNAL/SLOT`、Lambda 槽遍布登录与主窗口，例如 `VrdsClient/Login/login.cpp:75`、`VrdsClient/Client/vrdsclient.cpp:278` |
| 事件机制                                    | 【已实现】        | 重写 `keyPressEvent`、`resizeEvent`、`closeEvent`，见 `VrdsClient/Client/vrdsclient.cpp:59`、`87`、`VrdsClient/Client/vrdsclient.h:52` |
| Model/View 框架                             | 【部分实现/相关】 | 使用 `QTableWidget` 完成数据展示与编辑，属于 Qt item view 系列控件；未使用自定义 `QAbstractItemModel`/`QStandardItemModel` |
| Qt 网络模块                                 | 【已实现】        | `QNetworkAccessManager`、`QTcpSocket`、`QNetworkRequest/Reply`、`QWebSocket` 头文件，见 `VrdsClient/Login/login.cpp:64`、`VrdsClient/Client/vrdsclient.cpp:482`、`VrdsClient/Client/vrdsclient.h:14` |
| Qt 国际化                                   | 【代码未体现】    | 未发现 `QTranslator`、`.ts/.qm`、`tr()` 国际化工程配置       |
| Windows/Linux 平台开发                      | 【部分实现/相关】 | 当前业务代码以 Windows/MSVC 为主，`.pro` 中有 `win32:` 与 `msvc` 配置；Linux 仅存在 qmake install 占位，见 `VrdsClient/VrdsClient.pro:7`、`42` |
| 调试工具 GDB/Visual Studio/Valgrind         | 【部分实现/相关】 | 工程使用 MSVC/Qt Creator 构建，`VrdsClient.pro.user` 中存在 Valgrind 分析配置；源码主要使用 `qDebug` 输出日志，不包含可验证的 GDB/Visual Studio 调试流程 |
| 编码规范与文档习惯                          | 【部分实现/相关】 | 源码含较完整的中文注释、接口说明和分模块文件；代码规范与文档习惯主要由个人经历说明 |
| 独立分析与解决问题、沟通协作                | 非代码项          | 需在简历中由工作经历体现                                     |

### 加分项逐条对照

| JD 加分项                             | 标注              | 代码证据                                                     |
| ------------------------------------- | ----------------- | ------------------------------------------------------------ |
| 跨平台 Windows/macOS/Linux 商业化交付 | 【部分实现/相关】 | 当前是 Windows 桌面客户端工程，存在 `unix` install 占位，但未发现 macOS/Linux 完整适配代码 |
| QML/Qt Quick                          | 【代码未体现】    | 首源代码中未发现 QML/Qt Quick 业务实现                       |
| QThread                               | 【已实现】        | `MediaPlayerThread : public QThread`，见 `VrdsClient/Client/mediaplayerthread.h:21` |
| QOpenGL                               | 【代码未体现】    | 首源代码中未发现 `QOpenGL`/`QOpenGLWidget` 使用              |
| FFmpeg                                | 【已实现】        | 视频线程完整调用 `avformat`、`avcodec`、`avpicture`、`swscale`，见 `VrdsClient/Client/mediaplayerthread.h:67` 起 |
| GStreamer                             | 【代码未体现】    | 工程和源码中未发现 GStreamer 接入                            |
| 嵌入式 Linux、Qt Embedded、Qt for MCU | 【代码未体现】    | 当前工程为 Windows 桌面 Qt Widgets 客户端，不包含嵌入式部署代码 |
| 工业软件、上位机、SCADA、CAD          | 【部分实现/相关】 | 本项目属于桌面监控与远程控制上位机方向，包含实时状态轮询、多路视频监控、远程指令下发等上位机常见能力；不是 SCADA/CAD 软件 |

### 已实现技术在本文件中的章节定位

- 登录、Qt Network、JSON 配置：见 4.2 节。
- 二进制协议、C++ 基础、内存/线程基础：见 4.3 节。
- Qt Widgets 多页面 UI、事件、表格：见 4.4 节。
- TCP 网络与断线重连：见 4.5 节。
- HTTP 异步网络模块：见 4.6 节。
- 车辆管理表格交互：见 4.7 节。
- QThread、FFmpeg、音视频处理：见 4.8 节。
- 罗技 G923 外设 SDK 动态封装：见 4.10 节，注意当前为已封装、未接入主流程状态。

## 1. 项目概述

这是一个基于 Qt Widgets 开发的 Windows 桌面客户端，系统名称为“车辆监控及远程驾驶系统”。程序先进入登录界面，登录成功后进入主界面，主界面包含系统主页、车辆监控、车辆列表三个主要页面，实现以下能力：

- 用户/客户端登录校验，登录信息本地持久化。
- 车辆注册、删除、编辑、搜索、列表展示。
- 可在线车辆状态轮询，展示驾驶模式、电量、位置、速度。
- HTTP 业务接口与 TCP 长连接控制通道。
- 连接指定车辆，切换车辆前后左右的视频流。
- 自动驾驶/手动驾驶模式切换，远程紧急制动状态发送。
- FFmpeg 拉流、解码、格式转换、UI 显示。
- 罗技 G923 方向盘 SDK 的动态库封装接口。

## 2. 工程与技术栈

> JD 标注：【已实现】C++11、Qt Widgets、Qt Network、Qt WebEngineWidgets、Qt WebSockets、FFmpeg、多线程；【部分实现/相关】Model/View 基础、Windows 平台；【代码未体现】QML/Qt Quick、QOpenGL、GStreamer、Qt 国际化、嵌入式 Linux。

| 项目       | 内容                                                         |
| ---------- | ------------------------------------------------------------ |
| 语言标准   | C++11，`VrdsClient.pro` 中 `CONFIG += c++11`                 |
| Qt 版本    | Qt 5.14.2，MSVC2017 64bit，Release/Debug 构建目录可见        |
| Qt 模块    | `core`、`gui`、`widgets`、`network`、`webenginewidgets`、`webchannel`、`websockets` |
| 构建方式   | qmake，工程文件：`VrdsClient/VrdsClient.pro`                 |
| UI 技术    | Qt Designer `.ui`、`QStackedWidget`、`QTableWidget`、Material Widgets |
| 视频处理   | FFmpeg 解码与 `swscale` 格式转换                             |
| 网络通信   | `QNetworkAccessManager` 异步 HTTP、`QTcpSocket` 长连接       |
| 游戏方向盘 | 罗技 Logitech Steering Wheel SDK，静态库 + 动态库包装        |
| 本地配置   | JSON 配置文件，路径定义在 `VrdsClient/msg.h`                 |

### 2.1 工程源文件清单

| 源文件                                                       | 职责                                                     |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| `VrdsClient/main.cpp`                                        | 应用入口，开启高 DPI 支持，启动登录窗口                  |
| `VrdsClient/VrdsClient.pro`                                  | 工程配置，Qt 模块、FFmpeg、罗技 SDK、MaterialWidget 链接 |
| `VrdsClient/msg.h`                                           | 服务器地址、端口、协议头、车辆状态与控制协议结构体       |
| `VrdsClient/myfunc.h`                                        | 通用工具函数，IP 连通性检测、字符串尾部零去除、数值限幅  |
| `VrdsClient/Login/login.ui`                                  | 登录窗口界面                                             |
| `VrdsClient/Login/login.h`                                   | 登录窗口类声明                                           |
| `VrdsClient/Login/login.cpp`                                 | 登录、记住密码、HTTP 登录、用户信息获取、窗口跳转        |
| `VrdsClient/Client/vrdsclient.ui`                            | 主窗口界面                                               |
| `VrdsClient/Client/vrdsclient.h`                             | 主窗口类、槽函数、成员声明                               |
| `VrdsClient/Client/vrdsclient.cpp`                           | 主窗口初始化、页面切换、HTTP、TCP、车辆管理、视频控制    |
| `VrdsClient/Client/mediaplayerthread.h`                      | FFmpeg 视频解码播放线程类                                |
| `VrdsClient/G923/G923.h`                                     | 罗技 G923 SDK 动态加载与函数指针封装声明                 |
| `VrdsClient/G923/G923.cpp`                                   | 罗技 SDK 函数指针赋值与接口包装实现                      |
| `VrdsClient/G923/LogitechSteeringWheelLib.h`                 | 罗技官方 SDK 头文件                                      |
| `VrdsClient/pic.qrc`                                         | 图片资源                                                 |
| `VrdsClient/qss.qrc`                                         | QSS 资源                                                 |
| `VrdsClient/Resource/qss/psblack.css`                        | 黑色主题样式，含控件状态图片                             |
| `VrdsClient/lib/ffmpeg`                                      | 编译期链接的 FFmpeg 头文件、静态导入库、动态库           |
| `VrdsClient/lib/MaterialWidget`                              | Material 风格控件库                                      |
| `VrdsClient/G923/LogitechSteeringWheelLib.lib`、`LogitechSteeringWheelEnginesWrapper.dll` | 罗技 SDK 导入库与运行库                                  |

## 3. 整体架构

```text
main.cpp
  └─ Login
       ├─ QNetworkAccessManager：/Login、/GetUserDataByUsername
       ├─ JSON 配置读写
       └─ 创建 VrdsClient 主窗口

VrdsClient
  ├─ InitForm：页面布局、侧边栏、表格初始化
  ├─ InitData：20ms 控制定时器、1s 在线车辆轮询
  ├─ InitStreamer：创建 3 个 FFmpeg 播放线程
  ├─ InitTcp：TCP 连接、断线重连、协议帧读取
  ├─ QNetworkAccessManager：车辆管理/监控/模式切换 HTTP
  └─ QTcpSocket：周期发送 Control_Msg 控制帧

MediaPlayerThread
  ├─ QThread::run：循环拉流与解码
  ├─ condition_variable：暂停/恢复
  ├─ avformat/avcodec/swscale：解码与 YUV→RGB 转换
  └─ ImageShow(QImage)：信号回主线程显示
```

## 4. 按源码文件精确拆解

### 4.1 应用入口 `main.cpp`

源码位置：`VrdsClient/main.cpp`

主要实现：

- `Qt::AA_EnableHighDpiScaling`、`Qt::AA_UseHighDpiPixmaps`：程序启动时开启高 DPI 支持，保证高分屏显示清晰。
- 创建 `Login` 窗口并调用 `show()`，登录成功后由登录窗口创建 `VrdsClient`。

可面试表述：熟悉 Qt 桌面程序启动流程和高分屏适配。

### 4.2 登录模块

> JD 标注：【已实现】Qt Widgets、Qt Network、信号槽、事件驱动、JSON 本地配置读写。

文件：`VrdsClient/Login/login.ui`、`VrdsClient/Login/login.cpp`、`VrdsClient/Login/login.h`

UI 定位：

| 控件                        | 位置           |
| --------------------------- | -------------- |
| 用户名输入框 `Username`     | `login.ui:137` |
| 密码输入框 `Password`       | `login.ui:209` |
| 客户端 ID 输入框 `clientID` | `login.ui:275` |
| 记住密码 `SavePassword`     | `login.ui:295` |
| 登录按钮 `btn_login`        | `login.ui:341` |

业务实现：

| 函数                                  | 行号            | 说明                                                   |
| ------------------------------------- | --------------- | ------------------------------------------------------ |
| `Login::Login`                        | `login.cpp:13`  | 读取 `config/config.json`，回填用户名、密码、客户端 ID |
| `Login::on_btn_login_clicked`         | `login.cpp:53`  | 登录主流程                                             |
| `Login::on_SavePassword_stateChanged` | `login.cpp:170` | 记住密码状态写入 JSON                                  |

登录流程拆解：

1. `MyFunc::checkIpIsOnline` 使用 `QProcess` 执行 ping，检测服务器可达性，见 `myfunc.h:10`。
2. 构造 `QNetworkRequest`，向 `/Login` 发送 `application/x-www-form-urlencoded` 的 POST 请求，见 `login.cpp:64`。
3. 通过 `QNetworkReply::finished` 异步接收响应，不阻塞 UI 线程。
4. 对返回码 `200`、`404`、`402`、`405`、`406` 分别做成功或错误提示。
5. 登录成功后读写 `config/config.json` 保存用户名、密码、客户端 ID，见 `login.cpp:84`。
6. 通过 `/GetUserDataByUsername` 查询 `userid`，见 `login.cpp:107`。
7. 查询成功后 `new VrdsClient` 并显示主窗口，关闭登录窗口，见 `login.cpp:127`。

可面试表述：熟悉 Qt `QNetworkAccessManager` 异步 HTTP 通信、`QJsonDocument`/`QJsonObject` JSON 解析、`QTextStream` 本地配置读写。

### 4.3 协议与公共定义

> JD 标注：【已实现】C++ 基础、STL、内存管理、二进制协议；【相关】`#pragma pack(1)` 说明结构体内存布局理解。

文件：`VrdsClient/msg.h`

| 定义                                | 行号                   | 说明                                                         |
| ----------------------------------- | ---------------------- | ------------------------------------------------------------ |
| `#pragma pack(1)`                   | `msg.h:3`              | 按 1 字节对齐，保证结构体直接对应网络二进制帧                |
| `config_file_path`、`map_file_path` | `msg.h:7`              | 配置和地图文件路径                                           |
| 服务器 IP/HTTP 端口/TCP 端口        | `msg.h:10`             | 全局通信参数统一配置                                         |
| `HEADER` 系列                       | `msg.h:14`             | 协议帧头定义                                                 |
| `ManualDRIVE`、`AUTODRIVE`          | `msg.h:22`             | 驾驶模式枚举                                                 |
| `VehicleStatus`                     | `msg.h:35`             | 车辆状态帧：ID、心跳、经纬度、速度、转角、电量、驾驶模式、方位角、AEB 状态 |
| `Control_Msg`                       | `msg.h:50`             | 控制指令帧：用户 ID、客户端 ID、被控车辆、目标速度/角度、刹车、模式、推流使能 |
| `BASIC`                             | `msg.h:64`             | 通用最小帧，用于读取帧头判断消息类型                         |
| `extern` 全局变量                   | `msg.h:30`、`msg.h:68` | 用户信息和协议全局对象                                       |

文件：`VrdsClient/myfunc.h`

公共工具函数：

| 函数                          | 行号          | 说明                   |
| ----------------------------- | ------------- | ---------------------- |
| `MyFunc::checkIpIsOnline`     | `myfunc.h:10` | 用 ping 检测服务器网络 |
| `MyFunc::removeTrailingZeros` | `myfunc.h:23` | 去掉经纬度小数尾部 0   |
| `MyFunc::limit_ab`            | `myfunc.h:39` | 数值上下限约束         |

可面试表述：熟悉 `#pragma pack` 二进制协议定义，能用 `memcpy` + `reinterpret_cast` 收发结构体，知道网络帧解析需要关注结构体内存对齐。

### 4.4 主窗口框架与初始化

> JD 标注：【已实现】Qt Widgets、信号槽、事件机制、表格控件；【部分实现/相关】Model/View 通过 `QTableWidget` 落地，未实现自定义 Model。

文件：`VrdsClient/Client/vrdsclient.cpp`

构造函数与析构：

| 函数                                  | 行号                  | 说明                                                       |
| ------------------------------------- | --------------------- | ---------------------------------------------------------- |
| `VrdsClient::VrdsClient`              | `vrdsclient.cpp:26`   | 依次调用 `InitData`、`InitForm`、`InitStreamer`、`InitTcp` |
| `VrdsClient::~VrdsClient`             | `vrdsclient.cpp:40`   | 停止定时器、关闭 TCP、统一 `deleteLater` 清理              |
| `VrdsClient::keyPressEvent`           | `vrdsclient.cpp:59`   | `~` 打开侧边栏，F1/F2/F3 切换页面                          |
| `VrdsClient::resizeEvent`             | `vrdsclient.cpp:87`   | 窗口缩放事件处理入口                                       |
| `VrdsClient::InitForm`                | `vrdsclient.cpp:93`   | 页面、侧边栏、表格、控件状态初始化                         |
| `VrdsClient::on_btn_openFunc_clicked` | `vrdsclient.cpp:1302` | 打开功能侧边栏                                             |

`InitForm` 中的关键实现：

- `QtMaterialDrawer` 创建侧边栏，设置外部点击关闭、遮罩模式，见 `vrdsclient.cpp:106`。
- 侧边栏包含“系统主页”“车辆监控”“车辆列表”，通过 `QStackedWidget` 切换页面，见 `vrdsclient.cpp:140`。
- 车辆列表配置 10 列，表头、行高、选中方式、颜色、编辑策略，见 `vrdsclient.cpp:208`。
- 在线车辆表 `tb_ActiveVeh` 配置 4 列，见 `vrdsclient.cpp:245`。
- 视频控件设置 `QSizePolicy::Ignored` 和 `setScaledContents(true)`，避免视频画面撑开布局，见 `vrdsclient.cpp:194`。

UI 主窗口文件：`VrdsClient/Client/vrdsclient.ui`

| 页面/控件                                                    | 行号                              |
| ------------------------------------------------------------ | --------------------------------- |
| `stackedWidget` 页面容器                                     | `vrdsclient.ui:55`                |
| 系统主页 `main_page`                                         | `vrdsclient.ui:65`                |
| 车辆监控页 `monitor_page`                                    | `vrdsclient.ui:181`               |
| 前/左后/右后视频 `frontVideo`、`leftbackVideo`、`rightbackVideo` | `vrdsclient.ui:225`、`258`、`277` |
| 地图控件 `Map`                                               | `vrdsclient.ui:305`               |
| 在线车辆表 `tb_ActiveVeh`                                    | `vrdsclient.ui:341`               |
| 日志 `log`                                                   | `vrdsclient.ui:344`               |
| 被控车辆下拉框 `remoteCar`                                   | `vrdsclient.ui:386`               |
| 连接车辆按钮 `btn_connectVeh`                                | `vrdsclient.ui:396`               |
| 驾驶模式状态 `label_drivingMode`                             | `vrdsclient.ui:411`               |
| 速度显示 `speed`                                             | `vrdsclient.ui:440`               |
| 模式切换按钮 `btn_Mode`                                      | `vrdsclient.ui:517`               |
| 紧急制动 `AEB_button`                                        | `vrdsclient.ui:559`               |
| 车辆列表页 `VehInfo_page`                                    | `vrdsclient.ui:645`               |
| 车辆表格 `tb_VehList`                                        | `vrdsclient.ui:845`               |
| 新增车辆页 `VehRegisterPage`                                 | `vrdsclient.ui:849`               |
| 方向盘配置页 `page`                                          | `vrdsclient.ui:1006`              |

可面试表述：熟练使用 Qt Designer、`QStackedWidget` 多页面布局、`QTableWidget` 表格交互、Material Widgets 侧边栏，能实现键盘快捷键切换和窗口缩放适配。

### 4.5 TCP 长连接与控制指令发送

> JD 标注：【已实现】Qt Network、TCP 长连接、定时器、多线程中的线程安全考虑。

文件：`VrdsClient/Client/vrdsclient.cpp`

初始化：

| 函数                            | 行号                  | 说明                                     |
| ------------------------------- | --------------------- | ---------------------------------------- |
| `VrdsClient::InitTcp`           | `vrdsclient.cpp:482`  | 创建 `QTcpSocket`，启动 500ms 重连定时器 |
| `VrdsClient::readfromServer`    | `vrdsclient.cpp:510`  | 读取 TCP 数据并解析二进制帧              |
| `VrdsClient::CommuteWithServer` | `vrdsclient.cpp:1196` | 周期组装并发送 `Control_Msg`             |

关键逻辑：

- `connectTcpTimer` 每 500ms 检查 TCP 状态，未连接则 `connectToHost`，见 `vrdsclient.cpp:487`。
- `disconnected` 信号触发日志提示，并重新启动重连定时器，见 `vrdsclient.cpp:499`。
- `readfromServer` 使用 `readAll()` 后，将数据 `memcpy` 到 `BASIC`，再根据 `header` 判断是否拷贝为 `VehicleStatus`，见 `vrdsclient.cpp:513`。
- `CommuteWithServer` 由 20ms 定时器驱动，填充 `CtrlMsg.header`、`identity`、`heart_beat`、`remote_id`、`client_id`、`brake_enable` 和 `StreamPushEnable`，TCP 已连接时 `write` 发送，见 `vrdsclient.cpp:1196`、`1298`。

可面试表述：熟悉 `QTcpSocket` 长连接、定时重连、断线重连、`readyRead` 粘包/帧解析、结构体内存拷贝发送。

### 4.6 HTTP 业务接口

> JD 标注：【已实现】Qt 网络模块、异步网络通信、JSON 数据解析。

主窗口 HTTP 请求分布在 `vrdsclient.cpp`，登录请求在 `login.cpp`。异步请求统一使用 `QNetworkAccessManager`，响应完成后通过 `finished` Lambda 处理，并调用 `reply->deleteLater()` 避免内存泄漏。

| 接口                           | 行号                          | 功能                            |
| ------------------------------ | ----------------------------- | ------------------------------- |
| `/Login`                       | `login.cpp:64`                | 登录认证                        |
| `/GetUserDataByUsername`       | `login.cpp:107`               | 获取用户 ID                     |
| `/GetActiveVehByUserID`        | `vrdsclient.cpp:347`          | 1s 轮询在线车辆状态             |
| `/RegisterVehicle`             | `vrdsclient.cpp:550`          | 注册车辆                        |
| `/GetVehicleDataByUsername`    | `vrdsclient.cpp:617`          | 刷新车辆列表                    |
| `/UpdateClientFrontStreamAddr` | `vrdsclient.cpp:749`          | 修改前摄像头推流地址            |
| `/UpdateClientLeftStreamAddr`  | `vrdsclient.cpp:756`          | 修改左摄像头推流地址            |
| `/UpdateClientRightStreamAddr` | `vrdsclient.cpp:763`          | 修改右摄像头推流地址            |
| `/UpdateClientBackStreamAddr`  | `vrdsclient.cpp:770`          | 修改后摄像头推流地址            |
| `/DeleteVehicle`               | `vrdsclient.cpp:833`          | 删除车辆                        |
| `/RequestRemote`               | `vrdsclient.cpp:1003`         | 申请远程控制，进入自动/远程模式 |
| `/ReleaseRemote`               | `vrdsclient.cpp:1040`         | 释放远程控制                    |
| `/ReleaseMonitoring`           | `vrdsclient.cpp:1372`、`1488` | 释放车辆监控                    |
| `/RequestMonitoring`           | `vrdsclient.cpp:1398`         | 申请车辆监控                    |
| `/GetVehStreamAddrByUserVehID` | `vrdsclient.cpp:1428`         | 获取被控车辆各摄像头推流地址    |

在线车辆轮询逻辑位于 `InitData` 内：

- `GetActiveVehTimer` 每 1000ms 请求一次 `/GetActiveVehByUserID`，见 `vrdsclient.cpp:344`。
- 响应 JSON 解析为 `vehs` 数组，写入 `tb_ActiveVeh`。
- 驾驶模式用红色/蓝色字体区分“手动/自动”。
- 通过本地 `onlineCar` 哈希表记录车辆上下线，离线车辆从下拉框移除，见 `vrdsclient.cpp:363`、`438`。
- 当前被控车辆在线时自动选中，并在模式切换后显示当前速度、AEB 告警日志。

可面试表述：熟悉 Qt `QNetworkAccessManager` 异步 GET/POST、URL 参数拼装、JSON 列表刷新、定时轮询、在线状态差量更新。

### 4.7 车辆管理模块

> JD 标注：【已实现】Qt Widgets 表格编辑、信号槽、业务弹窗、HTTP 增删改查。

| 槽函数                           | 行号                 | 功能                     |
| -------------------------------- | -------------------- | ------------------------ |
| `on_btn_VehAdd_clicked`          | `vrdsclient.cpp:523` | 新增车辆                 |
| `on_btn_vehinfo_refresh_clicked` | `vrdsclient.cpp:609` | 刷新车辆列表             |
| `on_btn_veh_add_clicked`         | `vrdsclient.cpp:700` | 列表/新增页切换          |
| `on_btn_veh_edit_clicked`        | `vrdsclient.cpp:712` | 进入/退出编辑模式        |
| `on_tb_VehList_cellChanged`      | `vrdsclient.cpp:740` | 单元格修改后更新推流地址 |
| `on_btn_veh_delete_clicked`      | `vrdsclient.cpp:813` | 删除车辆                 |
| `on_btn_search_clicked`          | `vrdsclient.cpp:871` | 车辆 ID/推流地址查找     |

关键实现：

- 新增车辆前做必填和非空校验，POST 到 `/RegisterVehicle`，根据服务端返回码提示车辆重复或推流地址冲突，见 `vrdsclient.cpp:523`。
- 车辆列表展示 ID、活动状态、驾驶模式、经纬度、电量、用户名、前后左右推流地址。
- 编辑模式只允许修改推流地址列，通过 `QTableWidgetItem::setFlags` 控制可编辑状态，见 `vrdsclient.cpp:712`。
- 编辑推流地址后自动定位到对应更新接口，见 `vrdsclient.cpp:740`。
- 删除车辆需要二次确认和密码校验，通过 `QMessageBox`、`QInputDialog` 实现，见 `vrdsclient.cpp:813`。
- 查找支持车辆 ID 和推流地址，数字 ID 不足 5 位时自动补零匹配。

可面试表述：熟悉表格编辑权限控制、行选择、搜索滚动定位、业务弹窗、HTTP 增删改查完整闭环。

### 4.8 视频流播放与切换

> JD 标注：【已实现】QThread、std::mutex、condition_variable、FFmpeg、swscale、信号槽跨线程更新 UI；此段是对应“多线程编程”和“音视频处理加分项”的核心证据。

视频线程实现文件：`VrdsClient/Client/mediaplayerthread.h`

当前启用的类实现范围：`mediaplayerthread.h:21` 到约 `171`，文件后半段是旧版本注释备份，不参与编译。

类能力：

| 接口          | 行号                      | 说明                         |
| ------------- | ------------------------- | ---------------------------- |
| 构造函数      | `mediaplayerthread.h:25`  | 保存 RTMP 地址               |
| `StreamStart` | `mediaplayerthread.h:29`  | 打开流播放控制并唤醒等待线程 |
| `StreamStop`  | `mediaplayerthread.h:33`  | 停止拉流                     |
| `ResetUrl`    | `mediaplayerthread.h:36`  | 切换拉流地址                 |
| `run`         | `mediaplayerthread.h:45`  | FFmpeg 拉流解码主循环        |
| `ImageShow`   | `mediaplayerthread.h:166` | 解码帧信号                   |

`run` 解码流程：

1. 当 `StreamFlag` 为 false 时，`std::unique_lock` + `condition_variable::wait` 挂起线程，见 `mediaplayerthread.h:58`。
2. `avformat_alloc_context`、`avformat_open_input` 打开 RTMP 流，见 `mediaplayerthread.h:67`。
3. `avformat_find_stream_info` 获取流信息，遍历查找视频流，见 `mediaplayerthread.h:74`。
4. `avcodec_find_decoder`、`avcodec_open2` 打开解码器，见 `mediaplayerthread.h:91`。
5. `av_read_frame` 读取 AVPacket，`avcodec_send_packet` + `avcodec_receive_frame` 解码，见 `mediaplayerthread.h:120`、`128`、`133`。
6. `sws_getContext` + `sws_scale` 将视频帧转成 RGB24，见 `mediaplayerthread.h:110`、`139`。
7. 使用 RGB buffer 构造 `QImage`，发射 `ImageShow(image)` 信号，见 `mediaplayerthread.h:145`、`151`。
8. 流断开后释放 `AVFrame`、`SwsContext`、解码器上下文，短暂等待后继续外层循环。

主窗口线程创建与信号连接：

| 函数                       | 行号                 | 说明                                      |
| -------------------------- | -------------------- | ----------------------------------------- |
| `VrdsClient::InitStreamer` | `vrdsclient.cpp:271` | 创建前/左后/右后 3 个 `MediaPlayerThread` |
| `frontCamera` 信号连接     | `vrdsclient.cpp:278` | `QImage` 转 `QPixmap` 显示                |
| `leftbackCamera` 信号连接  | `vrdsclient.cpp:288` | 左后视频显示                              |
| `rightbackCamera` 信号连接 | `vrdsclient.cpp:296` | 右后视频显示                              |

连接车辆时的流切换顺序：

1. 点击“连接车辆”，先停止所有视频线程，见 `vrdsclient.cpp:1357`。
2. 若之前有被控车辆，先向 `/ReleaseMonitoring` 释放上一车辆监控，见 `vrdsclient.cpp:1372`。
3. 请求 `/RequestMonitoring` 绑定当前客户端与车辆，见 `vrdsclient.cpp:1398`。
4. 请求 `/GetVehStreamAddrByUserVehID` 获取各摄像头流地址，见 `vrdsclient.cpp:1428`。
5. 对每个播放线程 `ResetUrl` 后 `StreamStart`，实现多路视频无缝切换，见 `vrdsclient.cpp:1450`。

可面试表述：了解 FFmpeg 拉流、H.264/RTMP 视频流解码、YUV 到 RGB 转换、`QThread` 多线程播放、信号槽跨线程 UI 更新、`condition_variable` 暂停/恢复机制。

### 4.9 远程驾驶与紧急制动

文件：`VrdsClient/Client/vrdsclient.cpp`

| 槽函数                | 行号                  | 说明                              |
| --------------------- | --------------------- | --------------------------------- |
| `on_btn_Mode_clicked` | `vrdsclient.cpp:993`  | 自动驾驶/手动驾驶模式切换         |
| `CommuteWithServer`   | `vrdsclient.cpp:1196` | 20ms 周期发送控制帧，携带刹车状态 |

模式切换逻辑：

- 未连接车辆时禁止切换。
- 切到自动模式时 POST `/RequestRemote`，成功后设置 `CtrlMsg.control_mode = AUTODRIVE`，开启 AEB 按钮，见 `vrdsclient.cpp:1003`、`1027`。
- 切回手动时 POST `/ReleaseRemote`，成功后设置 `CtrlMsg.control_mode = ManualDRIVE`，关闭 AEB 按钮，见 `vrdsclient.cpp:1040`。
- `AEB_button` 勾选时在周期发送帧中将 `brake_enable` 置 1，见 `vrdsclient.cpp:1202`。
- 在线车辆轮询中若当前车辆 AEB 字段非空且非 0，会在日志区追加 AEB 告警，见 `vrdsclient.cpp:420`。

可面试表述：熟悉模式状态机、安全指令下发、紧急制动使能位设计、20ms 高频控制帧发送。

### 4.10 罗技 G923 方向盘封装

> JD 标注：【已实现】动态库加载、SDK 函数指针封装、外设驱动式接口抽象；注意当前主流程未调用，属已封装未接入状态。

文件：`VrdsClient/G923/G923.h`、`VrdsClient/G923/G923.cpp`

工程链接配置见 `VrdsClient.pro:48`，加载和函数解析入口在 `G923::GetFunc`，`G923.cpp:304`。

核心封装：

- 使用 `QLibrary` 动态加载 `LogitechSteeringWheelEnginesWrapper.dll`，见 `G923.cpp:306`。
- 将 SDK 中约 48 个 API 映射为函数指针，并在 `GetFunc` 中通过 `resolve` 批量绑定，见 `G923.cpp:11` 起。
- 提供方向盘状态读取 `GetWheelData`、连接初始化、状态更新、按钮检测、力反馈、路面效果、LED、软停止等接口。
- `G923.h` 中对所有接口均有中文注释，工程内也保留方向盘速度、转角、档位、踏板输入到 `CtrlMsg` 的注释版本实现。

源码现状备注：`G923` 类在当前主流程中被注释，未在 `Login` 或 `VrdsClient` 中主动调用；该模块属于“已完成 SDK 封装，待接入主控制流程”的扩展能力。若岗位 JD 关注外设接入，可描述为：负责完成罗技方向盘 SDK 的动态加载与接口封装。

### 4.11 地图与 WebEngine 预留

相关代码：

- UI 中保留 `QWebEngineView Map`，见 `vrdsclient.ui:305`。
- `msg.h:8` 定义 `map_file_path`。
- `vrdsclient.h:9`、`.pro` 已链接 `webenginewidgets` 和 `webchannel`。
- `InitMap`、`runJavaScript` 地图标记代码为注释保留，见 `vrdsclient.cpp:261`、`406`。

源码现状备注：当前主流程没有调用 `InitMap`，地图模块处于界面布局和扩展代码保留状态，不是已启用功能。

## 5. 可直接用于简历/面试的项目描述

以下描述建议根据真实岗位 JD 做措辞调整：

```text
使用 Qt/C++ 开发“车辆监控及远程驾驶系统”Windows 桌面客户端，负责登录认证、车辆管理、远程监控、控制帧下发、视频流播放等完整功能。

技术实现包括：
1. 使用 qmake 管理 Qt 5 Widgets 工程，集成 Qt Network、Qt WebEngineWidgets、Qt WebChannel、Qt WebSockets。
2. 基于 QStackedWidget、QTableWidget、Material Widgets 实现多页面桌面 UI，提供系统主页、车辆监控、车辆列表、车辆注册页面。
3. 使用 QNetworkAccessManager 异步实现登录、车辆增删改查、监控申请、远程控制等 REST 接口调用，并完成异常码处理和定时轮询。
4. 使用 QTcpSocket 实现服务器长连接，封装 1 字节对齐的二进制控制/状态帧，通过定时器按 20ms 周期发送控制消息，支持断线自动重连。
5. 基于 FFmpeg 拉取 RTMP 视频流，在 QThread 中完成解码、YUV 转 RGB，并通过 ImageShow 信号将 QImage 异步显示到前/左后/右后视频区。
6. 支持连接车辆时多路视频地址切换，以及自动驾驶/手动驾驶模式切换和远程紧急制动。
7. 使用 QLibrary 封装罗技 G923 方向盘 SDK，将力反馈、按钮、踏板等接口抽象为可复用模块。
```

## 6. 可追问的技术点

如果面试官基于这份源码提问，可以重点准备以下位置：

| 追问点                                      | 源码位置                                     |
| ------------------------------------------- | -------------------------------------------- |
| 为什么结构体要 `#pragma pack(1)`            | `msg.h:3`                                    |
| TCP 粘包/断帧如何处理                       | `vrdsclient.cpp:510`                         |
| 控制帧发送频率怎么控制                      | `vrdsclient.cpp:339`、`1196`                 |
| 断线后如何自动重连                          | `vrdsclient.cpp:487`                         |
| 为什么网络回复用 `deleteLater`              | `login.cpp:135`、`vrdsclient.cpp:604` 等多处 |
| 视频线程如何暂停和恢复                      | `mediaplayerthread.h:29`、`58`               |
| 切换视频时如何避免多路同时播放错乱          | `vrdsclient.cpp:1357`、`1450`                |
| 车辆在线状态如何差量更新                    | `vrdsclient.cpp:344`、`438`                  |
| 罗技 SDK 为什么用函数指针和 `QLibrary`      | `G923.cpp:304`                               |
| 当前哪些功能是启用状态、哪些是预留/注释状态 | 见第 4.10、4.11 节                           |

## 7. 源码现状与交付提示

以下内容便于你在面试前自查，确保口头描述与代码一致：

- 当前启用主流程：登录、主窗口页面、车辆管理、HTTP 轮询、TCP 控制帧、三路 FFmpeg 视频播放、模式切换和 AEB 刹车状态发送。
- 注释保留但未启用：地图 `InitMap`、地图 marker 调用、罗技 G923 接入主流程、QWebSocket 方向盘数据通道、系统设置页部分按钮。
- 如果 JD 要求“多媒体/直播/低延迟”，优先讲第 4.8 节；如果 JD 要求“硬件外设/驱动/SDK”，优先讲第 4.10 节；如果 JD 要求“网络通信/协议/多线程”，优先讲第 4.5、4.6、4.7、4.8 节。