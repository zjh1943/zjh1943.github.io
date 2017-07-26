Status: public
description: COLOR公寓后台系统，数据库设计
title: COLOR公寓数据库设计
tags: mysql,db,rdb,color


# 数据库文档

>oc为opencart的简称，所有数据库表格均以oc_开头。


## oc_room
`room`代表一个单间. 每个`room`必须属于某个`house`。

|      列名     |     类型     |  取值范围 |          代表含义         | 其他 |
| ------------- | ------------ | --------- | ------------------------- | ---- |
| id            | int          |           | id                        |      |
| house_id      | int          |           | 属于哪个house             |      |
| name          | varchar      |           | 房间名称                  |      |
| detail        | text         |           | 见下注释                  |      |
| price         | float        |           | 房间价格，元/月           |      |
| area          | int          |           | 面积                      |      |
| orientation   | tinyint      | 0,1,2,3,4 | 未编辑，东西南北          |      |
| style_id      | int          |           | PK of oc_room_style       |      |
| slogan        | text         |           | slogan                    |      |
| weight        | int          |           | 影响排序规则，允许负值    |      |
| state         | tinyint      | 0~5       | 见下注释                  |      |
| butler_id     | int          |           | 服务此房间的管家          |      |
| number        | tinyint u    | [0~max)   | 在大房间内的序号          |      |
| has_lav       | tinyint      |           | 有独立卫生间              |      |
| has_balcony   | tinyint      |           | 有私人阳台                |      |
| hidden        | tinyint      |           | 强制隐藏此房间            |      |
| update_at     | timestamp    |           | 上次更改                  |      |
| ceate_at      | timestamp    |           | 创建日期                  |      |
| complete_at   | date         |           | 预计上线时间              |      |
| privilidge    | bigint u     |           | 权限禁止管理              |      |
| second_weight | int          |           | 智能排序规则,每天定时更新 |      |
| room_type     | varchar(256) |           | 房间类型                  |      |

privilidge每个位的含义：

* 0: 禁止邀请其他人
* 1: 禁止被邀请

state:  

* 0：初始状态
* 9：预上线
* 10：已上线
* 11：已预订
* 12：已租出

detail:  
编辑员对房间的描述。示例如下："\[\]"括起来的是标题，剩下的是内容。


>[主卧带卫生间，超大阳台，超值空间]  
周边：人大附中朝阳学校芍药居分校：包括人大附小，初中，高中部。另被 对外经贸大学、北京服装学院、现代文学馆所环绕，周边医院有著名的中日 友好医院！银行，超市等配套设施齐全，毗邻元大都遗址公园。  
交通：您想每天睡到自然醒么，距离地铁芍药居站步行（地铁10号线和13 号线交汇）只需2分钟，公交119路; 547路; 942快; 专22路步行只需5分 钟，多么舒适多么快捷！快拿起电话拨号吧！  
价格：正规主卧，朝南向，带阳台，距离地铁站仅需2分钟的超便利位置， 超高性价比。

* room_type: varchar定义规则: 小区名字+房间序号+卧室个数+大厅个数+价格  
(拼接的字符串相同则认为同一类房源)


## oc_house
`house`代表一套房子，比如一套三室一厅。`house`必须属于某个`district`.

|         列名        |     类型    | 取值范围 |      代表含义      | 其他 |
| ------------------- | ----------- | -------- | ------------------ | ---- |
| id                  | int         |          | id                 |      |
| name                | varchar(20) |          | 名称               |      |
| floor               | u smallint  |          | 楼层               |      |
| building_id         | int         |          | PK of oc_buiding   |      |
| area                | u smallint  |          | 面积               |      |
| room_count          | u tinyint   |          | 卧室个数           |      |
| holl_count          | u tinyint   |          | 厅个数             |      |
| lav_count           | u tinyint   |          | 卫生间个数         |      |
| total_floor         | u smallint  |          | 总楼层             |      |
| building_name       | varchar(10) |          | 大楼名称           |      |
| community_id        | int         |          | PK of oc_community |      |
| position_id         | int         |          | PK of oc_position  |      |
| building_postion_id | int         |          | PK of oc_position  |      |
| state               | tinyint u   |          | 状态               |      |
| elec_key            | varchar(16) |          | 电费合同账号       |      |
| gas_key             | varchar(16) |          | 燃气费客户编码     |      |
| butler_id           | int         |          | 服务此房间的管家   |      |

state:(装修状态)  

* 0：初始状态
* 1：已签约
* 2：已测量
* 3：已装修完毕
* 4：采购订单已发出
* 5：家具设备已签收和验货
* 6：家具已安装
* 7：已清洁
* 8：已拍摄
* 9：已完成



## oc_building

`building`代表一栋楼。一个`building`对应一个唯一的地理位置（经纬度）。

|     列名     |     类型    | 取值范围 |      代表含义     | 其他 |
|--------------|-------------|----------|-------------------|------|
| id           | int         |          | id                |      |
| name         | varchar(20) |          | 名称              |      |
| community_id | int         |          | 所属小区          |      |
| position_id  | int         |          | PK of oc_position |      |



## oc_community
代表一个小区，一个小区一有**唯一**的街道号码。若XX小区分为3期，每期的街道号不同，则视为三个不同小区。

|      列名     |     类型    | 取值范围 |       代表含义       | 其他 |
| ------------- | ----------- | -------- | -------------------- | ---- |
| id            | int         |          | id                   |      |
| name          | varchar(40) |          |                      |      |
| position_id   | int         |          | PK of oc_position    |      |
| position_desc | varchar(40) |          | 地理位置的自定义描述 |      |
| boundary_id   | int         |          | oc_boundary.id       |      |



## oc_position


|      列名     |      类型      | 取值范围 |  代表含义  | 其他 |
| ------------- | -------------- | -------- | ---------- | ---- |
| id            | int            |          | id         |      |
| province      | varchar(8)     |          | 省份       |      |
| city          | varchar(8)     |          | 城市       |      |
| district      | varchar(8)     |          | 区         |      |
| street        | varchar(16)    |          | 街道名     |      |
| street_number | int(12)        |          | 街道号码   |      |
| longtitude    | decimal(13,10) |          | 经度       |      |
| latitude      | decimal(13,10) |          | 纬度       |      |
| custom_desc   | text           |          | 自定义描述 |      |



## oc_renter_bill

代表一个实际发生的订单，租客与迈芒的订单。

|       列名      |     类型     | 取值范围 |      代表含义      | 其他 |
| --------------- | ------------ | -------- | ------------------ | ---- |
| id              | int          |          | id                 |      |
| room_id         | int          |          | room id            |      |
| start_time      | date         |          | 合同开始时间       |      |
| expect_end_time | date         |          | 合同结束时间       |      |
| real_end_time   | date         |          | 实际结束合同的时间 |      |
| price           | decimal(9,6) |          | 标价，元/月        |      |
| real_price      | decimal(9,6) |          | 实际价格，元/月    |      |
| deposit         | int unsigned |          | 押金金额           |      |
| pay_cycle_id    | int          |          |                    |      |
| user_id         | int          |          | oc_user.id         |      |
| state           | int          | 0,1,2,3  | 见注释             |      |
| month_paid      | tinyint      |          | 已付清月数         |      |
| service_fee     | int unsigned |          | 服务费，元/月      |      |
| reduce          | int          |          | 减少金额           |      |
| reduce_reason   | varchar(32)  |          | 减少金额原因       |      |


state:

* 0:未生效
* 1:已生效
* 2:已取消
* 3:已完成

## oc_renter_bill_payment(废弃)

租金支付

