
# Telecom
## 功能
- MT(收),MO(发)消息处理关键入口，Dialer和TeleService的消息中转站
- 呈上：发call状态变化消息给IInCallService,并收到Dialer发出的call状态变化消息
- 启下:把Dialer消息继续发给IConnectionService，接收TeleService发出的call消息并传给Dialer
### 获取TelecomManager对象
来电，拨号核心流程
```
   public static TelecomManager from(Context context) {
        return (TelecomManager) context.getSystemService(Context.TELECOM_SERVICE);
    }
```
### 获取Service服务的Binder对象
```
private ITelecomService getTelecomService() {
        if (mTelecomServiceOverride != null) {
            return mTelecomServiceOverride;
        }
        if (sTelecomService == null) {
            ITelecomService temp = ITelecomService.Stub.asInterface(
                    ServiceManager.getService(Context.TELECOM_SERVICE));
...
        return sTelecomService;
 }
```
在 Android 系统启动过程中，SystemServer加载时将启动ITelecomService系统服务
