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
> 在安卓源码中，ril_event_loop函数是用于处理RIL（Radio Interface Layer）事件的核心部分。它负责处理来自Java层的请求和事件。并在接收到请求后调用相应的处理函数
- 继续跟踪ril_event_loop() hardware/ril/libril/ril_event.cpp，通过select()函数监听来自Socket的请求，监听到则调用下面的对应方法进行处理，（处理timer_list、watch_table、pending_list）
```
void ril_event_loop()
{
  
    for (;;) {
  n = select(nfds, &rfds, NULL, NULL, ptv);
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
#### rild.c main入口
```
 dlsym()函数：加载动态链接库，dlsym函数接受两个参数，一个是动态链接库的句柄（dlHandle），另一个是要查找的符号的名称（"RIL_Init"）。dlsym返回一个指向该符号的指针，这个指针可以被用来调用该符号所代表的函数。
```
```
 (const RIL_RadioFunctions *(*)(const struct RIL_Env *, int, char **))，这是一个函数指针类型的声明。它表示一个指向函数的指针。*(*)(const struct RIL_Env *, int, char **)表示这个函数参数是一个函数指针。
```
```
rilInit =
        (const RIL_RadioFunctions *(*)(const struct RIL_Env *, int, char **))
        dlsym(dlHandle, "RIL_Init");
funcs = rilInit(&s_rilEnv, argc, rilArgv);
```
- 所以这两句就相当于funcs  = RIL_Init(&s_rilEnv, argc, rilArgv)
- 于是去寻找一个名为RIL_Init()的函数
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
> mainLoop()和ril_event_loop()是两个线程，并行执行的
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
- 程进入at_open，跟进at通道开启 hardware/ril/reference-ril/atchannel.c  开启一个新线程，调用readerLoop进行处理
```
int at_open(int fd, ATUnsolHandler h)
{
    pthread_attr_init (&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    ret = pthread_create(&s_tid_reader, &attr, readerLoop, &attr);

}
```
#### readerLoop()
> readerLoop()函数负责从AT通道读取一行数据，并在超时时返回NULL。
> 先后读取line1，line2，二者通常是成对的，表示一个完整的unsolicited SMS（modem主动上报）
- 进入readerLoop()方法  主要调用了s_unsolHandler()方法和processLine()进行消息处理
```
static void *readerLoop(void *arg __unused)
{
    for (;;) {
        const char * line;
        line = readline();
        if (line == NULL) { break; }
        if(isSMSUnsolicited(line)) {
            char *line1;
            const char *line2;
            line1 = strdup(line);//复制line
            line2 = readline();
            if (line2 == NULL) {free(line1); break;}
            if (s_unsolHandler != NULL) {s_unsolHandler (line1, line2);}
            free(line1);
        } else {
            processLine(line);
        }
    }
    onReaderClosed();
    return NULL;
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
- 即s_unsolHandler (line1, line2)相当于onUnsolicited(line1, line2)
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
#### rild.c的s_rilEnv定义
- 追溯RIL_onUnsolicitedResponse()，它的实现为：#define RIL_onUnsolicitedResponse(a,b,c) s_rilenv->OnUnsolicitedResponse(a,b,c)，追踪s_rilenv初始化，首先是在ril_reference的RIL_Init()初始化的，s_rilenv = env，而env是该函数的第一个参数，RIL_Init()被rild.c的main函数调用; s_rilEnv在rild.c中定义
> 在Android的RIL（Radio Interface Layer）架构中，static struct RIL_Env s_rilEnv定义了一个结构体，该结构体包含了RIL需要与Java层交互的函数指针。这些函数指针用于处理来自Java层的请求和响应，以及与Modem的通信
```
static struct RIL_Env s_rilEnv = {
    RIL_onRequestComplete,
    RIL_onUnsolicitedResponse,
    RIL_requestTimedCallback,
    RIL_onRequestAck
};
```
#### RIL_onUnsolicitedResponse()
- s_rilEnv里面函数指针的**具体实现又是在ril.cpp中实现**，所以进入ril.cpp的RIL_onUnsolicitedResponse().根据unsolResponseIndex查找对应的s_unsolResponse，并调用对应的函数
```
void RIL_onUnsolicitedResponse(int unsolResponse, const void *data,size_t datalen)
{
    if (s_unsolResponses[unsolResponseIndex].responseFunction) {
        ret = s_unsolResponses[unsolResponseIndex].responseFunction(
                (int) soc_id, responseType, 0, RIL_E_SUCCESS, const_cast<void*>(data),
                datalen);
    }
```

#### s_unsolResponses
- 跟进s_unsolResponses,
```
static UnsolResponseInfo s_unsolResponses[] = {
#include "ril_unsol_commands.h"
};
```
#### ril_unsol_commands.h
- 进入ril_unsol_commands.h 根据不同的消息类型调用不同的函数
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
sp<RadioImpl> radioService[SIM_COUNT];//每张卡都有一个RadioImpl实例
int radio::callStateChangedInd(int slotId,int indicationType, int token, RIL_Errno e, void *response,size_t responseLen) {
    if (radioService[slotId] != NULL && radioService[slotId]->mRadioIndication != NULL) {
        Return<void> retStatus = radioService[slotId]->mRadioIndication->callStateChanged(convertIntToRadioIndicationType(indicationType));
        radioService[slotId]->checkReturnStatus(retStatus);
    }
}
```
#### callStateChanged()
- 跟进callStateChanged()，发现它是IRadioIndication.h中的方法，最终是通过RadioIndication进行HIDL调用，然后就算java层的RIL处理
```
public class RadioIndication extends IRadioIndication.Stub
```
#### processLine()
- 回到 hardware/ril/reference-ril/atchannel.c的readerLoop()方法,进入另一个分支,即处理不满足IsSmsUnsolicited的消息(可能是RIL请求Modem后得到的响应，也可能是其他类型的响应)，调用handleUnsolicited和handleFinalResponse进行处理
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
```
#### handleUnsolicited()
- 进入handleUnsolicited,最终还是调用了s_unsolHandler处理unsolicited消息
```
static void handleUnsolicited(const char *line)
{
    if (s_unsolHandler != NULL) {
        s_unsolHandler(line, NULL);
    }
}
```
#### handleFinalResponse()
- 进入handleFinalResponse(),发信号给s_commandcond线程，使其脱离阻塞状态。RIL等待Modem的响应，当响应到达时，通知RIL可以继续处理响应。
```
static void handleFinalResponse(const char *line)
{
    sp_response->finalResponse = strdup(line);
    pthread_cond_signal(&s_commandcond);
}
```
**至此，RIL_Init初始化完成，回到rild.c的main入口，注册来自ril-reference的RIL_Init()函数**
```
 funcs = rilInit(&s_rilEnv, argc, rilArgv);
    RIL_register(funcs);
```
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
- 进入hardware/ril/libril/ril_service.cpp registerService()  与java层交互

```
struct RadioImpl : public V1_1::IRadio {
    int32_t mSlotId;
    sp<IRadioResponse> mRadioResponse;
    sp<IRadioIndication> mRadioIndication;
    sp<V1_1::IRadioResponse> mRadioResponseV1_1;
    sp<V1_1::IRadioIndication> mRadioIndicationV1_1;
    ...
void radio::registerService(RIL_RadioFunctions *callbacks, CommandInfo *commands) {
    for (int i = 0; i < simCount; i++) {
        radioService[i] = new RadioImpl;
        radioService[i]->mSlotId = i;
        (void) radioService[i]->registerAsService(serviceNames[i]);
    }
}
```