|    列名    |   类型  | 取值范围 |      代表含义     | 其他 |
|------------|---------|----------|-------------------|------|
| id         | int     |          | id                |      |
| bill_id    | int     |          | oc_renter_bill.id |      |
| month      | tinyint |          | 月数              |      |
| for_month  | tinyint |          | 付了几个月的钱    |      |
| type       | int     |          | 见下注释          |      |
| count      | int     |          | 金额              |      |
| coupon_id  | int     |          | oc_coupon.id      |      |
| payment_id | int     |          | oc_payment.id     |      |

type:

* 1:支付押金（count为负数时表示退换押金）
* 2:支付房租
* 4:支付服务费
* 8:用优惠券抵扣房租。
* 6: 支付房租和服务费。


## oc_landlord_bill_payment

房东租金支付

|   列名  |   类型  | 取值范围 |       代表含义      | 其他 |
|---------|---------|----------|---------------------|------|
| id      | int     |          | id                  |      |
| bill_id | int     |          | oc_landlord_bill.id |      |
| month   | tinyint |          | 月数                |      |
| type    | int     |          | 见下注释            |      |
| count   | int     |          | 金额                |      |

type:

* 0:支付押金（count为负数时表示退换押金）
* 1:支付房租


 

## oc_landlord_bill

代表一个实际发生的订单，房主与迈芒的订单。

|       列名      |     类型     | 取值范围 |      代表含义      | 其他 |
|-----------------|--------------|----------|--------------------|------|
| id              | int          |          | id                 |      |
| house_id        | int          |          |                    |      |
| start_time      | date         |          | 合同开始时间       |      |
| expect_end_time | date         |          | 合同结束时间       |      |
| real_end_time   | date         |          | 实际结束合同的时间 |      |
| price           | decimal(9,6) |          | 标价，元/月        |      |
| pay_cycle_id    | int          |          | PK of oc_pay_cycle |      |
| user_id         | int          |          | 客户 id            |      |
| deposit         | int unsigned |          | 押金金额           |      |
| state           | int          | 0,1,2,3  | 见注释             |      |
| property_fee    | float        |          | 物业费(元/月)      |      |
| agency_fee      | float        |          | 中介费             |      |
| agency_fee      | float        |          | 中介费             |      |


备注：  

* 正常情况`expect_end_time=real_end_time`,当用户提前终止合约的时候，两个时间不等。


state:

* 0:未生效
* 1:已生效
* 2:已取消
* 3:已完成


## oc_job
客户职业

|      列名     |     类型    |   取值范围   | 代表含义 | 其他 |
| ------------- | ----------- | ------------ | -------- | ---- |
| id            | int         |              | id       |      |
| name          | varchar(20) |              | 职业名称 |      |
| detail        | text        |              | 职业描述 |      |
| parent_job_id | int         | PK of oc_job | 所属职业 |      |
| ver_job       | int         |              | 使用版本 |      |

ver_job:

* 0:初始化 （老版本使用）
* 1:2015-11-30整理job分类后使用版本 (新版本)


## oc_user

客户账号。

|         列名         |     类型     |  取值范围  |      代表含义     | 其他 |
| -------------------- | ------------ | ---------- | ----------------- | ---- |
| id                   | int          |            | id                |      |
| email                | varchar(50)  |            |                   |      |
| mobile               | varchar(20)  |            |                   |      |
| full_name            | varchar(32)  |            |                   |      |
| icon                 | text         |            | 头像地址          |      |
| identify_card_img    | text         |            | 身份证照片        |      |
| identify_card_number | varchar(20)  |            | 身份证号码        |      |
| user_type            | tinyint      | 0,1,2      | 租客,房主,都是    |      |
| verified             | tinyint      | 0,1        | 未实名认证,已认证 |      |
| gender               | tinyint      | 0,1        | 女，男            |      |
| birthday             | date         |            | 生日              |      |
| job_id               | int          | PK(oc_job) | 职业              |      |
| pwd_hash             | varchar(255) |            |                   |      |
| pwd_reset_token      | varchar(255) |            |                   |      |
| auth_key             | varchar(32)  |            |                   |      |
| status               | tinyint      | 10,0       | 激活,未激活       |      |
| create_at            | timestamp    |            |                   |      |
| update_at            | timestamp    |            |                   |      |
| bank_card            | varchar(24)  |            | 银行卡            |      |
| bank_name            | varchar(20)  |            | 银行名称          |      |
| bank_of_deposit      | varchar(20)  |            | 开户行            |      |
| hobby                | varchar(30)  |            | 爱好              |      |
| state                | tinyint      |            | below             |      |
| invite_code          | char(6)      |            | 邀请码            |      |
| invite_by_user_id    | int          |            | 邀请人            |      |
| invitee_awarded      | tinyint      |            | 邀请人已奖励      |      |
| say                  | varchar(256) |            | 用户say           |      |

detail:

* state: 默认/系统生成（0），用户激活（1）



## oc_keyword
代表一个关键字。e.g. 南北通透、朝南、落地窗、大床、安静、核心商圈

|    列名    |     类型     | 取值范围 |    代表含义    | 其他 |
| ---------- | ------------ | -------- | -------------- | ---- |
| id         | int          |          | id             |      |
| name       | varchar(10)  | 10 chars | 名称           |      |
| display    | int          | 0-1      | 是否显示       |      |
| key_type   | int          |          | 关键字类型     |      |
| key_desc   | text         |          | 关键字描述信息 |      |
| icon_url   | varchar(256) |          | icon的oss地址  |      |
| serial_num | int          |          | 排序           |      |

* display: 房屋筛选里不展示(0), 展示(1)
* key_type: 未分配(0,老版本使用), 特色配置(1), 其它配置(2)
* serial_num: 值小的在最前面

## oc_keyword_of_room
`room`和`keyword`的映射关系。每一行表示某个`room`被添加了某个`keyword`。

|      列名      | 类型 | 取值范围 |         代表含义         | 其他 |
| -------------- | ---- | -------- | ------------------------ | ---- |
| id             | int  |          | id                       |      |
| room_id        | int  |          | 房间id                   |      |
| keyword_id     | int  |          | 关键字                   |      |
| display_detail | int  |          | 是否显示在详情页         |      |
| display_widget | int  |          | 是否显示在点击查看小窗口 |      |

* display_detail: 不展示(0), 展示(1)
* display_widget: 不展示(0), 展示(1)

## oc_keyword_of_house

## oc_keyword_of_community
结构跟`oc_keyword_of_room`类似




## oc_room_style

|  列名  |     类型    | 取值范围 | 代表含义 | 其他 |
|--------|-------------|----------|----------|------|
| id     | int         |          | id       |      |
| name   | varchar(10) |          | 风格名称 |      |
| detail | text        |          | 风格描述 |      |


## oc_butler

管家。

|         列名         |     类型    | 取值范围 |  代表含义  | 其他 |
|----------------------|-------------|----------|------------|------|
| id                   | int         |          | id         |      |
| full_name            | varchar(10) |          |            |      |
| nick_name            | varchar(10) |          | 昵称       |      |
| identify_card_img    | text        |          | 身份证照片 |      |
| identify_card_number | varchar(20) |          | 身份证号码 |      |
| pwd                  | varchar(32) |          |            |      |
| mobile               | varchar(20) |          |            |      |
| tel                  | varchar(20) |          | 分机电话   |      |
| custom_desc          | text        |          | 自定义描述 |      |
| gender               | tinyint     | 0,1      | 女，男     |      |
| icon                 | text        |          | 头像地址   |      |


## oc_bulter_and_room

|    列名    | 类型 | 取值范围 | 代表含义 | 其他 |
|------------|------|----------|----------|------|
| id         | int  |          | id       |      |
| bulter_id  | int  |          |          |      |
| room_id    | int  |          |          |      |



## oc_image_of_room

