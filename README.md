# cross - 美诺福业务管理系统

## 基础资料

* 厂商 - 维保仪器设备的生产厂商
  - 编号
  - 名称

* 客户
  - 编号
  - 名称
  - 作业区
    - 作业区名称
  
* 设备类型 - 维保仪器设备的特定分类
  
* 设备 - 维保服务所涉及的仪器设备的类型及其规格定义
  
  - 名称 - 
  - 型号 - 设备厂商所给定的型号编码
  - 厂商 - 生产厂商，缺省为美诺福
  - 类型 - 设备类型
  - 编号 - 可选，任何便于标识的统一编码
  - 标签 
  - 图片
  - 备注
  
* 维保设备 - 由客户指定的需进行维保的仪器设备
  - 所属公司（客户）
  - 设备
  - 设备序列号
  - 作业区
  - 安装地点
  - 状态 - 草稿/启用/停用
  - 负责人
  - 权重（重要性）
  - 标签 
  - 图片
  - 备注
  
* 设备检修标准

* 巡检
  - 期间 - 巡检开始和完成时间
    - 开始时间
    - 完成时间
  - 设备 - 本次巡检的设备
  - 检测明细
    - 检测部位
    - 检测值
    - 是否通过检测
  - 检测人
  - 审核 ？
  
## 角色

* 维修技术部经理
* 维修站长
* 维修工程师

## 需求

### 系统管理员

系统管理员用于分配各子系统主管权限（子系统其它业务角色均由该子系统业务主管指定），而并不涉及具体的业务操作。其中，缺省系统管理员为系统预设的系统管理员，用于系统实际启用后指定首位系统管理员，一旦有系统管理员被指定，该缺省系统管理员将自动失效，系统将保持至少一位系统管理员。


 - 指定首位维修技术部经理

系统初始状态下有一个缺省系统管理员，用于指定一个用户为首位维修技术部经理。一旦指定维修技术部经理：

* 缺省系统管理员账号失效；
* 维修技术部经理可以指定其他用户为维修技术部经理；
* 维修技术部经理可以收回其他（不包括自己）维修技术部经理权限；


### 用户 - 新用户注册

新用户在被许可进入系统前需自行注册账号，内容主要包括：

* 用户账号 - 必需且唯一，角色登录标识
* 姓名
* 密码 - 6位以上数字和字母组合，不能是纯数字
* 手机
* 头像
* 邮箱

新用户完成注册后即可登录系统

### 维修技术部经理 - 新建维保技术服务站

随着公司业务的发展，新设维保技术服务站时，维修技术部经理向系统提供新建维保技术服务站的基本信息，内容主要包括：

名称 - 维保技术服务站名称
编码 - 可选，唯一
服务公司 - 服务公司名称
Logo

建立维保技术服务站后，维修技术部经理即可指定一新注册用户为该维保技术服务站长。

### 用户 - 新用户注册

新用户在被许可进入系统前需自行注册，内容主要包括：

* 用户名 - 必需且唯一，用户账号及登录标识
* 姓名
* 密码 - 6位以上数字和字母组合，不能是纯数字
* 手机
* 头像
* 邮箱

新用户完成注册后即可登录系统






## 问题
* 在巡检时如果某个检测部位未达检测标准，这时检测人员作什么操作？
* 点巡检记录审核是何含义？ 是否需要专门审核？有审核不通过的情况？
* 安装地点同作业区有何区别？区分作业区和安装地点有何作用？
* 多个甲方单位拥有同种设备的情况是否普遍？同种设备应具有相同的巡检和点检标准是吗？

## 系统安装
假设安装目录为: /home/jsmtest/apps

Clone cross：

```
git clone -b docker-deploy-test https://github.com/JSMetta/cross.git
cd cross
git pull origin dev

git clone -b vcross-1.0.1 https://github.com/JSMetta/VCross.git
cd VCross
git pull origin vcross-1.0.1

cd ..

docker-compose up --build -d

```
cross项目在/home/jsmtest/apps/cross中


