Registrants类似观察者，一对多，观察状态
## 使用：Class xxxRegistrants
### 创建RegistrantsList
- 封装好增加/删除Registrant方法
- 增加add(...)  addUnique(..)（不重复）	   
- 删除remove()  如registerForxxx,unregisterForxxx
### 注册监听某消息
- xxxRegistrants.registerForxxx()
- 发送通知，通知所有注册到 xxxRegistrants中的Registrant
- xxxRegistrants.notifyResult(...)
### RegistrantsList内部的方法:
- notifyResult(obj res)(公共方法，执行internalNotifyRegistrants（）
- internalNotifyRegistrants(...)通知Registrant列表，执行internalNotifyRegistrant()通知单个Registrant对象：handler.sendMessage(msg)
	
	