|    列名   |   类型  | 取值范围 |   代表含义  | 其他 |
| --------- | ------- | -------- | ----------- | ---- |
| id        | int     |          | id          |      |
| room_id   | int     |          |             |      |
| img       | text    |          |             |      |
| type      | tinyint | 0~5      | 见下定义    |      |
| room_desc | text    |          | 图片描述    |      |

`type`:

* 0：未编辑
* 1：高清大图
* 2：列表封面
* 3：三维
* 4：户型图(无效,户型图和house绑定,参照oc_house_type)
* 5：普通
* 6：竖图


## oc_image_of_house

|       列名      |   类型  | 取值范围 |      代表含义      | 其他 |     |
| --------------- | ------- | -------- | ------------------ | ---- | --- |
| id              | int     |          | id                 |      |     |
| house_id        | int     |          | oc_house.id        |      |     |
| img             | text    |          |                    |      |     |
| house_desc      | text    |          | 公用图片描述       |      |     |
| floor           | int     |          | 复式房第几层       |      |     |
| house_layout_id | int     |          | oc_house_layout.id |      |     |
| type            | tinyint | 0~5      | 见下定义           |      |     |

`floor`: 默认为１，当type为４户型图时，需要配置


## oc_image_of_community

|     列名     |   类型  | 取值范围 |     代表含义    | 其他 |
| ------------ | ------- | -------- | --------------- | ---- |
| id           | int     |          | id              |      |
| community_id | int     |          | oc_community.id |      |
| img          | text    |          |                 |      |
| type         | tinyint | 0~5      | 见下定义        |      |
| comm_desc    | text    |          | 小区图片描述    |      |



## oc_image_of_room_style
（表格结构跟`oc_image_of_room`基本一样，在此略）

## oc_district

|     列名    |    类型    | 取值范围 |    代表含义    | 其他 |     |
| ----------- | ---------- | -------- | -------------- | ---- | --- |
| id          | int        |          | id             |      |     |
| name        | varchar(8) |          | 名称           |      |     |
| city        | varchar(8) |          | 城市           |      |     |
| enable      | boolean    | 0-1      | 是否网站上显示 |      |     |
| weight      | int        |          | 列表显示顺序   |      |     |
| boundary_id | int        |          | oc_boundary.id |      |     |

`enable`:

* 0：不显示
* 1：显示

## oc_circle

商圈。

|     列名    |     类型    | 取值范围 |    代表含义    | 其他 |
| ----------- | ----------- | -------- | -------------- | ---- |
| id          | int         |          | id             |      |
| name        | varchar(20) |          |                |      |
| district_id | int         |          | oc_district.id |      |
| enable      | boolean     | 0-1      | 是否网站上显示 |      |
| boundary_id | int         |          | oc_boundary.id |      |

`enable`:

* 0：不显示
* 1：显示

## oc_circle_of_community

小区所属商圈。

|     列名     | 类型 | 取值范围 | 代表含义 | 其他 |
|--------------|------|----------|----------|------|
| id           | int  |          | id       |      |
| circle_id    | int  |          |          |      |
| community_id | int  |          |          |      |


## oc_subway_line

|      列名     |     类型    | 取值范围 |  代表含义  | 其他 |
|---------------|-------------|----------|------------|------|
| id            | int         |          | id         |      |
| name          | varchar(10) |          | 名称       |      |
| number        | tinyint     |          | 线路号     |      |
| station_count | tinyint     |          | 总共站数量 |      |


## oc_subway_station

|     列名    |     类型    | 取值范围 |    代表含义    | 其他 |
| ----------- | ----------- | -------- | -------------- | ---- |
| id          | int         |          | id             |      |
| name        | varchar(10) |          | 名称           |      |
| position_id | int         |          |                |      |
| enable      | boolean     | 0-1      | 是否网站上显示 |      |
| boundary_id | int         |          | oc_boundary.id |      |

## oc_subway_station_of_line

|        列名       | 类型 | 取值范围 |    代表含义    | 其他 |
|-------------------|------|----------|----------------|------|
| id                | int  |          | id             |      |
| subway_line_id    | int  |          | 线路id         |      |
| subway_station_id | int  |          |                |      |
| sequence          | int  | 1~N      | 是线路的第几站 |      |


## oc_subway_line_of_community
|      列名      | 类型 | 取值范围 | 代表含义 | 其他 |
|----------------|------|----------|----------|------|
| id             | int  |          | id       |      |
| subway_line_id | int  |          |          |      |
| community_id   | int  |          |          |      |


## oc_subway_station_of_community
|        列名       | 类型 | 取值范围 | 代表含义 | 其他 |
|-------------------|------|----------|----------|------|
| id                | int  |          | id       |      |
| subway_station_id | int  |          |          |      |
| community_id      | int  |          |          |      |


## oc_price_range

选购房屋时，列出的价格区间选项。

| 列名 | 类型 | 取值范围 | 代表含义 | 其他 |
|------|------|----------|----------|------|
| id   | int  |          | id       |      |
| low  | int  |          | 最低价格 |      |
| high | int  |          | 最高价格 |      |


## oc_pay_cycle

付款周期

|   列名   |   类型  |  取值范围 |    代表含义    | 其他 |
|----------|---------|-----------|----------------|------|
| id       | int     |           | id             |      |
| discount | float   | 0~1       | 折扣数         |      |
| cycle    | tinyint | 3,6,12,24 | 多少月付款一次 |      |


## oc_mobile_validate

待验证的短信验证码

|    列名   |     类型    | 取值范围 |    代表含义    | 其他 |
|-----------|-------------|----------|----------------|------|
| id        | int         |          | id             |      |
| mobile    | varchar(16) |          | 手机号         |      |
| code      | varchar(4)  |          | 验证码         |      |
| create_at | timestamp   |          | 创建于XX时刻   |      |
| ip        | varchar(16) |          | IP             |      |
| type      | tinyint     |          | 解释见下       |      |
| used      | tinyint     |          | 是否已经使用了 |      |

type:

0: 预定房间
1: 注册账号
3：找回密码


## oc_schedule_bill

客户预定订单

|    列名   |     类型    | 取值范围 | 代表含义 | 其他 |
|-----------|-------------|----------|----------|------|
| id        | int         |          | id       |      |
| mobile    | varchar(16) |          | 手机号   |      |
| name      | varchar(6)  |          | 姓名     |      |
| job       | varchar(10) |          | 职业     |      |
| room_id   | int         |          | oc_room  |      |
| create_at | timestamp   |          |          |      |



## oc_bill_subscriber

订单订阅者

|      列名     |     类型    | 取值范围 |    代表含义    | 其他 |
|---------------|-------------|----------|----------------|------|
| id            | int         |          | id             |      |
| name          | varchar(6)  |          |                |      |
| mobile        | varchar(16) |          |                |      |
| email         | varchar(40) |          |                |      |
| status        | tinyint     | 0,1      | 0:激活，1:屏蔽 |      |
| trigger_event | tinyint     |          |                |      |

* trigger_event：预定房间（0），保洁（1），维修（2）。


## oc_cash_flow

公司现金流量表。

|    列名   |       类型       | 取值范围 |          代表含义          | 其他 |
|-----------|------------------|----------|----------------------------|------|
| id        | int              |          | id                         |      |
| bill_type | tinyint unsigned |          |                            |      |
| bill_id   | int              |          | 订单号                     |      |
| money     | int              |          | 实际收入金额，负数表示支出 |      |
| create_at | timestamp        |          |                            |      |
| comment   | text             |          | 备注                       |      |

解释：

* bill_type: 租入(0),租出(1),家具(2),装修(3)

## oc_furniture

家具定义表

