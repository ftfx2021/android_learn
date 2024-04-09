# TelephonyRegistry
主要功能：上报消息
两种方法：
notifyxxx
发送broadcast
## notifyxxx
- 检测权限等参数
- 遍历mRecords，取出对应Record对象（监听者的Record对象）
- 配置Record回调函数
```
  public void notifyCellInfoForSubscriber(int subId, List<CellInfo> cellInfo) {
        if (!checkNotifyPermission("...")) {return;}
        if (VDBG) {...}
        if (cellInfo == null) {...}
        int phoneId = getPhoneIdFromSubId(subId);
        synchronized (mRecords) {
            if (validatePhoneId(phoneId)) {
                mCellInfo.set(phoneId, cellInfo);
                for (Record r : mRecords) {
                    if (...) {
                        try {
                            if (DBG_LOC) { log(...);}
                            r.callback.onCellInfoChanged(cellInfo);
                        } catch (RemoteException ex) {
                            mRemoveList.add(r.binder);
                        }
                    }
                }
            }
            handleRemoveListLocked();
        }
//部分还有发送广播方法
    }
```
mRecords由 本类的 listen(...)维护

注册监听和发通知流程

- 根据需要监听事件，重写IPhoneStateListener接口中的方法（监听事件都在定义该接口中）
  + r.callback.onCellInfoChanged(cellInfo);
- 调用TelephonyManager listen()->里面调用 telephonyRegistry.listenFromListener(...) listenFromListener(...)调用TelephonyRegistry中的 listenWithEventList方法，再调用listen方法在里面处理Record
- 处理IPhoneStateListener接口中的方法
- Call状态变化消息上报流程：
  + RILJ - CallTracker - Phone - DefaultPhoneNotifier - TelephonyRegistry -监听者
