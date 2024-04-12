Registrants类似观察者，一对多，观察状态
， Registantlist 为通知者， Registrant为观察者 Registrantlist 
作为通知者 负责对通知者的增加（ add/addUnique 删除（ remove ），并且能够发出通知
( notifyRegistrants ；而 Registrant 作为观察者，由其｜「1ternalNotifyRegist ants 方法负责响应通知者
发出的 notifyRegistrants 通
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
	
	