|   列名  |       类型       | 取值范围 |       代表含义       | 其他 |
|---------|------------------|----------|----------------------|------|
| id      | int              |          | id                   |      |
| name    | varchar(40)      |          | 一款物品有唯一的名称 |      |
| type    | tinyint unsigned |          | 家具类型             |      |
| comment | text             |          | 备注             |      |

type:

* 0：未知
* 1：床
* 2：沙发
* 3：书桌
* 4：衣柜
* 5：电视
* 6：空调
* 7：床垫
* 8：床套
* 9：床
* 10：写字台
* 11：护眼台灯
* 12：挂画
* 13：写字台椅子
* 14：餐桌
* 15：多功能衣架
* 16：隔板+支架
* 17：小米盒子
* 18：360路由器
* 19：窗帘
* 20：锁
* 21：冰箱
* 22：洗衣机
* 23：宽带


## oc_furniture_bill

家具订单

|     列名     |       类型       | 取值范围 |   代表含义   | 其他 |
|--------------|------------------|----------|--------------|------|
| id           | int              |          | id           |      |
| furniture_id | int              |          |              |      |
| money        | int              |          | 须支付总金额 |      |
| count        | int unsigned     |          | 采购的数量   |      |
| status       | tinyint unsigned | 0,1,2    | 见备注       |      |
| suplier      | varchar(30)      |          | 供货商名称   |      |
| comment      | text             |          | 备注         |      |
| create_at    | timestamp        |          |              |      |
| update_at    | timestamp        |          |              |      |

status: 

* 0：未提交，编辑中。默认值。
* 1：已提交给供应商，还未声场完毕。
* 2：已生产完毕，已入库。


## oc_furniture_deploy

家具配置表。house/room与家具之间的分配关系。

|     列名     |   类型  | 取值范围 |          代表含义          | 其他 |
|--------------|---------|----------|----------------------------|------|
| id           | int     |          | id                         |      |
| furniture_id | int     |          | 家具编号                   |      |
| house_id     | int     |          |                            |      |
| room_id      | int     |          | 如果为空，表示是套房的家具 |      |
| count        | tinyint |          | 数量，默认为1              |      |

## oc_decorate_bill

装修订单。

|     列名    |     类型     | 取值范围 |   代表含义   | 其他 |
|-------------|--------------|----------|--------------|------|
| id          | int          |          | id           |      |
| house_id    | int          |          |              |      |
| company     | varcahr(30)  |          | 装修公司名称 |      |
| money       | int unsigned |          | 金额         |      |
| comment     | text         |          | 备注         |      |
| create_at   | timestamp    |          | 下单时间     |      |
| complete_at | timestamp    |          | 完成时间     |      |


## oc_coupon    

优惠券

|    列名    |    类型   | 取值范围 |     代表含义     | 其他 |
|------------|-----------|----------|------------------|------|
| id         | int       |          | id               |      |
| code       | char(6)   |          | 名称             |      |
| use_method | tinyint   |          | 见下解释         |      |
| get_from   | tinyint   |          | 见下解释         |      |
| state      | tinyint   | 0,1,2    |                  |      |
| validity   | timestamp |          | 到期日期         |      |
| cash       | smallint  |          | 抵扣金额         |      |
| stackable  | tinyint   |          | 可叠加使用       |      |
| create_at  | timestamp |          |                  |      |
| update_at  | timestamp |          |                  |      |
| user_id    | int       |          | 已被用户领取     |      |
| locked     | tinyint   |          | 已锁定（支付中） |      |

注解：

use_method:

* 0: 支付房租均可使用
* 1: 支付任何费用均可使用，包括O2O服务。

state：init(0), active(1), bind(2), used(3)。  
validity： 有效期，天数。最多365天。

from:

* 0: default
* 1：首月半价
* 2：社交活动
* 3：推荐其它好友 
* 4：暂时废弃
* 5：自己被邀请

## oc_third_payment

第三方支付

|     列名     |     类型     | 取值范围 |       代表含义      | 其他 |
|--------------|--------------|----------|---------------------|------|
| id           | int          |          | id                  |      |
| trade_no     | char(32)     |          | 交易号              |      |
| method       | tinyint      |          | below               |      |
| type         | tinyint      |          | below               |      |
| total_fee    | float        |          | 付款金额            |      |
| state        | tinyint      |          | below               |      |
| ext          | varchar(128) |          | json 附加订单信息   |      |
| third_result | varchar(128) |          | json 第三方返回信息 |      |
| create_at    | timestamp    |          |                     |      |
| update_at    | timestamp    |          |                     |      |


* method: 支付宝(0)，微信(1)，银联(2)。
* type: 未知(0)，交房租(1)，预定房间（2），水电气(3)。
* state: 已创建(0)，已成功(1)，已失败(2)。


## oc_coupon_seed

优惠码，生成种子记录

| 列名 |  类型 | 取值范围 |         代表含义        | 其他 |
|------|-------|----------|-------------------------|------|
| seed | int u |          | 当前种子值，默认值53213 |      |


## oc_clean_order
|     列名    |     类型     | 取值范围 |   代表含义   | 其他 |
|-------------|--------------|----------|--------------|------|
| id          | int          |          |              |      |
| state       | tinyint      |          | below        |      |
| house_id    | int          |          |              |      |
| user_id     | int          |          | 申请人       |      |
| message     | varchar(256) |          |              |      |
| create_at   | timestamp    |          |              |      |
| update_at   | timestamp    |          |              |      |
| serve_at    | timestamp    |          | 设定服务时间 |      |
| complete_at | timestamp    |          | 完成时间     |      |

* state: 默认（0），提交（1），已安排(2), 完成（3），取消（4）

## oc_repair_order

服务

|     列名    |     类型     | 取值范围 |   代表含义   | 其他 |
|-------------|--------------|----------|--------------|------|
| id          | int          |          |              |      |
| state       | tinyint      |          | below        |      |
| room_id     | int          |          |              |      |
| house_id    | int          |          |              |      |
| user_id     | int          |          |              |      |
| message     | varchar(256) |          |              |      |
| type        | tinyint      |          |              |      |
| serve_at    | timestamp    |          | 设定服务时间 |      |
| complete_at | timestamp    |          | 完成时间     |      |
| create_at   | timestamp    |          |              |      |
| update_at   | timestamp    |          |              |      |

* state: 默认（0），提交（1），完成（2），取消（3）
* type: 家电维修（0），房屋漏水（1），管道疏通/维修（2），房屋维修（3），其它（4）


## oc_house_state_log

房屋状态更改日志

|    列名    |    类型   | 取值范围 | 代表含义 | 其他 |
|------------|-----------|----------|----------|------|
| id         | int       |          |          |      |
| house_id   | int       |          |          |      |
| from_state | tinyint u |          |          |      |
| to_state   | tinyint u |          |          |      |
| create_at  | timestamp |          |          |      |



## oc_room_state_log

房屋状态更改日志

|    列名    |    类型   | 取值范围 | 代表含义 | 其他 |
|------------|-----------|----------|----------|------|
| id         | int       |          |          |      |
| room_id    | int       |          |          |      |
| from_state | tinyint u |          |          |      |
| to_state   | tinyint u |          |          |      |
| create_at  | timestamp |          |          |      |


## oc_renter_pay_bill

租房合同确立后，创建每笔支付订单。