## 常用命令

cd /home/jsmtest/apps/cross
git pull origin docker-deploy-test
docker build -t jsmetta/cross .
docker run -d --name redis -p 6379:6379 redis

docker run --name mongodb --restart unless-stopped -v /home/mongo/data:/data/db -v home/mongo/backups:/backups -d mongo --smallfiles

docker run -d -p 8089:8080 --link redis:redis --name cross jsmetta/cross

docker run -it --link mongodb:mongo --rm mongo mongo --host mongo test

docker run -d --name nginx -p 80:80 --link cross:cross cross/nginx

docker-compose up --build

docker exec -it mongodb mongo

#### Remove dangling images
docker images -f dangling=true

docker images purge

#### 备份数据库

脚本文件：/home/mongo/dailybackup.sh

shudo crontab -e

## 开发文档

处理消息时，如果返回：
* Promise.resolve(true) - 接收消息
* Promise.resolve(false) - 拒绝消息，重新进入消息列表
* Promise.reject(err) - 拒绝消息，消息将被废弃

### REST服务

#### PoTransactions - 采购单交易集合

采购单交易集合资源提供采购单交易查询和交易执行服务

##### 采购单交易查询服务

采购单交易查询服务实现为标准Finelets REST查询服务。
* 参数
  * id -- 采购单标识
* 返回 -- 采购单交易资源（PoTransaction）数据集合。

##### 执行采购单业务交易
执行采购单业务交易实现为标准Finelets Create REST服务。
* 参数
  * id -- Uri param，采购单标识
  * type -- Uri query，业务交易类型，有效交易类型包括：
    * commit -- 提交审批
    * review -- 审核
    * inv -- 入库
* 交易数据（request body）
  * __v -- 采购单版本号
  * actor -- 当前业务交易执行者标识，例如，如果当前交易为入库，则actor为库管人员
  * date -- 交易日期
  * data -- 交易相关信息
  * remark -- 交易备注
* 返回结果
  * 201 -- 交易成功，并返回采购入库交易资源
  * 400 -- 交易参数错误，如单号不存在、无交易数据等
  * 500 -- 交易失败
   
###### 采购入库
采购入库是对处于“执行”状态的采购单相关采购到货进行入库，其结果包括：
* 记录采购入库单
* 更新库存量 -- 库存量 = 当前库存量 + 入库量
* 更新在单量 -- 在单量 = 当前在单量 - 入库量

采购入库的交易相关信息包括：
* qty -- 入库数量，required
* loc -- 库位
* refNo -- 参考单号
  
业务规则包括：

* 交易数据合法
  * 由id指定的采购单必须存在
  * 当前采购单版本号必须与交易数据中__v给出的版本号一致
  * 当前采购单必须处于“执行/open”状态
  * 必须指定入库交易者，且入库交易者必须存在
  * 如未指定入库日期，则取数据库当前日期
  * 必须指定非0的入库数量，入库数量可以为负， 以应用于入库数量对冲 

采购入库交易返回采购入库单资源：
* type -- ‘inv’，为采购单交易类型
* id -- 采购单交易标识
* parent -- 采购单标识
* actor -- 交易者标识
* date -- 交易日期
* data -- 入库信息
  * qty -- 数量
  * loc -- 库位
  * refNo -- 参考单号
* remark -- 交易备注
  

## 业务规则

### 采购单业务交易



### 采购交易导入
采购交易可以通过CSV文件导入，格式为：
* 首行为字段名称
* 字段
  * 交易编号
  * 品名
  * 料品类型 - 低值易耗品/资产/料品（采购/委外）
  * 规格
  * 单位
  * 供应商类型 - 实体店/电商/厂家
  * 供应商名称
  * 供应商链接
  * 采购周期
  * 采购单价
  * 数量
  * 金额
  * 申请人
  * 申请日期
  * 审核人
  * 审核日期
  * 采购日期
  * 采购人
  * 到货日期
  * 领用人
  * 领用日期
  * 领用项目
  * 领用数量
  * 货位

