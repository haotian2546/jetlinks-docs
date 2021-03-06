# 从消息网关中订阅消息

消息网关作为jetlinks进程内部的消息总线,负责各种消息的转发.使用发布订阅模式,使用topic来划分不同的消息类型.
可根据topic订阅并处理消息,如:

```java

//订阅所有设备上线消息
@Subscribe("/device/*/*/online")
public Mono<Void> handleDeviceOnline(DeviceOnlineMessage message){
    //处理消息
    return doSomeThing();
}

```

## 设备消息

所有设备消息的`topic`的前缀均为: `/device/{productId}/{deviceId}`.
如:设备`device-1`上线消息: `/device/product-1/device-1/online`.
可通过通配符订阅所有设备的指定消息,如:`/device/*/*/online`,
或者订阅所有消息:`/device/**`.

::: tip
使用通配符订阅可能将收到大量的消息,请保证消息的处理速度,否则会影响系统消息吞吐量.
:::

设备Topic列表

::: warning
列表中的topic已省略前缀`/device/{productId}/{deviceId}`,使用时请加上.
:::

|  topic   | 类型  | 说明 |
|  ----  | ----  | ----|
| /online  | DeviceOnlineMessage | 设备上线   |
| /offline  | DeviceOfflineMessage |  设备离线  |
| /message/event/{eventId}  | DeviceEventMessage |  设备事件  |
| /message/property/report  | ReportPropertyMessage |  设备上报属性  |
| /message/property/read/reply  | ReadPropertyMessageReply |  读取属性回复  |
| /message/property/write/reply  | WritePropertyMessageReply |  修改属性回复  |
| /message/function/reply  | FunctionInvokeMessageReply |  调用功能回复  |
| /message/property/report  | ReportPropertyMessage |  设备上报属性  |
| /register  | DeviceRegisterMessage |  设备注册,通常与子设备消息配合使用  |
| /unregister  | DeviceUnRegisterMessage |  设备注销,同上  |
| /message/children/{childrenDeviceId}/{topic}  | ChildDeviceMessage |  子设备消息,{topic}为子设备消息对应的topic  |
| /message/children/reply/{childrenDeviceId}/{topic}  | ChildDeviceMessage |  子设备回复消息,同上  |

## 设备告警

在配置了设备告警规则,设备发生告警时,会发送消息到消息网关.

```js
`/rule-engine/device/alarm/{productId}/{deviceId}/{ruleId}`

{
  "productId":"",
  "alarmId":"",
  "alarmName":"",
  "deviceId":"",
  "deviceName":"",
  "...":"其他从规则中获取到到信息"
}
```

## 规则引擎

规则引擎中产生的日志,事件等也会发送到消息网关中.

### 规则日志

```js
`/rule-engine/{instanceId}/{nodeId}/log`
{
    "instanceId":"规则实例ID",
    "nodeId":"节点ID",
    "level":"日志级别",
    "message":"日志内容",
    "timestamp":"事件戳",
    "context":{} //上下文信息
}
```

### 规则事件

```js

`/rule-engine/{instanceId}/{nodeId}/event/{eventType}`

{
    "event":"事件类型",//
    "instanceId":"实例ID",
    "nodeId":"节点ID",
    "ruleData":{
        "data":"本节点产生的数据",
        "attributes":{}//其他拓展属性
    }//规则数据
}

```

事件类型:

|  event   | 名称  | 说明 |
|  ----  | ----  | ----|
| node_started  | 节点启动事件 | 启动规则时触发   |
| node_execute_before  | 节点准备执行 |  执行节点之前触发  |
| node_execute_result  | 节点产生数据 |    |
| node_execute_done  | 节点执行完成 |    |
| node_execute_fail  | 执行失败 |    |

## 日志

### 系统日志

topic格式: `/logging/system/{logger名称,.替换为/}/{level}`.

```js
`/logging/system/org/jetlinks/pro/TestService/{level}`

{
"name":"org.jetlinks.pro.TestService", //logger名称
"threadName":"线程名称",
"level":"日志级别",
"className":"产生日志的类名",
"methodName":"产生日志的方法名",
"lineNumber"：32,//行号
"message":"日志内容",
"exceptionStack":"异常栈信息",
"createTime":"日志时间",
"context":{} //上下文信息
}

```