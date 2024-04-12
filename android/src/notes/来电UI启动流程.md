
- 从CallIntentProcessor起，进入其processIncomingCallIntent()方法
```
 @Override
        public void processIncomingCallIntent(CallsManager callsManager, Intent intent) {
            CallIntentProcessor.processIncomingCallIntent(callsManager, intent);
        }
```
- 继续进入CallIntentProcessor.processIncomingCallIntent的方法，调用了CallsManager的processIncomingCallIntent方法
```
static void processIncomingCallIntent(CallsManager callsManager, Intent intent) {
        callsManager.processIncomingCallIntent(phoneAccountHandle, clientExtras);
    }
```
- 进入CallsManager的processIncomingCallIntent方法，最终调用了addCall方法
```
public Call processIncomingCallIntent(PhoneAccountHandle phoneAccountHandle, Bundle extras,
        boolean isConference) {
addCall(call);
}
```
- 进入addCall,添加了对call的监听事件，onCallAdd在CallsManagerListenerBase()定义

```
 @VisibleForTesting
    public void addCall(Call call) {
        for (CallsManagerListener listener : mListeners) {
            listener.onCallAdded(call);
    }
```
- 找到它的重写类，InCallController.java的onCallAdded方法，绑定服务，将call添加到服务中
```
    @Override
    public void onCallAdded(Call call) {
            bindToServices(call);
            inCallService.addCall(sanitizeParcelableCallForService(info, parcelableCall));
    }
```
- 进入InCallService.java的addCall方法，获取了一个类型为MSG_ADD_CALL的消息对象并发送
```
  @Override
        public void addCall(ParcelableCall call) {
            mHandler.obtainMessage(MSG_ADD_CALL, call).sendToTarget();
        }
```
- 根据MSG_ADD_CALL找到处理消息的方法：执行Phone的internalAddCall(),最后执行了Phone的fireCallAdded()
```
 final void internalAddCall(ParcelableCall parcelableCall) {
            fireCallAdded(call);
}

```
- 进入fireCallAdded(),执行了监听的OnCallAdded方法
```
   private void fireCallAdded(Call call) {
        for (Listener listener : mListeners) {
            listener.onCallAdded(this, call);
        }
    }

```
- 寻找onCallAdded方法，被InCallService()重写
```
  @Override
        public void onCallAdded(Phone phone, Call call) {
            InCallService.this.onCallAdded(call);
        }
```
- 在InCallService的实现类中找到onCallAdded，可见最终调用了InCallPresenter的onCallAdded方法，该类通常是在应用程序中展示来电界面、去电界面以及通话界面的地方。
```
  public void onCallAdded(Call call) {
67      Trace.beginSection("InCallServiceImpl.onCallAdded");
68      InCallPresenter.getInstance().onCallAdded(call);
69      Trace.endSection();
70    }
```
- 进入InCallPresenter的onCallAdded方法，执行callList.onCallAdded方法，在callList中添加call

```
 public void onCallAdded(final android.telecom.Call call) {

803          callList.onCallAdded(context, call, latencyReport);

813    }
```
- 在CallList.java中找到onCallAdded,向所有Listeners发送了通知
```
public void onCallAdded(){
      notifyGenericListeners();
}
```
- 进入notifyGenericListeners
```
private void notifyGenericListeners() {
    Trace.beginSection("CallList.notifyGenericListeners");
     for (Listener listener : listeners) {
      listener.onCallListChange(this);
 }
     Trace.endSection();
  }
```
- 找到实现监听的对象，即实现了onCallListChange()方法的类，位于InCallPresenter类中，执行startOrFinishUi启动ui
```
@Override
  public void onCallListChange(CallList callList) {
    newState = startOrFinishUi(newState);
  }
```
- 进入startOrFinishUi()方法，执行showInCall()显示来电界面
```
   private InCallState startOrFinishUi(InCallState newState) {
 if ((showCallUi || showAccountPicker) && !shouldStartInBubbleMode()) {
       LogUtil.i("InCallPresenter.startOrFinishUi", "Start in call UI");
       showInCall(false /* showDialpad */, !showAccountPicker /* newOutgoingCall */);
   }
}
```
- 进入showCallUi(),启动activity
```
  public void showInCall(boolean showDialpad, boolean newOutgoingCall) {
    LogUtil.i("InCallPresenter.showInCall", "Showing InCallActivity");
    context.startActivity(
    InCallActivity.getIntent(context, showDialpad, newOutgoingCall, false /* forFullScreen */));
   }
```
