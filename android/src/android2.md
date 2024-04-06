ndrid系统的体系结构设计为多层结构，这种结构在给用户提供安全保护的同时还保持了开放平台的灵活性。如下图所示：



Google官方提供的Android系统的四层架构图

从上到下进行简单介绍：

## 一、应用层 Applications：
应用层由运行在Android设备上的所有应用构成，包括预装的系统应用和自己安装的第三方应用。大部分是由Java语言编写并运行在Dalvik虚拟机中，另一部分应用是通过c++/c语言编写的本地应用。但无论采用何种编程语言，两类应用运行的安全环境相同，都在应用沙箱中运行。而程序员正是在这层中，通过Android提供的组件和API进行开发，从而编写出形形色色的app。
## 二、应用框架层
### Application Framework
集中体现Android系统系统的组件设计思想，是Android应用开发的核心，为开发者开发应用时提供基础的API框架。框架层由多个系统服务组成。我们知道Android应用是由若干个组件构成，组件与组件之间的通信是通过框架层提供的服务集中调度和传递消息实现的，而不是组件之间直接进行的。
### View System 
主要用于UI设计，包括List、Grid、Text、Button、Webview等。
### Activity Manager Service -AMS 
负责管理应用程序中的activity的生命周几以及提供activity之间切换功能等 Intent相关。
### Windows Manager Service-WMS 
用于管理所有的窗口程序，如Dialog、Toast等。
### Recource Manager 
提供非代码资源的管理 如布局文件、图形、字符串资源文件等。
### Location Manager 
负责与定位功能相关功能
### Content Providers
提供了一组通用的数据访问接口，可用于应用程序间的内容交互，比如获取手机联系人数据等。
### Notification Manager
用户管理手机状态栏中的自定义信息等。
### Telephony Manager 
手机底层功能管理模块，可用于获取手机串号或者调用短信功能
### Pacakage Manager 
Android系统内的包管理模块，负责管理安装的应用程序。
### XMPP Service
用于主持XMPP协议的服务，比如与Google Talk通信等
## 三、类库层
主要由类库 Libraries 和Android运行时 Android Runtime 两部分组成：
### 1.类库 Libraries
由一系列的二进制动态库构成，大部分来源于优秀的第三方类库，另一部分是系统原生类库，通常使用c/c++语言开发。（因为java代码无法直接调用c/c++驱动代码,所以在这一层,系统通过封装了一系列的函数库供上层使用.）以下列举一些比较重要的类库的功能，以供了解：

#### Surface Manager: 
负责管理显示与存取操作间的互动，另外也负责将2D绘图与3D绘图进行显示上的合成
#### Media Framework: 
一个开源的多媒体框架,允许我们创造出更高质量与全新的播放器效果
#### SQLite: 
安卓自带的数据库，是一个嵌入式的数据库
#### OpenGL ES: 
是 OpenGL 三维图形 API 的子集，针对手机、PDA和游戏主机等嵌入式设备而设计。3D效果库
#### FreeType:
一个完全免费（开源）的、高质量的且可移植的字体引擎。支持位图、矢量、字体等
#### SGL:
2D图形引擎库
#### SSL:
位于TCP/IP协议与各种应用层协议之间，为数据通信提供支持。是安全数据通信的支持。
#### WebKit:
是一个开源的浏览器引擎。
#### Libc:
c层中最基本的函数库
### 2.Android运行时
Android Runtime 是由Java核心类库（Core Libraries）和Android虚拟机（Dalvik）共同构成。

Java核心类库包括框架层和应用层所用到的基本Java库。

Dalvik是为Android量身打造的Java虚拟机，它与标准Java虚拟机JVM的差别在于Dalvik是基于寄存器设计的，而JVM是基于栈结构设计的；JVM通过解码class文件（java编译生成的的:.java---.class 的class文件）中的内容来运行程序；而Dalvik运行时是由java字节码文件进一步转化而来的文件，，并被打包成一个DEX可执行文件，Dalvik虚拟机通过解释DEX文件来执行这些字节码 ，即android的class 文件实际上只是编译过程中的中间目标文件，需要链接成dex 文件后才能在dalvik 上运行；Dalvik能够更快的编译较大的应用程序，允许在有限的内存空间中同时运行多个虚拟机的实例，每一个Dalvik应用作为一个独立的Linux进程执行，这样可以防止某一虚拟机崩溃时所有的应用都被关闭。

## 四、系统内核层 Linux Kernel
Android内核具有和标准的Linux内核一样的功能，主要实现内存管理、进程调度、进程间通信（Android增加了一种进程间的通信机制IPC Binder）、设备驱动（Display Driver: 显示驱动；Camera Driver: 照相机驱动；Flash Memory Driver: 闪存驱动；Binder Driver: IPC通讯驱动；KeyPad Driver: 键映射驱动；Wifi Driver:Wifi驱动；Audio Driver:音频驱动；Power Management:电量管理驱动）等
