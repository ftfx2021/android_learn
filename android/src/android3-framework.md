# Android Framework中的Application Framework层介绍「建议收藏」



Android的四层架构相比大家都很清楚，老生常谈的说一下分别为：

Linux2.6内核层，核心库层，应用框架层，应用层。我今天重点介绍一下应用框架层Framework。

        Framework层为我们开发应用程序提供了非常多的API，我们通过调用特殊的API构造我们的APP，满足我们业务上的需求。写APP的人都知道，学习Android开发的第一步就是去学习各种各样的API，什么Activity,Service,Notification等。这些都是framework提供给我们的，那么我就详细的讲讲Framework到底在整个Android架构中扮演着什么角色。

Framework功能

         Framework其实可以简单的理解为一些API的库房，android开发人员将一些基本功能实现，通过接口提供给上层调用，可以重复的调用

         我们可以称Framework层才真正是Java语言实现的层，在这层里定义的API都是用Java语言编写。但是又因为它包含了JNI的方法，JNI用C/C++编写接口，根据函数表查询调用核心库层里的底层方法，最终访问到Linux内核。那么Framework层的作用就有2个。

1.用Java语言编写一些规范化的模块封装成框架，供APP层开发者调用开发出具有特殊业务的手机应用。

2.用Java Native Interface调用core lib层的本地方法，JNI的库是在Dalvik虚拟机启动时加载进去的，Dalvik会直接去寻址这个JNI方法，然后去调用。

        2种方式的结合达到了Java方法和操作系统的相互通信。Android为什么要用Java编写Framework层呢？直接用C或C++不是更好？有关专家给出了如下解释：

      C/C++过于底层，开发者要花很多的经历对C/C++的语言研究清楚，例如C/C++的内存机制，如果稍不注意，就会忘了开启或者释放。而Java的GC会自动处理这些，省去了很多的时间让开发者专注于自己的业务。所以才会从C/C++的底层慢慢向上变成了JAVA的开发语言，该层通过JNI和核心运行库层进行交互。

         其实这些也是Java能发展这么迅速的原因，面对对象语言的优势。不用太关注内存，放心大胆的去做实现，才有时间去创造新的事物。

Android Framework中的Application Framework层介绍「建议收藏」

Framework API



Activity Manager


用来管理应用程序生命周期并提供常用的导航回退功能。



Window Manager


提供一些我们访问手机屏幕的方法。屏幕的透明度、亮度、背景。



Content Providers


使得应用程序可以访问另一个应用程序的数据（如联系人数据库)， 或者共享它们自己的数据。



View System


可以用来构建应用程序， 它包括列表（Lists)，网格（Grids)，文本框（Text boxes)，按钮（Buttons)， 甚至可嵌入的web浏览器。



Notification Manager


使得应用程序可以在状态栏中显示自定义的提示信息。



Package Manager


提供对系统的安装包的访问。包括安装、卸载应用，查询permission相关信息，查询Application相关信息等。



Telephony Manager


主要提供了一系列用于访问与手机通讯相关的状态和信息的方法，查询电信网络状态信息，sim卡的信息等。



Resource Manager


提供非代码资源的访问，如本地字符串，图形，和布局文件（Layout files )。



Location Manager


提供设备的地址位置的获取方式。很显然，GPS导航肯定能用到位置服务。



XMPP


可扩展通讯和表示协议。前身为Jabber，提供即时通信服务。例如推送功能,Google Talk。


每一层的介绍如下：
应用程序层（JAVA应用程序）：

该层提供一些核心应用程序包，例如电子邮件、短信、日历、地图、浏览器和联系人管理等。同时，开发者可以利用Java语言设计和编写属于自己的应用程序，而这些程序与那些核心应用程序彼此平等、友好共处。



应用程序框架层（JAVA框架）：

