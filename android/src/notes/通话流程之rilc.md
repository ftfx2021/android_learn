# 通话流程之rilc
## rild
### 分为两个模块

-  RIL Demon 处理整个RIL的event
-  手机厂商自己实现的部分 vendor ril
### 入口
```
hardware/ril/rild/rild.c
```
- 负责初始化事件循环、启动关联Modem的串口监听
- 主要函数：
```
RIL_startEventLoop();
//执行reference-ril.c的RIL_Init()
rilInit =(const RIL_RadioFunctions *(*)(const struct RIL_Env *, int, char **))dlsym(dlHandle, "RIL_Init");
funcs = rilInit(&s_rilEnv, argc, rilArgv);
//注册registerService,建立HIDL通信
 RIL_register(funcs);
 rilc_thread_pool();
```
## rild流程
## main入口
- 开启loop循环
- RIL_Init初始化
- 注册得到reference的回调函数

### RIL_startEventLoop()开启loop循环
#### RIL_startEventLoop()
- 位于hardware/ril/libril/ril.cpp中，创建一个线程并运行eventLoop
```
extern "C" void
RIL_startEventLoop(void) {
    int result = pthread_create(&s_tid_dispatch, &attr, eventLoop, NULL);
}
```
#### eventLoop()
- 进入eventLoop,执行ril_event_loop()
```
static void *eventLoop(void *param) {
    ril_event_loop();
}
```
#### ril_event_loop()
- 继续跟踪ril_event_loop() hardware/ril/libril/ril_event.cpp，循环调用三个方法，进入，分别处理timer_list、watch_table、pending_list
```
void ril_event_loop()
{
    for (;;) {
        // Check for timeouts
        processTimeouts();
        // Check for read-ready
        processReadReadies(&rfds, n);
        // Fire away
        firePending();
    }
}
```

### RIL_Init初始化  
#### RIL_RadioFunctions *RIL_Init()
- hardware/ril/reference-ril/reference-ril.c 开启线程，调用mainLoop
```
const RIL_RadioFunctions *RIL_Init(const struct RIL_Env *env, int argc, char **argv)
{
 
    pthread_attr_init (&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    ret = pthread_create(&s_tid_mainloop, &attr, mainLoop, NULL);
    return &s_callbacks;
}
```
#### mainLoop()
- 进入mainLoop,主要进行开启at通道和超时处理
```
mainLoop(void *param __unused)
{
    AT_DUMP("== ", "entering mainLoop()", -1 );
    at_set_on_reader_closed(onATReaderClosed);
    at_set_on_timeout(onATTimeout);
ret = at_open(fd, onUnsolicited);
RIL_requestTimedCallback(initializeCallback, NULL, &TIMEVAL_0);

    }
```
#### at_open()
- 进入at_open，跟进at通道开启 hardware/ril/reference-ril/atchannel.c  开启线程，调用readerLoop进行处理
```
int at_open(int fd, ATUnsolHandler h)
{
    pthread_attr_init (&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    ret = pthread_create(&s_tid_reader, &attr, readerLoop, &attr);

}
```
#### readerLoop()
- 进入readerLoop()方法  主要调用了s_unsolHandler()方法和processLine()进行消息处理
```
static void *readerLoop(void *arg __unused)
{
    for (;;) {
            if (s_unsolHandler != NULL) {
                s_unsolHandler (line1, line2);
            }
            free(line1);
        } else {
            processLine(line);
        }
    }
}
```
#### at_open()
- 查找s_unsolHandler()方法，发现它是一个ATUnsolHandler s_unsolHandler变量。寻找它的初始化，在at_open()中初始化过，而所赋值的变量是at_open的第二个参数，所以继续寻找调用at_open()的地方，即 hardware/ril/reference-ril/reference-ril.c 的mainLoop()
```
int at_open(int fd, ATUnsolHandler h)
{
    s_unsolHandler = h;
}
```
#### mainLoop()
- 进入mainLoop()，发现传入at_open的第二个参数为onUnsolicited(),说明s_unsolHandler变量最终是由onUnsolicited()进行初始化的
```
mainLoop(void *param __unused)
{
ret = at_open(fd, onUnsolicited);
 }
```
#### onUnsolicited()
- 进入onUnsolicited(),根据不同情况进行了不同处理，但都调用了RIL_onUnsolicitedResponse()函数
```
static void onUnsolicited (const char *s, const char *sms_pdu)
{
   
        RIL_onUnsolicitedResponse(RIL_UNSOL_SIGNAL_STRENGTH,
            response, sizeof(response));
}
```
#### RIL_onUnsolicitedResponse()
- 追溯RIL_onUnsolicitedResponse()，它的实现为：#define RIL_onUnsolicitedResponse(a,b,c) s_rilenv->OnUnsolicitedResponse(a,b,c)，只能追踪s_rilenv初始化的地方，最终发现它是在rild.c的main函数的rilInit初始化的，即funcs = rilInit(&s_rilEnv, argc, rilArgv); s_rilEnv的具体实现又是在ril.cpp中实现，所以进入ril.cpp的RIL_onUnsolicitedResponse()
```
void RIL_onUnsolicitedResponse(int unsolResponse, const void *data,size_t datalen)
{
    if (s_unsolResponses[unsolResponseIndex].responseFunction) {
        ret = s_unsolResponses[unsolResponseIndex].responseFunction(
                (int) soc_id, responseType, 0, RIL_E_SUCCESS, const_cast<void*>(data),
                datalen);
    }
```

