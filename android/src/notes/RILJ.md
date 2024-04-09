# RILJ
位于
```
frameworks\opt\telephony\src\java\com\android\internal\telephony
```
RIL.java实现了CommandsInterface接口
## 初始化
- 启动RILSender线程发送数据
- 启动RILReceiver线程接收数据
mCi对象：CommandsInterface（与基带通信（Modem Communication Interface）相关的对象）
## 工作原理（拨号请求为例）
### RILSender主动向RILC发送Request
旧
- 拨号->通过obtainCompleteMessage得到一个Message
- 新建得到并设置RILRequest对象RIL.obtain(RIL_REQUEST_DIAL,messgae)
- RILJ向RILC发送Request请求  标志'>'
- 得到外面传来的Message对象，存在RILRequest.mResult中
新
- RIL调用dial()
- 继续调用  RadioVoiceProxy voiceProxy中的dial()
```java
if (isAidl()) {
            mVoiceProxy.dial(serial, RILUtils.convertToHalDialAidl(address, clirMode, uusInfo));
        } else {
            mRadioProxy.dial(serial, RILUtils.convertToHalDial(address, clirMode, uusInfo));
        }
```
- 以mVoiceProxy为例，其实现类为IRadiovoiceImpl，其中的dial()调用mMockModemConfigInterface.dialVoiceCall(...)
- mMockModemConfigInterface实现类为MockModemConfigBase.java  其中的dialVoiceCall调用 mVoiceService[logicalSlotId].dialVoiceCall(...)
-  mVoiceService[logicalSlotId].dialVoiceCall()调用handler的子类的sendToTarget()进行发送
```
mCallStateHandler
                    .obtainMessage(MSG_REQUEST_DIALING_CALL, newCall.getCallId())
                    .sendToTarget();
```

### RILReceiver接收Response
#### Respnse分类：
- Solicted Response对RILJ发出的Request的回应	
- UnSolicted Response modem主动上报
#### 处理Solicted Response
- 取RILRequest对象
- 处理数据
- 输出标志'<'
- 在GsmCdmaCallTracker的handlerMessage中进行消息处理
#### 处理UnSolicted Response
- 读取号码
- 根据号码找到处理逻辑（可能通过RegistrantList继续上报消息）
- 通知到注册监听Call的人（register）	谁注册谁处理
- 在GsmCdmaCallTracker的handlerMessage中进行消息处理

			
			
			
		
