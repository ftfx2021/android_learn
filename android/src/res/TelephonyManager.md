
提供Telephony信息查询，修改，phone状态监听等功能
- TelecomManager 主要负责电话呼叫相关的功能，而 TelephonyManager 则主要用于管理手机通信相关的信息和状态

位于
```
frameworkds\base\telephony\java\android\telephony
```
# 四个接口->服务端->服务名：
| 接口                      | 服务               | 服务名            |
|--------------------------|-------------------|-------------------|
| IPhoneSubInfo.aidl   |IPhoneSubInfo.aidl  |  iphonesubinfo   |
| ITelecomService.aidl     | ITelecomService.java    | telecom (TELECOM_SERVICE)   |
| ITelephone.aidl          | ITelephone.java         | phone （TELEPHONY_SERVICE）        |
|    ITelephonyRegistry.aidl   |   ITelephonyRegistry.java    | telephony registry |


## TelecomServiceImpl
 
- 在TelecomLoaderService.java将自己注册到ServiceManager	addService(...)
- 在TelepcomManager(?)中通过ServiceManager得到Telecom Service  getTelecomService()
- TelecomManager得到Telecom Service得到TelephonyManager,获取相关状态实现自己方法

TelepcomManager.java
```
private ITelecomService getTelecomService(){
 ServiceManager.getService(xxx)
}
```
TelecomServiceImpl.java
```
 private TelephonyManager getTelephonyManager(int subId) {
        return ((TelephonyManager) mContext.getSystemService(Context.TELEPHONY_SERVICE))
                .createForSubscriptionId(subId);
    }
```

在TelepcomManager.java  getTelecomService
## iphonesubinfo Service
每个电话订阅对应着一个 SIM 卡，而 IPhoneSubInfo 服务则允许应用程序查询和管理这些订阅信息
- 创建Default Phone 后，在PhoneFactory.java初始化
  + 初始化ProxyController（管理多个数据连接的状态和切换）makeDefaultPhone(...)
  + 初始化PhoneSubInfoController对象  ProxyController.java ProxyController(...)
  + 注册PhoneSubInfoController到ServiceManager PhoneSubInfoController.java(新版通过TelephonyServiceManager.java的 class ServiceRegisterer的register方法注册，而register方法里使用ServiceManager.addService(xxx)注册 
- 在TelephonyManager通过ServiceManager得到 iphonesubinfo Service ->得到software version,deviceID等信息(得到这些信息具体实现由IsimRecords、Phone完成)
```java
static IPhoneSubInfo getSubscriberInfoService()
```
## PhoneInterfanceManager——TELEPHONY_SERVICE 
负责封装和提供电话功能相关的接口，以便让应用程序能够方便地与设备的电话功能进行交互
- TelephonyService依赖TELEPHONY_SERVICE实现大部分方法
- Default Phone 后，PhoneInterfanceManager在PhoneGlobals.java onCreate()初始化
- 构造方法
  + 得到一些关键类,phone,app,mCM（CallManager）...
- ServiceManager.addService(...)
- PhoneInterfanceManager的方法 最终由Phone实现

## telephony.registry Service

负责管理电话相关的注册信息和通知，以及向其他应用程序提供电话状态的更新

- 在SystemServer.java将teleRegistry实例注册到ServiceManager
