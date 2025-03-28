商定sku数据表 sku(id,sku_id,sku_code,name,stock,status[上架、下架],create_time,create_by,update_time,update_by,)

--->sku_color_size(id,sku_code,sku_color,sku_size,create,update)

selete * from sku_color_size cz left join sku s on cz.sku_id = s.sku_id;

2. 商定权限 admin、cashier、财务、经理、库存管理人员（order由谁管理）
3. product的sku_id改为sku_code
4. 销售模块和库存模块的业务联系

- **销售模块** 依赖于 **库存模块** 提供实时库存数据，以确保销售订单的准确性。
- **库存模块** 根据 **销售模块** 的销售数据动态更新库存数量。

下单时候，由于需要实时获取库存数量，1. 选择商品（color-size唯一确定一个sku_code）数量时请求获取库存数量，库存充足下达订单结算(已支付)同时更新库存数据，库存不足(设定订单状态为库存不足)实现挂单功能

   库存(增减)<-----请求----------销售sale------------->订单(挂单+支付)   下单选择商品的动作完成后其他行为都归结库存或者订单模块

2. 销售模块需要获取库存？  product left join on sku_code =

#### 销售模块

##### 功能需求

1. 下单 2. 促销 3、调度申请

##### 表设计

销售表:销售项=1：n  销售项:商品=1:n(数量，非商品)

sale(id,saleId,[list] orderId) y一个sale--success、error  ）

order(id,order_id,payment_id,saleLineId(list),status[已支付，未支付，库存不足，退单],create,update)

saleLineItem(id,saleLineItem_Id,product_id,quantity,totalPrice，status[0，-1，-...]是否缺货)

!payment(id,payment_id,order_id,strategy_id,amount,discountAmount，status[已支付。未支付])

strategy_id(id,strategy_id,)

##### 关系

* sale-1----n-order-1----1-payment--1---1--strategy-

* ​                  order-1----n-item

##### 顺序图----->(currentSale、orderService、skuService)

1. 开启销售，id   currentSale (redis）-->status-->orderDetail--->生成id---**返回id**
2. 获取item id----->根据id查找商品、库存、返回库存-->**返回库存量**

currentSlle---2List Propertity(acturally 2 order)-----**参数类型SaleItem**

1. 结束销售 计算总金额和数量-----successList/errList---遍历list---创建item--liistAddItem---cerrentSale.addList
   * successList---status--sku(-)--成功--payment--id---存入db--
   * errList---status--
2. (假设有)支付：选择优惠策略
3. 设置订单状态(未完成/已完成/未支付)，提交订单------>支付(不具体实现支付，只返回状态)，挂单
4. 库存模块对应处理、调度申请(完成订单库存数量立刻更新，库存调拨请求走流程)

ps：订单状态为挂单，所有item的库存数量不改变，标记为负数item对应的商品库存需更新(+)，当订单状态发生改变时候才全部更新(-)库存数量

### 系统顺序图

1. 开启销售 ---saleId、salestatus
2. 查看商品库存 （productId）
3. 结束销售--->创建saleLineItem、currentSale.2list<order> [successList,errorList]
       * --->生成成功order ---->successList.setid---->setSaleLineItemList()---->计算数量和价格---库存调度-->未支付
      *  --->生成挂单order ---->errorList.setid---->setSaleLineItemList()----->计算数量和价格------->挂单
4. 支付订单

* 创建paymentId---->payment设置价格、总数、支付方式----->设置支付状态----()>successList.setPayment()
* errorList.setPayment()
* 成功订单+成功

### 思考题

什么时候发出调度请求？由谁发出调度请求？怎么发出？新的页面？

关于销售模块是否进行库存调度or所有商品库存充足的订单是否走调度申请的问题，如下

ps：销售模块不处理订单状态，完成下单后所有订单状态修改为未完成，丢给订单模块完成，由订单模块进行调度申请，库存模块完成调度，再改变状态

2. 销售模块只处理已支付和未完成状态，已支付的订单直接由销售模块通知库存增减，然后再丢给订单模块入账  。其他挂单状态如上丢给订单模块处理。   --------我倾向于这种操作

#### 订单模块

##### 需求

1. 入账：未--->completed
2. 缺货补发---->状态改变、库存改变
3. 退单，状态改变，库存需求？

##### 顺序图----操作订单状态

###### 入账

ku

#### 库存模块

##### 需求

1. 库存调拨(+)：处理调度请求+实现库存调度
2. 处理发货(-)  

##### 数据表  调拨申请(扩展：消息队列)

transfer_requestes(id,t_id.sku_id,request_amount,status(未完成、已完成、已取消))

实现---->添加库存

#### 财务模块

##### 需求

1. 审计财务(订单收入)
2. 审计库存流转(调拨)