该层是Android应用开发的基础，开发人员大部分情况是在和她打交道。应用程序框架层包括活动管理器、窗口管理器、内容提供者、视图系统、包管理器、电话管理器、资源管理器、位置管理器、通知管理器和XMPP服务十个部分。在Android平台上，开发人员可以完全访问核心应用程序所使用的API框架。并且，任何一个应用程序都可以发布自身的功能模块，而其他应用程序则可以使用这些已发布的功能模块。基于这样的重用机制，用户就可以方便地替换平台本身的各种应用程序组件。



系统库和android运行时层（本地框架和JAVA运行环境）：

系统库包括九个子系统，分别是图层管理、媒体库、SQLite、OpenGLEState、FreeType、WebKit、SGL、SSL和libc。

Android运行时包括核心库和Dalvik虚拟机，前者既兼容了大多数Java语言所需要调用的功能函数，又包括了Android的核心库，比如android.os、android.net、android.media等等。后者是一种基于寄存器的java虚拟机，Dalvik虚拟机主要是完成对生命周期的管理、堆栈的管理、线程的管理、安全和异常的管理以及垃圾回收等重要功能。



LINUX内核层：

Android核心系统服务依赖于Linux内核，如安全性、内存管理、进程管理、网络协议栈和驱动模型。Linux内核也是作为硬件与软件栈的抽象层。

驱动：显示驱动、摄像头驱动、键盘驱动、WiFi驱动、Audio驱动、flash内存驱动、Binder（IPC）驱动、电源管理等。



2. android源码目录结构：



[plain]
view plain
copy