|      列名      |    类型   | 取值范围 |       代表含义       | 其他 |
|----------------|-----------|----------|----------------------|------|
| id             | int       |          |                      |      |
| state          | tinyint u |          |                      |      |
| index          | tinyint u |          | 支付订单的顺序       |      |
| renter_bill_id | int       |          |                      |      |
| start_month    | tinyint u | [0,11]   | 此次支付起始月份     |      |
| for_month      | tinyint u | [1,12]   | 支付N个月房租        |      |
| rent_fee       | int u     |          | 租金                 |      |
| service_fee    | int u     |          | 服务费金额           |      |
| deposit        | int u     |          | 押金金额             |      |
| money_to_pay   | int u     |          | 此次支付需要支付金额 |      |
| reduce         | int u     |          | 减免金额             |      |
| coupon_reduce  | int u     |          | 优惠券减免           |      |
| money_paid     | int u     |          | 已支付金额           |      |
| need_pay_at    | timestamp |          | 需要缴纳房租的时间   |      |

state: 默认（0），分笔支付中（1），支付完成（2），取消（3）。

## oc_renter_payment

每一条代表一次支付的记录。如果是在线支付，会有online_payment_id链接到对应的第三方支付信息。

|        列名       |    类型   | 取值范围 |       代表含义       | 其他 |
|-------------------|-----------|----------|----------------------|------|
| id                | int       |          |                      |      |
| renter_bill_id    | int       |          |                      |      |
| money             | int       |          |                      |      |
| method            | tinyint u |          |                      |      |
| online_payment_id | int       |          | oc_renter_payment.id |      |
| create_at         | timestamp |          |                      |      |

* method: 线下支付宝（0），线下刷卡（1），现金（2），线上支付（3）
* money: 负数表示退款

## oc_advice

用户吐槽

|   列名  |     类型     | 取值范围 |  代表含义  | 其他 |
|---------|--------------|----------|------------|------|
| id      | int          |          |            |      |
| user_id | int          |          | oc_user.id |      |
| mobile  | varchar(20)  |          |            |      |
| email   | varchar(50)  |          |            |      |
| content | varchar(250) |          |            |      |




## oc_coupon_state_log

优惠券发放和使用日志。

|    列名    |    类型   | 取值范围 | 代表含义 | 其他 |
|------------|-----------|----------|----------|------|
| id         | int       |          |          |      |
| coupon_id  | int       |          |          |      |
| from_state | tinyint u |          |          |      |
| to_state   | tinyint u |          |          |      |
| create_at  | timestamp |          |          |      |

## oc_article

娱乐圈、彩客秀文章

|    列名   |      类型      | 取值范围 |   代表含义   | 其他 |
|-----------|----------------|----------|--------------|------|
| id        | int            |          |              |      |
| type      | tinyint u      |          |              |      |
| weight    | int            |          |              |      |
| create_at | timestamp      |          |              |      |
| author    | varchar(20)    |          | 作者姓名     |      |
| cover_img | varchar(256)    |          | 封面图像地址 |      |
| head_img  | varchar(256)    |          | 作者头像地址 |      |
| title     | varchar(50)    |          | 标题         |      |
| brief     | varchar(128)   |          | 内容简介     |      |
| content   | varchar(10000) |          | 文章全文     |      |

* type: 彩客秀（0），娱乐圈（1）
* weight: 权重，值越小，排名越靠前

## oc_bank_card

|    列名   |     类型    | 取值范围 |         代表含义        | 其他 |
|-----------|-------------|----------|-------------------------|------|
| id        | int         |          |                         |      |
| user_id   | int         |          | oc_user.id              |      |
| card      | varchar(24) |          | 卡号                    |      |
| bank_name | varchar(64) |          | 开户行                  |      |
| full_name | varchar(24) |          | 真实姓名                |      |
| type      | tinyint u   |          | below                   |      |
| validity  | date        |          | 有效期                  |      |
| cvv       | varchar(8)  |          | 信用卡CVV码             |      |
| comment   | varchar(64) |          | 备注,标记房租还是水电卡 |      |

* type: 借记卡（0），信用卡（1）

## oc_butler_task

管家任务

|       列名       |     类型     | 取值范围 |       代表含义       | 其他 |
| ---------------- | ------------ | -------- | -------------------- | ---- |
| id               | int          |          |                      |      |
| butler_id        | int          |          |                      |      |
| user_name        | varchar(32)  |          | 用户名               |      |
| tel_num          | varchar(20)  |          | 手机号               |      |
| channel_id       | int          |          | 渠道id               |      |
| create_at        | timestamp    |          | 创建时间             |      |
| update_at        | timestamp    |          | 上次更改             |      |
| show_room_time   | varchar(24)  |          | 看房时间             |      |
| show_room_style  | varchar(128) |          | 客户看房类型         |      |
| hope_price_id    | int          |          | 理想价位id           |      |
| status           | tinyint      |          | 任务状态             |      |
| room_id          | int          |          | room_id              |      |
| room_name_brief  | varchar(64)  |          | 房间名字介绍         |      |
| failure_reason   | varchar(64)  |          | 失败原因             |      |
| earnest_money_id | int          |          | 定金id               |      |
| bill_id          | int          |          | 租房合同id           |      |
| contact_content  | varchar(64)  |          | 已联系内容           |      |
| stars            | int          |          | 星级                 |      |
| rev_money        | int          |          | 尾款（签合同收到钱） |      |
| earnest_method   | varchar(64)  |          | 定金支付方式         |      |
| earnest_pay_time | date         |          | 定金支付时间         |      |
| rev_money_method | varchar(64)  |          | 尾款支付方式         |      |
| rev_money_time   | date         |          | 尾款支付时间         |      |

* status: 初始状态(0), 已分配(1), 已联系(2), 未成功(3), 未成功潜在客户(4), 改约时间(5), 已交定金(6), 直接签约(7), 已完成(表示合同完结8), 已审核(运维审核9)
* room_id: 没有确定(0), 确定后填写真正预约room_id(room_id)
* pay_method: 参考oc_renter_payment 线下支付宝（0），线下刷卡（1），现金（2），线上支付（3） 

## oc_earnest_money

管家任务定金表

|      列名      |    类型   | 取值范围 |       代表含义       | 其他 |
|----------------|-----------|----------|---------------------|------|
| id             | int       |          |                     |      |
| price          | int       |          | 定金金额            |      |
| state          | int       |          | 状态                |      |
| user_id        | int       |          | user id             |      |
| create_at      | timestamp |          |                     |      |

* state: 初始状态(0)， 已交定金还没签订合同（1）， 交定金签订合同（2）, 用户取消预定（3）,直接签合同（4）
* price: 负数表示退款

## oc_channel

渠道

|    列名         |      类型      | 取值范围 |   代表含义   | 其他 |
|-----------------|----------------|----------|--------------|------|
| id              | int            |          |              |      |
| channel_name    | varchar(32)    |          |    渠道名字   |      |


## oc_hope_price

心理价位

|    列名          |      类型      | 取值范围 |   代表含义   | 其他 |
|------------------|----------------|----------|--------------|------|
| id               | int            |          |              |      |
| hope_price_range | varchar(32)    |          |    价格范围  |      |

* hope_price_range: 为id对应的解释,不限(1)， 1500以下(2)，1500-2000(3)，2000-2500(4), 2500-3000(5), 3000-3500(6), 3500-4000(7), 4000-4500(8), 4500以上(9)

## oc_task_state_log

任务状态更改日志

|    列名    |    类型   | 取值范围 | 代表含义 | 其他 |
|------------|-----------|----------|----------|------|
| id         | int       |          |          |      |
| task_id    | int       |          |          |      |
| from_state | tinyint u |          |          |      |
| to_state   | tinyint u |          |          |      |
| create_at  | timestamp |          |          |      |

* from_state、to_state: 参照oc_butler_task state状态描述


## oc_sale_activity

特价房活动，每一行代表一个活动。

