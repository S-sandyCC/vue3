#### async/await

1. async 声明返回promise，await阻塞async异步代码块后面的代码
2. await等待promise执行的结果，解决回调地狱
3. const data2 = await fetchData2(） 等同 fetchData2().then((res)=>data2 =res)

#### nextTick()

1. vue的异步更新机制，非立刻更新Dom
2. nextTick()手动刷新Dom，适用在需要在数据更新后立刻操作视图元素
3. : 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM
4. ---->在Dom执行更新之后自动调用nextTic()内的逻辑

#### Object

1. object.keys(obj):将数组的index或键值对数组(json)的key提取转为数组
2. object.extries()：将键值对象转为键值对数组 [ ['foo', 'bar'], ['baz', 42] ]
3. new map(object.extries(obj)):键值对象-->键值对数组-->map

### userRouter  useRoute

1. router.push() -->动态路由
2. route.param  --->路由信息

### Recordable Record 键值对对象

1. Recordable  --->  <string , any>
2. Record ----> <k,v>

##### 复杂表单校验

![image-20250326165141963](C:\Users\NBKJ\Desktop\前端\note\vue3\images\image-20250326165141963.png)

提交前缀

token

cookie:存储在浏览器，由浏览器自动携带

session：浏览器sessionID+服务器session，由浏览器请求自动携带，服务器认证

token：服务器+本地 ，token请求时候放在请求头的Authoration，避开CROS跨域检查

#### 项目代码规范

eslint、commitlint+husky提交规范、CI/CD代码规范、jenkits、ngnix

### 迭代二

1. 商定sku数据表 sku(id,sku_id,sku_code,name,stock,status[上架、下架],create_time,create_by,update_time,update_by,)

   --->sku_color_size(id,sku_is,sku_color,sku_size,create,update)

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

   sale

   order(id,order_id,payment_id,saleLineId(list),status[已支付，未支付，库存不足，退单],create,update)

   saleLineItem(id,saleLineItem_Id,product_id,quantity,totalPrice，status[0，-1，-...]是否缺货)

   !payment(id,payment_id,order_id,strategy_id,amount,discountAmount)

   strategy_id(id,strategy_id,)

   ##### 顺序图----->(currentSale、orderService、skuService)

   1. 开启销售，id
   2. 添加item----->~~根据id查找库存~~---->设置saleLineItem状态
   3. 结束销售 统一检查库存，返回检查状态，计算总金额和数量
   4. (假设有)支付：选择优惠策略
   5. 设置订单状态(未完成/已完成)，流转订单------>支付(不具体实现支付，只返回状态)，挂单
   6. 库存模块对应处理、调度申请(完成订单库存数量立刻更新，订单模块 库存调拨请求走流程)

   ps：订单状态为挂单，所有item的库存数量不改变，标记为负数item对应的商品库存需更新(+)，当订单状态发生改变时候才全部更新(-)库存数量

   ### 思考题

   什么时候发出调度请求？由谁发出调度请求？怎么发出？新的页面？完成调度修改调度请求状态即可提醒订单模块下一步流转

   关于销售模块是否进行库存调度or所有商品库存充足的订单是否走调度申请的问题，如下

   ps：销售模块不处理订单状态，完成下单后所有订单状态修改为未完成，丢给订单模块完成，由订单模块进行调度申请，库存模块完成调度，再改变状态

   2. 销售模块只处理已支付和未完成状态，已支付的订单直接由销售模块通知库存增减，然后再丢给订单模块入账  。其他挂单状态如上丢给订单模块处理。   --------我倾向于这种操作
   3. 加入商品时检查库存or结束销售时候检查库存   ---->2

   #### 订单模块

   ##### 需求

   1. 入账：未--->completed
   2. 缺货补发---->状态改变、库存改变
   3. 退单，状态改变，库存需求？

   ##### 顺序图----操作订单状态

   ###### 入账

   ku

   ###### 缺货补发

   1. 查看订单缺货商品详情
   2. 根据订单号选择需要补充的商品库存
   3. 发送调度申请，等待仓库调度完成(非同步)
   4. 调度请求完成，订单状态流转为[已交付？]
   5. 进行订单入账

   ###### 退单

   1. 修改订单状态

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

   

   1. 用例不正确，主成功场景只要描述下单成功+通知库存减少数量+交付订单模块实现支付，失败场景不需要写太多系统错误or页面错误，写库存不足的挂单情况或者未支付等类似的，可以从订单有什么状态或者库存模块有什么问题方面考虑

   1. 更新product表
   2. 画sku两个表格的ER图
   3. 

   

   

   

   打包部署-->部署到虚拟机-->端口转发-->CICD配置虚拟机ip

   2. 版本、分支   模块-->分支/develop-->测试+报告--->master