```
交易编号,料品类型,品名,规格,单位,数量,采购单价,金额,供应商名称,供应商类型,参考单号,供应商链接,采购周期,申请人,申请日期,审核人,审核日期,采购日期,采购人,到货日期,领用人,领用日期,领用数量,领用项目,货位,备注

transNo,partType,partName,spec,unit,qty,price,amount,supplier,supply,refNo,supplyLink,purPeriod,applier,appDate,reviewer,reviewDate,purDate,purchaser,invDate,user,useDate,useQty,project,invLoc,remark

const expected = {
						transNo: 'xulei00001',
						partType: '物料',
						partName: 'JSM-A1实验用格子布',
						spec: 'abcd',
						unit: '米',
						qty: 150,
						price: 8800,
						amount: 8800,
						supplier: '绍兴惟楚纺织品有限公司',
						supply: '厂商',
						refNo: 'JSMCONV20181109A',
						supplyLink: '开票中',
						purPeriod: 80,
						applier: '徐存辉',
						appDate: new Date('2018/11/9').toJSON(),
						reviewer: '徐存辉',
						reviewDate: new Date('2018/11/9').toJSON(),
						purchaser: '徐存辉',
						purDate: new Date('2018/11/9').toJSON(),
						invDate: new Date('2018/12/12').toJSON(),
						user: '测试组',
						useDate: new Date('2018/12/12').toJSON(),
						useQty: 100,
						project: '测试组',
						invLoc: ' h234',
						remark: 'remark'
					}

```

## MQ
Consumers receive messages from a particular queue in one of two ways:
* By subscribing to it via the basic.consume AMQP command. This will place the channel being used into a receive mode until unsubscribed from the queue.
* Requesting a single message from the queue is done by using the basic.get AMQP command. This will cause the consumer to receive the next message in the queue and then not receive further messages until the next basic.get. You shouldn’t use basic.get in a loop as an alternative to
basic.consume, because it’s much more intensive on Rabbit.

When a Rabbit queue has multiple consumers, messages received by the queue are served in a round-robin fashion to the consumers.

Every message that’s received by a consumer is required to be acknowledged. Either the consumer must explicitly send an acknowledgement to RabbitMQ using the basic.ack AMQP command,
or it can set the auto_ack parameter to true when it subscribes to the queue.

If a consumer receives a message and then disconnects from Rabbit (or unsubscribes
from the queue) before acknowledging, RabbitMQ will consider the message
undelivered and redeliver it to the next subscribed consumer.

Both consumers and producers can create queues by using the queue.declare
AMQP command. But consumers can’t declare a queue while subscribed to another
one on the same channel.

Here are some other useful properties you can set for the queue:
* exclusive—When set to true, your queue becomes private and can only be
consumed by your app. This is useful when you need to limit a queue to only
one consumer.
* auto-delete—The queue is automatically deleted when the last consumer
unsubscribes. If you need a temporary queue used only by one consumer, combine
auto-delete with exclusive. When the consumer disconnects, the queue
will be removed.

With passive set to true, queue.declare will return successfully if the queue exists, and return an error without
creating the queue if it doesn’t exist.

A queue is said to be bound to an exchange by a routing key.

There are four kinds of exchanger:
* direct - if the routing key matches, then the message is delivered to the corresponding queue.
  * 利用缺省Exchange, 通过: `$channel->basic_publish($msg, '', 'queue-name');`可以向特定的队列发送消息
  * 可以实现将向多个队列发送消息
* fanout - when you send a message to a fanout
exchange, it’ll be delivered to all the queues attached to this exchange.
* topic - 可以实现将多种消息发送给多个队列
* headers