|    列名    |     类型     | 取值范围 |   代表含义   | 其他 |
|------------|--------------|----------|--------------|------|
| id         | int          |          |              |      |
| start_time | timestamp    |          |              |      |
| end_time   | timestamp    |          |              |      |
| state      | tinyint      |          | below        |      |
| name       | varchar(64)  |          |              |      |
| cover_img  | varchar(256) |          | cover image  |      |
| banner_img | varchar(256) |          | banner image |      |
| create_at  | timestamp    |          |              |      |

* state: default(0),active(1),over(2)


## oc_rooms_in_sale

参与活动的房间

|     列名    | 类型 | 取值范围 |       代表含义      | 其他 |
|-------------|------|----------|---------------------|------|
| id          | int  |          |                     |      |
| activity_id | int  |          | oc_sale_activity.id |      |
| room_id     | int  |          | oc_room.id          |      |
| price       | int  |          | 活动价格            |      |


## oc_pre_order

预定房间列表

|     列名     |    类型   | 取值范围 |   代表含义   | 其他 |
|--------------|-----------|----------|--------------|------|
| id           | int       |          |              |      |
| user_id      | int       |          | oc_user.id   |      |
| room_id      | int       |          | oc_room.id   |      |
| price        | int       |          | 约定的价格   |      |
| deposit_paid | int       |          | 已交付的定金 |      |
| state        | tinyint   |          | below        |      |
| create_at    | timestamp |          |              |      |

* state: default(0),已付定金(1),已取消(2),已完成入住(3)

## oc_system_log

|    列名   |      类型     | 取值范围 | 代表含义 | 其他 |
|-----------|---------------|----------|----------|------|
| id        | int           |          |          |      |
| app       | varchar(32)   |          | app name |      |
| username  | varchar(64)   |          |          |      |
| ip        | varchar(16)   |          |          |      |
| action    | varchar(128)  |          |          |      |
| method    | varchar(8)    |          |          |      |
| params    | varchar(1024) |          |          |      |
| create_at | timestamp     |          |          |      |


## oc_main_banner

网站首页，Banner配置。

|   列名  |     类型     | 取值范围 |    代表含义    | 其他 |
|---------|--------------|----------|----------------|------|
| id      | int          |          |                |      |
| weight  | int          |          |                |      |
| open    | tinyint      |          | 是否显示       |      |
| type    | tinyint      |          | below          |      |
| text    | varchar(128) |          | banner显示文字 |      |
| img     | varchar(256) |          | 图片地址       |      |
| url     | varchar(256) |          | 链接地址       |      |
| comment | varchar(32)  |          | 备注           |      |

* type：PC（0），手机（1），PC和手机都显示（2）

## oc_landlord_bill_increment

房东合同价格递增表

|     列名     |     类型     | 取值范围 |       代表含义      | 其他 |
| ------------ | ------------ | -------- | ------------------- | ---- |
| id           | int          |          |                     |      |
| bill_id      | int          |          | oc_landlord_bill.id |      |
| index        | int          |          | 第几年租金价格      |      |
| price        | decimal(9,6) |          | 实际价格            |      |
| create_at    | timestamp    |          |                     |      |
| increment_at | timestamp    |          | 递增房租价格时间    |      |

## oc_butler_check_fee

管家查询各表信息(退租、合同结束时查表，数据全部录入)
注：录入数据信息必须为退租当月信息。

|        列名       |      类型     | 取值范围 |      代表含义      | 其他 |
| ----------------- | ------------- | -------- | ------------------ | ---- |
| id                | int           |          |                    |      |
| butler_id         | int           |          | 抄表管家           |      |
| room_id           | int           |          | 房间id             |      |
| butler_check_time | date          |          | 本次抄表日期       |      |
| butler_check_val  | decimal(10,2) |          | 本次抄表数值       |      |
| used_val          | decimal(10,2) |          | 估算所用数值       |      |
| fee_payment       | decimal(10,2) |          | 需要缴费金额（元） |      |
| user_id           | int           |          | oc_user.id         |      |
| fee_type          | int           |          | 缴费类型           |      |
| method            | tinyint u     |          |                    |      |
| create_at         | timestamp     |          |                    |      |

* fee_type:水（0），电（1），燃气（2）
* method: 线下支付宝（0），线下刷卡（1），现金（2），线上支付（3）


## oc_company_elec_payment

公司代缴电费信息表

|       列名      |      类型     | 取值范围 |       代表含义       | 其他 |
| --------------- | ------------- | -------- | -------------------- | ---- |
| id              | int           |          |                      |      |
| charge_inst     | varchar(32)   |          | 收费单位名称         |      |
| house_id        | int           |          | 套房id               |      |
| elec_type       | int           |          | 用电类型             |      |
| elec_payment    | decimal(10,2) |          | 缴费金额（元）       |      |
| elec_last_time  | date          |          | 上次抄表日期         |      |
| elec_check_time | date          |          | 本次抄表日期         |      |
| elec_last_val   | decimal(10,2) |          | 上次抄表数值（度）   |      |
| elec_check_val  | decimal(10,2) |          | 本次抄表数值（度）   |      |
| elec_used_val   | decimal(10,2) |          | 本次所用电度数（度） |      |
| create_at       | timestamp     |          |                      |      |

* elec_type:居民用电(0)，一般工商业用电（1），大工业用电（2），农业用电（3）


## oc_user_elec_payment

租客缴电费信息表（每次录入公司代交电费后自动生成）

|       列名      |      类型     | 取值范围 |          代表含义          | 其他 |
| --------------- | ------------- | -------- | -------------------------- | ---- |
| id              | int           |          |                            |      |
| company_elec_id | int           |          | oc_company_elec_payment.id |      |
| user_id         | int           |          | oc_user.id                 |      |
| need_pay        | decimal(10,2) |          | 需要缴费金额（元）         |      |
| money_paid      | decimal(10,2) |          | 已经缴费金额（元）         |      |
| state           | int           |          | 支付状态                   |      |
| elec_check_time | date          |          | 本次缴费日期               |      |
| elec_used_val   | decimal(10,2) |          | 用电度数(价格做估算)       |      |

* state: 默认（0），未支付（1），支付完成（2）


## oc_company_water_payment

公司代缴水费信息表(水费+污水处理费)

|       列名       |      类型     | 取值范围 |      代表含义      | 其他 |
| ---------------- | ------------- | -------- | ------------------ | ---- |
| id               | int           |          |                    |      |
| house_id         | int           |          | 套房id             |      |
| water_payment    | decimal(10,2) |          | 缴费金额           |      |
| water_last_time  | date          |          | 上次抄表日期       |      |
| water_check_time | date          |          | 本次抄表日期       |      |
| water_last_val   | decimal(10,2) |          | 上次读数（吨）     |      |
| water_check_val  | decimal(10,2) |          | 本次读数（吨）     |      |
| water_used_val   | decimal(10,2) |          | 本次所用水量（吨） |      |
| create_at        | timestamp     |          |                    |      |


## oc_user_water_payment

租客缴水费信息表（每次录入公司代交水费后自动生成）

|       列名       |      类型     | 取值范围 |           代表含义          | 其他 |
| ---------------- | ------------- | -------- | --------------------------- | ---- |
| id               | int           |          |                             |      |
| company_water_id | int           |          | oc_company_water_payment.id |      |
| user_id          | int           |          | oc_user.id                  |      |
| need_pay         | decimal(10,2) |          | 本次需要缴费金额（元）      |      |
| money_paid       | decimal(10,2) |          | 本次已经缴费金额（元）      |      |
| state            | int           |          | 支付状态                    |      |
| water_check_time | date          |          | 本次缴费日期                |      |
| water_used_val   | decimal(10,2) |          | 用户所用水量（价格做估算）  |      |

* state: 默认（0），支付中（1），支付完成（2）


## oc_company_gas_payment

公司代缴燃气费信息表