Android 5.1  
|– Makefile  
|– abi  
|– art  
|– bionic （bionic C库）  
|– bootable （启动引导相关代码）  
|– build （存放系统编译规则及generic等基础开发包配置）  
|– cts （Android兼容性测试套件标准）  
|– dalvik （dalvik JAVA虚拟机）  
|– developers  
|– development （应用程序开发相关）  
|– device  
|– docs  
|– external （android使用的一些开源的模组）  
|– frameworks （核心框架——java及C++语言）  
|– hardware （部分厂家开源的硬解适配层HAL代码）  
|– kernel  
|– libcore  
|– libnativehelper  
|– ndk  
|– out （编译完成后的代码输出与此目录）  
|– packages （应用程序包）  
|– pdk  
|– prebuilts （x86和arm架构下预编译的一些资源）  
|– sdk （sdk及模拟器）  
|– system （底层文件系统库、应用及组件——C语言）  
|– tools  
`– vendor （厂商定制代码）

bionic 目录  
|– benchmarks  
|– libc （C库）  
| |– arch-arm （ARM架构，包含系统调用汇编实现）  
| |– …  
| |– bionic （由C实现的功能，架构无关）  
| |– dns  
| |– include （头文件）  
| |– kernel （Linux内核中的一些头文件）  
| |– private （？一些私有的头文件）  
| |– stdio （stdio实现）  
| |– tools （几个工具）  
| |– tzcode （时区相关代码）  
| |– upstream-dlmalloc  
| |– upstream-freebsd  
| |– upstream-netbsd  
| |– upstream-openbsd  
|– zoneinfo （时区信息）  
|– libdl （libdl实现，dl是动态链接，提供访问动态链接库的功能）  
|– libm （libm数学库的实现，）  
|– libstdc++ （libstdc++ C++实现库）  
|– linker （动态链接器）  
– test

bootable 目录  
|– bootloader （适合各种bootloader的通用代码）  
| |– legacy （估计不能直接使用，可以参考）  
| `– lk  
`– recovery （系统恢复相关）  
| |– edify （升级脚本使用的edify脚本语言）  
| |– etc （init.rc恢复脚本）  
| |– minui （一个简单的UI）  
| |– minzip （一个简单的压缩工具）  
| |– mtdutils （mtd工具）  
| |– res （资源）  
| |– tools （工具）  
| | – ota （OTA Over The Air Updates升级工具）  
| – updater （升级器）

build目录  
|– core （核心编译规则）  
|– libs   
| – host （主机端库，有android “cp”功能替换）  
|– target （目标机编译对象）  
| |– board （开发平台）  
| |– product （开发平台对应的编译规则）  
– tools （编译中主机使用的工具及脚本）


dalvik目录 dalvik虚拟机  
|– dexdump （dex反汇编）  
|– dexgen  
|– dexlist （List all methods in all concrete classes in a DEX file.）  
|– docs （文档）  
|– dx （dx工具，将多个java转换为dex）  
|– hit （？java语言写成）  
|– opcode-gen  
|– tools （工具）  
– vm （虚拟机实现）

development 目录 （开发者需要的一些例程及工具）  
|– apps （一些核心应用程序）  
| |– BluetoothDebug （蓝牙调试程序）  
| |– CustomLocale （自定义区域设置）  
| |– Development （开发）  
| |– Fallback （和语言相关的一个程序）  
| |– FontLab （字库）  
| |– GestureBuilder （手势动作）  
| |– NinePatchLab （？）  
| |– OBJViewer （OBJ查看器）  
| |– SdkSetup （SDK安装器）  
| |– SpareParts （高级设置）  
| |– Term （远程登录）  
| – launchperf （？）  
|– build （编译脚本模板）  
|– cmds （有个monkey工具）  
|– docs （文档）  
|– host （主机端USB驱动等）  
|– ide （集成开发环境）  
|– libraries  
|– ndk （本地开发套件——c语言开发套件）  
|– samples （例程）  
| |– AliasActivity （？）  
| |– ApiDemos （API演示程序）  
| |– BluetoothChat （蓝牙聊天）  
| |– BrowserPlugin （浏览器插件）  
| |– BusinessCard （商业卡）  
| |– Compass （指南针）  
| |– ContactManager （联系人管理器）  
| |– CubeLiveWallpaper （动态壁纸的一个简单例程）  
| |– FixedGridLayout （像是布局）  
| |– GlobalTime （全球时间）  
| |– HelloActivity （Hello）  
| |– Home （Home）  
| |– JetBoy （jetBoy游戏）  
| |– LunarLander （貌似又是一个游戏）  
| |– MailSync （邮件同步）  
| |– MultiResolution （多分辨率）  
| |– MySampleRss （RSS）  
| |– NotePad （记事本）  
| |– RSSReader （RSS阅读器）  
| |– SearchableDictionary （目录搜索）  
| |– SimpleJNI （JNI例程）  
| |– SkeletonApp （空壳APP）  
| |– Snake （snake程序）  
| |– SoftKeyboard （软键盘）  
| |– Wiktionary （？维基）  
| – WiktionarySimple（？维基例程）  
|– scripts （脚本）  
|– sdk （sdk配置）  
|– sdk_overlay  
|– sys-img  
|– testrunner （？测试用）  
|– tools （一些工具）  
– tutorials

external 目录  
|– apache-http （网页服务器）  
|– bison （自动生成语法分析器，将无关文法转换成C、C++）  
|– blktrace （blktrace is a block layer IO tracing mechanism）  
|– bluetooth （蓝牙相关、协议栈）  
|– bsdiff （diff工具）  
|– bzip2 （压缩工具）  
|– dhcpcd （DHCP服务）  
|– e2fsprogs （EXT2文件系统工具）  
|– elfcopy （复制ELF的工具）  
|– elfutils （ELF工具）  
|– embunit （Embedded Unit Project）  
|– emma （java代码覆盖率统计工具）  
|– esd （Enlightened Sound Daemon，将多种音频流混合在一个设备上播放）  
|– expat （Expat is a stream-oriented XML parser.）  
|– fdlibm （FDLIBM (Freely Distributable LIBM)）  
详细目录介绍：

\system\app
这个里面主要存放的是常规下载的应用程序，可以看到都是以APK格式结尾的文件。在这个文件夹下的程序为系统默认的组件，自己安装的软件将不会出现在这里，而是\data\文件夹中。下面是详细的介绍:
\system\app\AlarmClock.apk 闹钟
\system\app\AlarmClock.odex
\system\app\Browser.apk 浏览器
\system\app\Browser.odex
\system\app\Bugreport.apk Bug报告
\system\app\Bugreport.odex
\system\app\Calculator.apk 计算器
\system\app\Calculator.odex
\system\app\Calendar.apk 日历
\system\app\Calendar.odex
\system\app\CalendarProvider.apk 日历提供
\system\app\CalendarProvider.odex
\system\app\Camera.apk 照相机
\system\app\Camera.odex
\system\app\com.amazon.mp3.apk 亚马逊音乐
\system\app\Contacts.apk 联系人
\system\app\Contacts.odex
\system\app\DownloadProvider.apk 下载提供
\system\app\DownloadProvider.odex
\system\app\DrmProvider.apk DRM数字版权提供
\system\app\DrmProvider.odex
\system\app\Email.apk 电子邮件客户端
\system\app\Email.odex
\system\app\FieldTest.apk 测试程序
\system\app\FieldTest.odex
\system\app\GDataFeedsProvider.apk GoogleData提供
\system\app\GDataFeedsProvider.odex
\system\app\Gmail.apk Gmail电子邮件
\system\app\Gmail.odex
\system\app\GmailProvider.apk Gmail提供
\system\app\GmailProvider.odex
\system\app\GoogleApps.apk 谷歌程序包
\system\app\GoogleApps.odex
\system\app\GoogleSearch.apk 搜索工具
\system\app\GoogleSearch.odex
\system\app\gtalkservice.apk GTalk服务
\system\app\gtalkservice.odex
\system\app\HTMLViewer.apk HTML查看器
\system\app\HTMLViewer.odex
\system\app\IM.apk 即使通讯组件包含MSN、yahoo通
\system\app\ImCredentialProvider.apk
\system\app\ImProvider.apk
\system\app\ImProvider.odex
\system\app\Launcher.apk 启动加载器
\system\app\Launcher.odex
\system\app\Maps.apk 电子地图
\system\app\Maps.odex
\system\app\MediaProvider.apk 多媒体播放提供
\system\app\MediaProvider.odex
\system\app\Mms.apk 短信、彩信
\system\app\Mms.odex
\system\app\Music.apk 音乐播放器
\system\app\Music.odex
\system\app\MyFaves.apk T-Mobile MyFaves程序
\system\app\MyFaves.odex
\system\app\PackageInstaller.apk apk安装程序
\system\app\PackageInstaller.odex
\system\app\Phone.apk 电话拨号器
\system\app\Phone.odex
\system\app\Settings.apk 系统设置
\system\app\Settings.odex
\system\app\SettingsProvider.apk 设置提供
\system\app\SettingsProvider.odex
\system\app\SetupWizard.apk 设置向导
\system\app\SetupWizard.odex
\system\app\SoundRecorder.apk 录音工具
\system\app\SoundRecorder.odex
\system\app\Street.apk 街景地图
\system\app\Street.odex
\system\app\Sync.apk 同步程序
\system\app\Sync.odex
\system\app\Talk.apk 语音程序
\system\app\Talk.odex
\system\app\TelephonyProvider.apk 电话提供
\system\app\TelephonyProvider.odex
\system\app\Updater.apk 更新程序
\system\app\Updater.odex
\system\app\Vending.apk 制造商信息
\system\app\Vending.odex
\system\app\VoiceDialer.apk 语音拨号器
\system\app\VoiceDialer.odex
\system\app\YouTube.apk Youtube视频
\system\app\YouTube.odex





\system\bin
这个目录下的文件都是系统的本地程序，从bin文件夹名称可以看出是binary二进制的程序，里面主要是Linux系统自带的组件，主要文件简单的分析介绍：
\system\bin\akmd
\system\bin\am
\system\bin\app_process 系统进程
\system\bin\dalvikvm Dalvik虚拟机宿主
\system\bin\dbus-daemon 系统BUS总线监控
\system\bin\debuggerd 调试器
\system\bin\debug_tool 调试工具
\system\bin\dexopt DEX选项
\system\bin\dhcpcd DHCP服务器
\system\bin\dumpstate 状态抓取器
\system\bin\dumpsys 系统抓取器
\system\bin\dvz
\system\bin\fillup
\system\bin\flash_image 闪存映像
\system\bin\hciattach
\system\bin\hcid HCID内核
\system\bin\hostapd
\system\bin\hostapd_cli
\system\bin\htclogkernel
\system\bin\input
\system\bin\installd
\system\bin\itr
\system\bin\linker
\system\bin\logcat Logcat日志打印
\system\bin\logwrapper
\system\bin\mediaserver
\system\bin\monkey
\system\bin\mountd 存储挂载器
\system\bin\netcfg 网络设置
\system\bin\ping Ping程序
\system\bin\playmp3 MP3播放器
\system\bin\pm 包管理器
\system\bin\qemud QEMU虚拟机
\system\bin\radiooptions 无线选项
\system\bin\rild RIL组件
\system\bin\sdptool
\system\bin\sdutil
\system\bin\service
\system\bin\servicemanager 服务管理器
\system\bin\sh
\system\bin\ssltest SSL测试
\system\bin\surfaceflinger 触摸感应驱动
\system\bin\svc 服务
\system\bin\system_server
\system\bin\telnetd Telnet组件
\system\bin\toolbox
\system\bin\wlan_loader
\system\bin\wpa_cli
\system\bin\wpa_supplicant



\system\etc
从文件夹名称来看保存的都是系统的配置文件，比如APN接入点设置等核心配置。
\system\etc\apns-conf.xml APN接入点配置文件
\system\etc\AudioFilter.csv 音频过滤器配置文件
\system\etc\AudioPara4.csv
\system\etc\bookmarks.xml 书签数据库
\system\etc\dbus.conf 总线监视配置文件
\system\etc\dhcpcd
\system\etc\event-log-tags
\system\etc\favorites.xml 收藏夹
\system\etc\firmware 固件信息
\system\etc\gps.conf GPS设置文件
\system\etc\hcid.conf  内核HCID配置文件
\system\etc\hosts 网络DNS缓存
\system\etc\init.goldfish.sh
\system\etc\location 定位相关
\system\etc\mountd.conf 存储挂载配置文件
\system\etc\NOTICE.html 提示网页
\system\etc\permissions.xml 权限许可
\system\etc\pvplayer.conf
\system\etc\security
\system\etc\wifi WLAN相关组件
\system\etc\dhcpcd\dhcpcd-hooks
\system\etc\dhcpcd\dhcpcd-run-hooks
\system\etc\dhcpcd\dhcpcd.conf
\system\etc\dhcpcd\dhcpcd-hooks\01-test
\system\etc\dhcpcd\dhcpcd-hooks\20-dns.conf
\system\etc\dhcpcd\dhcpcd-hooks\95-configured
\system\etc\firmware\brf6300.bin
\system\etc\location\gps
[page_break]
\system\etc\location\gps\location 定位相关
\system\etc\location\gps\nmea GPS数据解析
\system\etc\location\gps\properties
\system\etc\security\cacerts.bks
\system\etc\security\otacerts.zip OTA下载验证
\system\etc\wifi\Fw1251r1c.bin
\system\etc\wifi\tiwlan.ini
\system\etc\wifi\wpa_supplicant.conf WPA验证组件



\system\fonts
字体文件夹，除了标准字体和粗体、斜体外可以看到文件体积最大的可能是中文字库，或一些unicode字库，从T-Mobile G1上可以清楚的看到显示简体中文正常，其中DroidSansFallback.ttf文件大小
\system\fonts\DroidSans-Bold.ttf
\system\fonts\DroidSans.ttf
\system\fonts\DroidSansFallback.ttf
\system\fonts\DroidSansMono.ttf
\system\fonts\DroidSerif-Bold.ttf
\system\fonts\DroidSerif-BoldItalic.ttf
\system\fonts\DroidSerif-Italic.ttf
\system\fonts\DroidSerif-Regular.ttf
\system\framework
framework主要是一些核心的文件，从后缀名为jar可以看出是是系统平台框架。
\system\framework\am.jar
\system\framework\am.odex
\system\framework\android.awt.jar AWT库
\system\framework\android.awt.odex
\system\framework\android.policy.jar
\system\framework\android.policy.odex
\system\framework\android.test.runner.jar
\system\framework\android.test.runner.odex
\system\framework\com.google.android.gtalkservice.jar GTalk服务
\system\framework\com.google.android.gtalkservice.odex
\system\framework\com.google.android.maps.jar 电子地图库
\system\framework\com.google.android.maps.odex
\system\framework\core.jar 核心库，启动桌面时首先加载这个
\system\framework\core.odex
\system\framework\ext.jar
\system\framework\ext.odex
\system\framework\framework-res.apk
\system\framework\framework-tests.jar
\system\framework\framework-tests.odex
\system\framework\framework.jar
\system\framework\framework.odex
\system\framework\input.jar 输入库
\system\framework\input.odex
\system\framework\itr.jar
\system\framework\itr.odex
\system\framework\monkey.jar
\system\framework\monkey.odex
\system\framework\pm.jar 包管理库
\system\framework\pm.odex
\system\framework\services.jar
\system\framework\services.odex
\system\framework\ssltest.jar
\system\framework\ssltest.odex
\system\framework\svc.jar 系统服务
\system\framework\svc.odex

\system\lib

lib目录中存放的主要是系统底层库，如平台运行时库。
\system\lib\libaes.so
\system\lib\libagl.so
\system\lib\libandroid_runtime.so Android运行时库
\system\lib\libandroid_servers.so 系统服务组件
\system\lib\libaudio.so 音频处理
\system\lib\libaudioeq.so EQ均衡器
\system\lib\libaudioflinger.so 音频过滤器
\system\lib\libbluetooth.so 蓝牙组件
\system\lib\libc.so
\system\lib\libcamera.so 超相机组件
\system\lib\libcameraservice.so
\system\lib\libcorecg.so
\system\lib\libcrypto.so 加密组件
\system\lib\libctest.so
\system\lib\libcutils.so
\system\lib\libdbus.so
\system\lib\libdl.so
\system\lib\libdrm1.so DRM解析库
\system\lib\libdrm1_jni.so
\system\lib\libdvm.so
\system\lib\libexif.so
\system\lib\libexpat.so
\system\lib\libFFTEm.so
\system\lib\libGLES_CM.so
\system\lib\libgps.so
\system\lib\libhardware.so
\system\lib\libhgl.so
\system\lib\libhtc_ril.so
\system\lib\libicudata.so
\system\lib\libicui18n.so
\system\lib\libicuuc.so
\system\lib\liblog.so
\system\lib\libm.so
\system\lib\libmedia.so
\system\lib\libmediaplayerservice.so
\system\lib\libmedia_jni.so
\system\lib\libnativehelper.so
\system\lib\libnetutils.so
\system\lib\libOmxCore.so
\system\lib\libOmxH264Dec.so
\system\lib\libpixelflinger.so
\system\lib\libpvasf.so
\system\lib\libpvasfreg.so
\system\lib\libpvauthor.so
\system\lib\libpvcommon.so
\system\lib\libpvdownload.so
\system\lib\libpvdownloadreg.so
\system\lib\libpvmp4.so
\system\lib\libpvmp4reg.so
\system\lib\libpvnet_support.so
\system\lib\libpvplayer.so
\system\lib\libpvrtsp.so
\system\lib\libpvrtspreg.so
\system\lib\libqcamera.so
\system\lib\libreference-ril.so
\system\lib\libril.so
\system\lib\librpc.so
\system\lib\libsgl.so
\system\lib\libsonivox.so
\system\lib\libsoundpool.so
\system\lib\libsqlite.so
\system\lib\libssl.so
\system\lib\libstdc++.so
\system\lib\libsurfaceflinger.so
\system\lib\libsystem_server.so
\system\lib\libthread_db.so
\system\lib\libUAPI_jni.so
\system\lib\libui.so
\system\lib\libutils.so
\system\lib\libvorbisidec.so
\system\lib\libwbxml.so
\system\lib\libwbxml_jni.so
\system\lib\libwebcore.so
\system\lib\libwpa_client.so
\system\lib\libxml2wbxml.so
\system\lib\libz.so
\system\lib\modules
\system\lib\modules\wlan.ko



\system\media
铃声音乐文件夹，除了常规的铃声外还有一些系统提示事件音
\system\media\audio
\system\media\audio\alarms 闹铃音
\system\media\audio\notifications 提示音
\system\media\audio\ringtones 铃声
\system\media\audio\ui 界面操作事件音
\system\media\audio\alarms\Alarm_Beep_01.ogg
\system\media\audio\alarms\Alarm_Beep_02.ogg
\system\media\audio\alarms\Alarm_Beep_03.ogg
\system\media\audio\alarms\Alarm_Buzzer.ogg
\system\media\audio\alarms\Alarm_Classic.ogg
\system\media\audio\alarms\Alarm_Rooster_02.ogg
\system\media\audio\notifications\Beat_Box_Android.ogg
\system\media\audio\notifications\CaffeineSnake.ogg
\system\media\audio\notifications\DearDeer.ogg
\system\media\audio\notifications\DontPanic.ogg
\system\media\audio\notifications\F1_MissedCall.ogg
\system\media\audio\notifications\F1_New_MMS.ogg
\system\media\audio\notifications\F1_New_SMS.ogg
\system\media\audio\notifications\Heaven.ogg
\system\media\audio\notifications\Highwire.ogg
\system\media\audio\notifications\KzurbSonar.ogg
\system\media\audio\notifications\OnTheHunt.ogg
\system\media\audio\notifications\TaDa.ogg
\system\media\audio\notifications\Tinkerbell.ogg
\system\media\audio\notifications\Voila.ogg
\system\media\audio\ringtones\BeatPlucker.ogg
\system\media\audio\ringtones\BentleyDubs.ogg
\system\media\audio\ringtones\BirdLoop.ogg
\system\media\audio\ringtones\CaribbeanIce.ogg
\system\media\audio\ringtones\CrazyDream.ogg
\system\media\audio\ringtones\CurveBall.ogg
\system\media\audio\ringtones\DreamTheme.ogg
\system\media\audio\ringtones\EtherShake.ogg
\system\media\audio\ringtones\FriendlyGhost.ogg
\system\media\audio\ringtones\GameOverGuitar.ogg
\system\media\audio\ringtones\Growl.ogg
\system\media\audio\ringtones\InsertCoin.ogg
\system\media\audio\ringtones\LoopyLounge.ogg
\system\media\audio\ringtones\LoveFlute.ogg
\system\media\audio\ringtones\MidEvilJaunt.ogg
\system\media\audio\ringtones\MildlyAlarming.ogg
\system\media\audio\ringtones\NewPlayer.ogg
\system\media\audio\ringtones\Noises1.ogg
\system\media\audio\ringtones\Noises2.ogg
\system\media\audio\ringtones\Noises3.ogg
\system\media\audio\ringtones\OrganDub.ogg
\system\media\audio\ringtones\Ring_Classic_02.ogg
\system\media\audio\ringtones\Ring_Digital_02.ogg
\system\media\audio\ringtones\Ring_Synth_02.ogg
\system\media\audio\ringtones\Ring_Synth_04.ogg
\system\media\audio\ringtones\RomancingTheTone.ogg
\system\media\audio\ringtones\SitarVsSitar.ogg
\system\media\audio\ringtones\SpringyJalopy.ogg
\system\media\audio\ringtones\T-Jingle.ogg
\system\media\audio\ringtones\Terminated.ogg
\system\media\audio\ringtones\TwirlAway.ogg
\system\media\audio\ringtones\VeryAlarmed.ogg
\system\media\audio\ringtones\World.ogg
\system\media\audio\ui\Effect_Tick.ogg



\system\sounds
默认的音乐测试文件，仅有一个test.mid文件，用于播放测试的文件。
\system\sounds\test.mid



\system\usr
用户文件夹，包含共享、键盘布局、时间区域文件等。
\system\usr\keychars
\system\usr\keylayout
\system\usr\share
\system\usr\srec
\system\usr\keychars\qwerty.kcm.bin
\system\usr\keychars\qwerty2.kcm.bin
\system\usr\keychars\trout-keypad-qwertz.kcm.bin
\system\usr\keychars\trout-keypad-v2.kcm.bin
\system\usr\keychars\trout-keypad-v3.kcm.bin
\system\usr\keychars\trout-keypad.kcm.bin
\system\usr\keylayout\h2w_headset.kl
\system\usr\keylayout\qwerty.kl
\system\usr\keylayout\trout-keypad-qwertz.kl
\system\usr\keylayout\trout-keypad-v2.kl
\system\usr\keylayout\trout-keypad-v3.kl
\system\usr\keylayout\trout-keypad.kl
\system\usr\share\bsk
\system\usr\share\zoneinfo
\system\usr\share\bsk\V_FD_speed_101.bsk
\system\usr\share\bsk\V_FD_std_101.bsk
\system\usr\share\zoneinfo\zoneinfo.dat
\system\usr\share\zoneinfo\zoneinfo.idx
\system\usr\srec\config
\system\usr\srec\config\en.us
\system\usr\srec\config\en.us\baseline.par
\system\usr\srec\config\en.us\baseline11k.par
\system\usr\srec\config\en.us\baseline8k.par
\system\usr\srec\config\en.us\dictionary
\system\usr\srec\config\en.us\g2p
\system\usr\srec\config\en.us\grammars
\system\usr\srec\config\en.us\models
\system\usr\srec\config\en.us\dictionary\basic.ok
\system\usr\srec\config\en.us\dictionary\cmu6plus.ok.zip
\system\usr\srec\config\en.us\dictionary\enroll.ok
\system\usr\srec\config\en.us\g2p\en-US-ttp.data
\system\usr\srec\config\en.us\grammars\VoiceDialer.g2g
\system\usr\srec\config\en.us\models\generic.swiarb
\system\usr\srec\config\en.us\models\generic11.lda
\system\usr\srec\config\en.us\models\generic11_f.swimdl
\system\usr\srec\config\en.us\models\generic11_m.swimdl
\system\usr\srec\config\en.us\models\generic8.lda
\system\usr\srec\config\en.us\models\generic8_f.swimdl
\system\usr\srec\config\en.us\models\generic8_m.swimdl

整个Android平台的文件不止是这么多，部分文件在/data文件夹中都是用户文件夹，这里就不介绍了。



/system/framework
這會放 Android 系統的核心程式庫。像是 core.jar, framework-res.apk, com.google.android.gtalkservice.jar,…等等。疑，1.0r1 不是已經將 gtalk 等相關 APIs 移除了嗎？怎麼他的程式庫還在？雖然許多程式庫都是以 jar 結尾的，不過裡面 Java classes 還是以 dex 格式存在著。


/system/app 放的是系統預載的應用程式執行檔。而這裡放的是使用者自己安裝的應用程式執行檔 (*.apk)。/data/data/<app-package-name>
當你在程式中用 Context.openFileOutput() 所建立的檔案，都放在這個目錄下的 files 子目錄內。而用 Context.getSharedPreferences() 所建立的 preferences 檔 (*.xml) ，則是放在 shared_pref 這個子目錄中。/data/location/gps 