#### 
- 跟进s_unsolResponses
```
static UnsolResponseInfo s_unsolResponses[] = {
#include "ril_unsol_commands.h"
};
```
#### ril_unsol_commands.h
- 进入ril_unsol_commands.h 根据不同的消息类型调用不同的方法
```
  {RIL_UNSOL_RESPONSE_RADIO_STATE_CHANGED, radio::radioStateChangedInd, WAKE_PARTIAL},
    {RIL_UNSOL_RESPONSE_CALL_STATE_CHANGED, radio::callStateChangedInd, WAKE_PARTIAL},
    {RIL_UNSOL_RESPONSE_VOICE_NETWORK_STATE_CHANGED, radio::networkStateChangedInd, WAKE_PARTIAL},
    {RIL_UNSOL_RESPONSE_NEW_SMS, radio::newSmsInd, WAKE_PARTIAL},
.....
    {RIL_UNSOL_SIGNAL_STRENGTH, radio::currentSignalStrengthInd, DONT_WAKE},

```
#### callStateChangedInd()
- 以RIL_UNSOL_RESPONSE_CALL_STATE_CHANGED为例，进入callStateChangedInd  hardware/ril/libril/ril_service.cpp,最终调用了callStateChanged()方法
```
int radio::callStateChangedInd(int slotId,
                               int indicationType, int token, RIL_Errno e, void *response,
                               size_t responseLen) {
    if (radioService[slotId] != NULL && radioService[slotId]->mRadioIndication != NULL) {
#if VDBG
        RLOGD("callStateChangedInd");
#endif
        Return<void> retStatus = radioService[slotId]->mRadioIndication->callStateChanged(
                convertIntToRadioIndicationType(indicationType));
        radioService[slotId]->checkReturnStatus(retStatus);
    } else {
        RLOGE("callStateChangedInd: radioService[%d]->mRadioIndication == NULL", slotId);
    }

    return 0;
}
```
#### callStateChanged()
- 跟进callStateChanged()，发现它是IRadioIndication.h中的方法，最终是通过RadioIndication进行HIDL调用
```
public class RadioIndication extends IRadioIndication.Stub
```
#### processLine()
- 回到 hardware/ril/reference-ril/atchannel.c的readerLoop()方法,进入另一个分支,调用handleUnsolicited和handleFinalResponse进行处理
```
static void processLine(const char *line)
{
    pthread_mutex_lock(&s_commandmutex);

    if () {

        handleUnsolicited(line);
    } else if () {
        handleFinalResponse(line);
    } 
}

####  handleUnsolicited()
```
- 进入handleUnsolicited,最终还是调用了s_unsolHandler
```
static void handleUnsolicited(const char *line)
{
    if (s_unsolHandler != NULL) {
        s_unsolHandler(line, NULL);
    }
}
```
#### handleFinalResponse()
- 进入handleFinalResponse(),发信号给s_commandcond线程，使其脱离阻塞状态
```
static void handleFinalResponse(const char *line)
{
    sp_response->finalResponse = strdup(line);
    pthread_cond_signal(&s_commandcond);
}
```
**至此，RIL_Init初始化完成，回到rild.c的main入口**
### 注册得到reference的回调函数
#### RIL_register()
- 进入hardware/ril/libril/ril.cpp RIL_register()  注册服务进行通信
```
extern "C" void
RIL_register (const RIL_RadioFunctions *callbacks) {

    radio::registerService(&s_callbacks, s_commands);

}
```'
#### registerService()
- 进入hardware/ril/libril/ril_service.cpp registerService()
```
void radio::registerService(RIL_RadioFunctions *callbacks, CommandInfo *commands) {

    for (int i = 0; i < simCount; i++) {
        radioService[i] = new RadioImpl;
        radioService[i]->mSlotId = i;
        (void) radioService[i]->registerAsService(serviceNames[i]);
    }
}
```
