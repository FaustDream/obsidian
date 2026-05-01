# Engine对象、事件触发顺序

# Engine对象、事件触发顺序

##### Engine对象

| 

分类
 

对象管理
 

说明
|
| 

业务对象&查询
 

AppPackageManager
 

模块管理，获取引擎中已安装应用的相关信息
|
| 

 

BitObjectManager
 

业务对象管理，对引擎中模块数据进行新增、删除、修改等
|
| 

 

BitObjectTrackManager
 

数据跟踪管理
|
| 

 

ViewManager
 

对象视图管理
|
| 

 

Query
 

查询
|
| 

 

IndexManager
 

索引管理
|
| 

流程相关
 

WorkflowTemplateManager
 

流程模板管理
|
| 

 

WorkflowInstanceManager
 

流程实例管理
|
| 

 

WorkItemManager
 

流程工作项管理
|
| 

 

UrgencyManager
 

修办
|
| 

 

AgencyManager
 

委托
|
| 

组织单元
 

Organization
 

组织单元管理，获取组织中的角色、用户等
|
| 

权限
 

FunctionActManager
 

功能权限管理，获取角色权限的相关设置
|
| 

日志
 

LogWriter
 

日志管理
|
| 

消息
 

Notifier
 

消息通知，可用于发送消息
|
| 

基础配置
 

EngineConfig
 

引擎基础配置
|
| 

系统集成
 

BitBus
 

业务集成
|
| 

其他设置
 

DataDictManager
 

数据字典，对数据字典的新增、修改等
|
| 

 

SettingManager
 

其他设置
|
| 

任务提醒
 

TaskManager
 

定时服务，用于定时触发执行任务、提醒等
|
| 

SNS
 

SNSManager
 

SNS管理，处理动态、信息分享等
|
| 

其他
 

AppMarketManager
 

应用市场管理
|
##### 事件触发顺序

![](5f848962afcddefb-20250407113344-ytkehtk.png)
