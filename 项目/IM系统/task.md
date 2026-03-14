任务进程（`task.go`）本质上是一个“后台服务”，它要去连IM网关的Websocket，相当于“敲门进大楼”

IM网关的规则是：
不管是浏览器里的用户，还是这个后台服务，只要进门，都要出示一个JWT Token
给的“REDIS_SYSTEM_ROOT_TOKEN”就是这栋楼里给“运维/系统账号”的出入证

task服务要做的事是：
- 从Kafka的msgChatTransfer中消费消息
- 再通过WebSocket推送给对应在线用户
因为这些动作本身就是”帮别人转发消息“的系统行为，权限大于普通用户，所以需要用”系统账号“的token才能被网关信任

总体来说：
- kafka是”消息中转站“
- task是”快递员“，从Kafka中消费消息
- IM网关是”前台+广播系统“
- WebSocket是”快递员和前台之间的对讲机“
- 系统用户Token是”快递员的身份证“