|      列名      |      类型     | 取值范围 |        代表含义        | 其他 |
| -------------- | ------------- | -------- | ---------------------- | ---- |
| id             | int           |          |                        |      |
| house_id       | int           |          | 套房id                 |      |
| charge_inst    | varchar(32)   |          | 收费单位名称           |      |
| gas_type       | int           |          | 用气类型               |      |
| gas_payment    | decimal(10,2) |          | 缴费金额               |      |
| gas_last_time  | date          |          | 上次抄表日期           |      |
| gas_check_time | date          |          | 本次抄表日期           |      |
| gas_last_val   | decimal(10,2) |          | 上次读数（立方米）     |      |
| gas_check_val  | decimal(10,2) |          | 本次读数（立方米）     |      |
| gas_used_val   | decimal(10,2) |          | 本次所用气量（立方米） |      |
| create_at      | timestamp     |          |                        |      |

* gas_type:居民用气(0)，学校用气（1），机关事业和企业单位用气（2）

## oc_user_gas_payment

租客缴燃气费信息表（每次录入公司代交燃气费后自动生成）

|      列名      |      类型     | 取值范围 |          代表含义          | 其他 |
| -------------- | ------------- | -------- | -------------------------- | ---- |
| id             | int           |          |                            |      |
| company_gas_id | int           |          | oc_company_gas_payment.id  |      |
| user_id        | int           |          | oc_user.id                 |      |
| need_pay       | decimal(10,2) |          | 本次需要缴费金额（元）     |      |
| money_paid     | decimal(10,2) |          | 本次已经缴费金额（元）     |      |
| state          | int           |          | 支付状态                   |      |
| gas_check_time | date          |          | 本次缴费日期               |      |
| gas_used_val   | decimal(10,2) |          | 用户所用气量（价格做估算） |      |

* state: 默认（0），支付中（1），支付完成（2）

## oc_other_fee_payment

每一条代表一次支付的记录。如果是在线支付，会有third_payment_id链接到对应的第三方支付信息。

|       列名       |      类型     | 取值范围 |        代表含义        | 其他 |
| ---------------- | ------------- | -------- | ---------------------- | ---- |
| id               | int           |          |                        |      |
| fee_type         | int           |          | 缴费类型               |      |
| user_id          | int           |          | oc_user.id             |      |
| check_fee_id     | int           |          | oc_butler_check_fee.id |      |
| money            | decimal(10,2) |          |                        |      |
| method           | tinyint u     |          |                        |      |
| third_payment_id | int           |          | oc_third_payment.id    |      |
| create_at        | timestamp     |          |                        |      |

* method: 线下支付宝（0），线下刷卡（1），现金（2），线上支付（3）
* fee_type:水（0），电（1），燃气（2）
* check_fee_id:默认没有，只有在退租或合同结束时结算水电气费用时，生成支付订单才用。


## oc_user_wallet

用户钱包，如果remain_money>0，水电气费用金额会从中扣除。

|       列名       |      类型     | 取值范围 |       代表含义      | 其他 |
| ---------------- | ------------- | -------- | ------------------- | ---- |
| id               | int           |          |                     |      |
| user_id          | int           |          | oc_user.id          |      |
| remain_money     | decimal(10,2) |          | 钱包剩余金额        |      |
| create_at        | timestamp     |          |                     |      |


## oc_main_page

首页内容配置表

|  列名  |     类型     | 取值范围 |  代表含义  | 其他 |
|--------|--------------|----------|------------|------|
| id     | int          |          |            |      |
| column | tinyint u    |          | 栏目id     |      |
| index  | tinyint u    |          | 栏目内序号 |      |
| img    | varchar(128) |          | 图片       |      |
| url    | varchar(128) |          | 跳转地址   |      |
| config | varchar(512) |          | json配置项 |      |
| open   | tinyint      |          | 是否打开   |      |
| show   | tinyint      |          | 显示在哪里 |      |

`show`:  

* 0，PC和Mobile都显示
* 1，只显示在PC
* 2，只显示在Mobile

`column`: 

* 0，表示首页Banner图的配置。`config`可配置,格式为：
`{"alt":"图片描述信息"}`。
* 1，表示精选房间的配置，此时，`config`需要配置，格式为：`{"price":2000,"name":"大冲城市花园3栋6座301"}`。
* 2，表示房屋列表快速链接的配置，包括：南山区、福田区、特价房。
* 3，表示彩客秀配置，此时，需要配置`config`，格式为：`{"title":"小女神的设计师之路","brief":"XXXXXX"}`.
* 4，表示娱乐圈。`config`可配置,格式为：
`{"alt":"图片描述信息"}`。
* 5，表示新闻配置。'config'需要配置,格式为:`{"title":"测试1"，"from":"深圳日报","create_time":"2015-10-11"}`。


## oc_special_room

特价房

|    列名   |  类型 |    取值范围   | 代表含义 |
|-----------|-------|---------------|----------|
| id        | int   |               |          |
| room_id   | int   | PK of oc_room |          |
| old_price | float |               | 原来价格 |


## oc_boundary

|      列名     |      类型      | 取值范围 |         代表含义        | 其他 |
| ------------- | -------------- | -------- | ----------------------- | ---- |
| id            | int            |          | id                      |      |
| center_lat    | decimal(13,10) |          | 区域地图上中心纬度      |      |
| center_lng    | decimal(13,10) |          | 区域地图上中心经度      |      |
| sw_lat        | decimal(13,10) |          | 区域地图上西南角纬度    |      |
| sw_lng        | decimal(13,10) |          | 区域地图上西南角经度    |      |
| ne_lat        | decimal(13,10) |          | 区域地图东北角纬度      |      |
| ne_lng        | decimal(13,10) |          | 区域地图东北角经度      |      |
| show_zoom     | int            |          | 地图显示时级别          |      |
| boundary      | text           |          | json格式数据            |      |
| show_name     | varchar(64)    |          | 地图显示(如:南山区)     |      |
| grade         | int            |          | 级别(小区/商圈/区域/市) |      |
| hide          | int            |          | 是否在地图上隐藏(默认0) |      |
| boundary_desc | varchar(512)   |          | 位置描述信息            |      |

`boundary`:

* 该区域所表示的范围,由多组经纬度坐标围成的一个区域,为json数据字符串组成.
* 格式如下:
```json
{
    "main":[
        {"pointLon":"113.96189","pointLat":"22.550857"},
        {"pointLon":"113.961921","pointLat":"22.5488"},
        {"pointLon":"113.962195","pointLat":"22.547987"},
        {"pointLon":"113.964019","pointLat":"22.548375"},
        {"pointLon":"113.964234","pointLat":"22.550515"},
        {"pointLon":"113.964329","pointLat":"22.551041"}
    ],
    "center":[{"pointLon":"113.96189","pointLat":"22.550857"}]}
```

`grade`:

* 1:小区(如:大冲城市花园)
* 2:商圈(如:科技园)
* 3:区域(如:南山区)
* 4:市(如:深圳市)
* 5:地铁站（如:世界之窗）

## oc_user_question

|     列名    |     类型     | 取值范围 |    代表含义    | 其他 |     |
| ----------- | ------------ | -------- | -------------- | ---- | --- |
| id          | int          |          | id             |      |     |
| user_id     | int          |          | oc_user.id     |      |     |
| question    | varchar(256) |          | 提问问题       |      |     |
| answer      | text         |          | 答案           |      |     |
| thumbs_up   | int          |          | 点赞数量       |      |     |
| device_type | varchar(32)  |          | 提问时设备类型 |      |     |
| hide        | int          |          | 是否显示       |      |     |
| create_at   | timestamp    |          | 创建时间       |      |     |
| update_at   | timestamp    |          |                |      |     |
| room_id     | int          |          | 哪个房间提问   |      |     |

