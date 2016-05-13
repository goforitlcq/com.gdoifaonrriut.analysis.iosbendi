##MediaFilterCache
1. 根据广告类型和媒体的评级和是否为网赚读取对应的**分成**和**扣量**比设置进入map
2. 将自己media自身中的数据和filters_level中的数据进行比较，只要不相同直接将数据设置进入**DATA_MEDIA**的redis中，  
**point**这儿更新完之后会直接更新mediaApp中对象的值


##MeidaPriceCache
1. 根据appid-adid将分成价格price存入map中
2. 同时存入DATA_MEDIA_PRICE的redis中

## MediaCache ##
1. 通过media，media-app，media-channel联表查询 新建继承自media的mediaApp和mediaChannel对象并放入监视器map中

## AdsCache ##
1. 通过ads,ad_plan的联表查询 新建app对象并放入监视器中
2. 同时更新redis中存取的相关类型的值
	1. 设置控量处理


## ShowProcessor ##
1. 先求出媒体新增独立请求数，作为**invalid**假量字段
2. 如果广告id字段不为空，那么将**invalid**假量置为0
2. 计算**save**变量
4. 放入内存等待1分钟入库
4. **平滑控量**（针对mac）
5. **展示频次控制**（针对udid）

## ClickProcessor ##
1. 获取控量处理，如果剩余数为0，直接返回，不进行如下统计
2. 分别对从media和ad两个方面考虑isStop的取值
3. **控量**（没有区分平滑和快速）
4. **点击频次控制**
5. 获取今天是否被点击过（查询(按照时间，类型和标识查的)callBack表获取**历史纪录**的CallbackItem对象），如果是放入内存，等待统计
6. 按照的vals和media以及ad 重新初始化item
7. 进行防作弊计算invalid
8. 如果是以点击计费，计算出income和cost，并放入缓存**DATA-CLICK-INFO**
9. 计算**save**变量
10. 插入callback数据库和redis的value数据库
11. 添加进入内存的操作
12. 特殊回调模式，所以增加点击记录

## JumpProcessor ##
1. 获取控量处理，如果剩余数为0，直接返回，不进行如下统计
2. **控量**（没有区分平滑和快速）
3. 获取独立点击（针对今天的），如果为1，而且不存在30天的的历史纪录，计算invalid，save，更新callback数据库和redis的value数据库，添加进入内存的操作
4. 添加进入内存的操作
5. 特殊回调模式，所以增加点击记录

## ActiveProcessor ##
1. 获取30天的记录，如果记录为空（不存在点击，已经激活）直接返回
2. 如果callback中点击，跳转和激活都不存在，直接返回
3. 广告下线后，我们只接收广告主三天内的激活，给媒体下发只发1天内的激活
4. 多次激活对应的价格读取
5. 判断item.action广告是否已经激活
	1. 已经激活
		1. 进行下层判断并作激活处理
		2. 返回
	2. 没有激活
		1. 判断历史是否激活isActive
			1. 已经激活，直接返回
6. ip假量
	1. 针对同一个ip同一adid和同一action在当天不得超过指定值，超过指定为假量-->invalid
	2. redis中的值 **IP_20160513|action-adid-ip|num** 针对num自增1--》ipCount
	3. 如果应用媒体不是测试状态，那么判断ipCount>ads.ipNum-->invalid |= 1
7. 时间假量
	1. 计算line中的时间和callBack中的打开时间之差-->interval
	2. 和广告中的字段进行比较，如果超过设置范围-->invalid |=2
8. 下面开始计算invalid子字段
9. 乐抢评分浮动金额  
	1. 
10. 判断是进程激活
	1. 判断广告是否计入报表
		1. 是，无操作
		2. 否，将收入和消费清空
	2. unique = 1
11. 放入内存进行统计
12. 更新item中的数据将其更新到callback表中，如果是今天的数据直接update，如果不是，需要add
13. 将激活信息存入redis当中addActive，对应前面的isActive函数
14. 同步激活时间，用于控制深度任务的显示
15. 媒体所有广告前10不扣量，渠道针对单一广告前50不扣量
16. 下发请求对接sendScore将请求的url以及其他信息等放入**ACTION-HTTP-REQUEST**
17. 快速精准控量快速任务完成之后删除		
18. 快速任务实时上报IDFA给广告主，将对应的http请求和数据信息放入**REPORT-IDFA-TOCP**
## isStop ##

## invalid ##


## save ##
1. ShowProcessor
	1. 根据媒体和广告求出是否要进行**cpa随机扣量**
	2. 独立展次不为0，save为false，不是停止状态 将扣量设置为1
2. ClickProcessor
 	1. 媒体所有广告前10不扣量，渠道针对单一广告前50不扣量
 
	 