## oc_question_of_room
`room`和`question`的映射关系。

|     列名    | 类型 | 取值范围 |       代表含义      | 其他 |
| ----------- | ---- | -------- | ------------------- | ---- |
| id          | int  |          | id                  |      |
| room_id     | int  |          | 房间id              |      |
| question_id | int  |          | oc_user_question.id |      |


## oc_question_of_house

## oc_question_of_community
结构跟`oc_question_of_room`类似


## oc_life_service

|       列名       |      类型      | 取值范围 |    代表含义   | 其他 |     |
| ---------------- | -------------- | -------- | ------------- | ---- | --- |
| id               | int            |          | id            |      |     |
| service_icon_url | varchar(256)   |          | icon的oss地址 |      |     |
| service_name     | varchar(128)   |          | 服务场地名字  |      |     |
| line_icon_url    | varchar(256)   |          | 路线icon地址  |      |     |
| line_desc        | text           |          | 路线描述      |      |     |
| longtitude       | decimal(13,10) |          | 经度          |      |     |
| latitude         | decimal(13,10) |          | 纬度          |      |     |
| service_type     | int            |          | 服务类型      |      |     |

* service_type: 未定义(0), 购物(1), 美食(2), 运动(3), 娱乐(4)


## oc_life_service_of_community

|       列名      | 类型 | 取值范围 |      代表含义      | 其他 |
| --------------- | ---- | -------- | ------------------ | ---- |
| id              | int  |          | id                 |      |
| community_id    | int  |          | 小区id             |      |
| life_service_id | int  |          | oc_life_service.id |      |
| hide            | int  |          | 是否显示           |      |
| serial_num      | int  |          | 排序               |      |

* hide: 显示(0), 隐藏(1)

## oc_user_destination

|    列名   |     类型    | 取值范围 |  代表含义  | 其他 |     |
| --------- | ----------- | -------- | ---------- | ---- | --- |
| id        | int         |          | id         |      |     |
| room_id   | int         |          | oc_room.id |      |     |
| user_id   | int         |          | oc_user.id |      |     |
| dest_name | varchar(64) |          | 目的地     |      |     |
| create_at | timestamp   |          | 创建时间   |      |     |

## oc_hobby
爱好

|       列名      |     类型    |    取值范围    |  代表含义  | 其他 |     |
| --------------- | ----------- | -------------- | ---------- | ---- | --- |
| id              | int         |                | id         |      |     |
| name            | varchar(32) |                | 名字       |      |     |
| detail          | text        |                | 爱好描述   |      |     |
| parent_hobby_id | int         | PK of oc_hobby | 所属爱好   |      |     |
| display         | int         | 0-1            | 显示在筛选 |      |     |
| serial_num      | int         |                | 排序       |      |     |

* serial_num: 值小的在最前面

## oc_hobby_of_user
用户爱好

|    列名    |     类型    | 取值范围 |    代表含义   | 其他 |     |
| ---------- | ----------- | -------- | ------------- | ---- | --- |
| id         | int         |          | id            |      |     |
| user_id    | int         |          | oc_user.id    |      |     |
| hobby_id   | int         |          | oc_hobby.id   |      |     |
| grade      | varchar(16) |          | hobby级别描述 |      |     |
| serial_num | int         |          | 排序          |      |     |

## oc_house_layout

代表一种户型图

|     列名    |     类型    | 取值范围 |    代表含义    | 其他 |     |
| ----------- | ----------- | -------- | -------------- | ---- | --- |
| id          | int         |          | id             |      |     |
| img         | varchar(64) |          | 户型图oss地址  |      |     |
| layout_desc | varchar(64) |          | 户型图描述信息 |      |     |

## oc_slice

代表一片区域，一个大厅可以由多个slice组成，一个房间也可以由多个slice组成.


|       列名      |     类型    | 取值范围 |      代表含义      | 其他 |     |
| --------------- | ----------- | -------- | ------------------ | ---- | --- |
| id              | int         |          | id                 |      |     |
| boundary        | text        |          | json数据格式       |      |     |
| slice_desc      | varchar(64) |          | slice描述信息      |      |     |
| house_layout_id | int         |          | oc_house_layout.id |      |     |

`boundary`:
* 该区域所表示的范围,由多组像素坐标围成的一个区域,相对坐标点为左下角坐标,为json数据字符串组成，编辑需要按照区域顺序填写，切忌不能跳跃填写．
* 格式如下:
[{"pointX":"162","pointY":"60"},{"pointX":"162","pointY":"290"},{"pointX":"367","pointY":"60"},{"pointX":"367","pointY":"288"}]

`slice_desc`:
* 命名规则:户型图名字／地址＋区域介绍


## oc_slice_of_scene

一个场景信息

|     列名     |     类型     | 取值范围 |       代表含义       | 其他 |     |
| ------------ | ------------ | -------- | -------------------- | ---- | --- |
| id           | int          |          | id                   |      |     |
| scene_name   | varchar(128) |          | 场景命名             |      |     |
| slice_id     | int          |          | oc_slice.id          |      |     |
| house_img_id | int          |          | oc_image_of_house.id |      |     |
| comm_id      | int          |          | oc_community.id      |      |     |
| room_id      | int          |          | oc_room.id           |      |     |
| point_x      | int          |          | 该场景的摄影点x坐标  |      |     |
| point_y      | int          |          | 摄影点y坐标          |      |     |
| v_degree     | int          |          | 度数（正北顺时度数） |      |     |

`slice_id`:(非必须)
* 户型图中某个区域
`house_img_id`:(非必须)
* 每个场景需要和套房的哪个户型图关联．（考虑到复式楼）
`comm_id`:(非必须)
* 小区场景,如果该场景名字属于小区场景名,需要配置该项
`room_id`:(非必须)
*　这个场景对应的房间id，如果不是room则不用填写


## oc_expense_fee

公司支出费用，目前包括（中介推荐费\租客推荐费\非租客推荐费\活动返现）
注：推荐费是在成功签订合同后，公司支出的。

|    列名   |     类型     | 取值范围 |      代表含义     |    其他    |     |
| --------- | ------------ | -------- | ----------------- | ---------- | --- |
| id        | int          |          | id                |            |     |
| fee_type  | int          |          | 费用类型          |            |     |
| money     | int          |          | 支出金额(大于0)   |            |     |
| pay_time  | date         |          | 付款时间          |            |     |
| payee     | varchar(32)  |          | 收款人            |            |     |
| pay_desc  | varchar(128) |          | 付款详细描述信息  |            |     |
| create_at | timestamp    |          | 创建时间          |            |     |
| bill_id   | int          |          | oc_renter_bill.id | 推荐费必填 |     |
| house_id  | int          |          | oc_house.id       | 维修必填   |     |

`fee_type`:
* 0:未选择
* 1:中介推荐费
* 2:租客推荐费
* 3:朋友推荐费(非租客/非中介)
* 4:活动返现费
* 5:维修费支出

## oc_back_rent_reason
退租原因

| 列名 |     类型     | 取值范围 | 代表含义 | 其他 |     |
| ---- | ------------ | -------- | -------- | ---- | --- |
| id   | int          |          | id       |      |     |
| name | varchar(128) |          | 退租原因 |      |     |

## oc_back_rent
退租列表

|      列名      | 类型 | 取值范围 |           代表含义          | 其他 |     |
| -------------- | ---- | -------- | --------------------------- | ---- | --- |
| id             | int  |          | id                          |      |     |
| reason_id      | int  |          | oc_back_rent_reason.id      |      |     |
| bill_id        | int  |          | oc_renter_bill.id退租前合同 |      |     |
| back_rent_time | date |          | 退租时间                    |      |     |
| create_at      |      |          | 创建时间                    |      |     |


