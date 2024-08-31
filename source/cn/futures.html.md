---
title: Coinstore官方API文档

language_tabs: # must be one of https://git.io/vQNgJ
  

toc_footers:

includes:
  
language: 简体中文

other_language: English

present_url: /cn

url: /en

active: active

other_active: 

menu: 菜单

create_api: 创建 API Key

spot_goods: 现货

spot_goods_url: 'index.html'

contract: 永续合约

contract_active: active

contract_url: 'futures.html'

searchText: 搜索

search: true

code_clipboard: true
---

# 介绍

欢迎使用Coinstore开发者文档，此文档是Coinstore API的唯一官方文档。

本文档提供了相关API的使用方法介绍。

RESTful API包含了资产，订单及行情等接口。

Websocket则提供了行情相关的接口及推送服务。

Coinstore API提供的服务会在此持续更新，请大家及时关注。



# 快速开始

## 接入准备

如需使用API，请先登录网页端，通过【用户中心】-【API管理】创建一个API key，再据此文档详情进行开发和交易。

您可以点击 'https://www.coinstore.com/#/user/bindAuth/ManagementAPI' 创建 API Key。

每个用户可创建5组API Key，每组API key可以绑定5个不同的IP地址。API key一旦绑定了IP地址，则只能从绑定的IP地址使用该API key调用API接口。出于安全考虑，强烈建议您为API key绑定相应的IP地址。

创建成功后请务必记住以下信息：

- `API Key`  API 访问密钥
- `Secret Key` 签名认证加密所使用的密钥

## 接口类型
Coinstore为用户提供两种接口，您可根据自己的使用场景和偏好来选择适合的方式进行查询行情、交易。

**REST API**

REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如右：

- 在RESTful架构中，每一个URL代表一种资源；
- 客户端和服务器之间，传递这种资源的某种表现层；
- 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。 

交易或资产等一次性操作，建议开发者使用REST API进行操作

**WebSocket API**

WebSocket是HTML5一种新的协议（Protocol）。它实现了客户端与服务器全双工通信，通过一次简单的握手就可以建立客户端和服务器连接，服务器可以根据业务规则主动推送信息给客户端。

市场行情和买卖深度等信息，建议开发者使用WebSocket API进行获取。

### 接口鉴权

以上两种接口均包含公共接口和私有接口两种类型。

公共接口可用于获取基础信息和行情数据。公共接口无需认证即可调用。

私有接口可用于交易管理。每个私有请求必须使用您的API Key进行签名验证。

## 接入URLs

**REST API**

`https://futures.coinstore.com/api`

**WebSocket**

`wss://ws-futures.coinstore.com/socket.io/?EIO=3&transport=websocket`

为保证API服务的稳定性，建议使用日本AWS云服务器进行访问。如使用中国大陆境内的客户端服务器，连接的稳定性将难以保证。


## <span id="a4">签名认证</span>

**签名说明**

API 请求在通过 internet 传输的过程中极有可能被篡改，为了确保请求未被更改，除公共接口（基础信息，行情数据）外的私有接口均必须使用您的 API Key 做签名认证，以校验参数或参数值在传输途中是否发生了更改。

**签名算法**

使用HmacSHA256哈希函数作为签名函数
```
Mac hmacSha256 = Mac.getInstance("HmacSHA256");
```

**签名步骤**
    
1、签名有效字符串为请求参数和请求体
>注意：请求参数和请求体不进行任何排序或者去除空格，直接拼接成字符串作为payload
```
实例1：GET请求查询字符串
 ?symbol=aaaa88&size=10
 String payload = “symbol=aaaa88&size=10”；

实例2 :Post请求体
 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}
 String payload = “{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；

实例3：混合请求
 ?symbol=aaaa88&size=10
 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}
 String payload = “symbol=aaaa88&size=10{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；
```
2、使用签名函数对时间戳获得哈希值
>注意： X-CS-EXPIRES为13位时间戳，需要除以30000获取一个类时间戳，对其进行签名函数计算，获得函数值作为第三步的秘钥   
```
 String time = String.valueOf(X-CS-EXPIRES / 30_000);
 hmacSha256.init(new SecretKeySpec(Secret_Key.getBytes(), "HmacSHA256"));
 byte[] hash = hmacSha256.doFinal(time.getBytes());
 String key = Hex.toHexString(hash);
```
3、使用签名函数对签名有效字符串获得哈希值
    
```
 hmacSha256.reset();
 hmacSha256.init(new SecretKeySpec(key.getBytes(), "HmacSHA256"));
 hash = hmacSha256.doFinal(payload.getBytes());
 String sign= Hex.toHexString(hash);

```

# API接入说明

## <span id="a3">请求格式</span>
所有的API请求都是restful，目前只有两种方法：GET和POST。
- GET请求：所有的参数都在路径参数里
- POST请求: 路径里可以设置参数，参数可以以JSON格式发送在请求主体（body）里，没有参数的需要传{}

一个合法的请求由以下几部分组成：
- 方法请求地址：即访问服务器地址futures.coinstore.com，比如https://futures.coinstore.com/api/trade/order/place
- 必须和可选参数。
- X-CS-APIKEY： 即用户申请的API Key。
- X-CS-EXPIRES：您发出请求的时间戳。如：1629291143107。
- X-CS-SIGN：签名计算得出的字符串，用于确保签名有效和未被篡改。

>注意：X-CS-APIKEY ，X-CS-EXPIRES ，X-CS-SIGN 三个参数都在请求头中，另外需要设置'Content-Type':'application/json'。

## <span id="a3">返回格式</span>

所有的接口返回都是JSON格式。在JSON的第一层有3个字段code，msg和data。前两个字段表示请求状态和描述，实际的业务数据在data字段里。

```json
{
    "code": "0",
    "msg": "suc",
    "data": // per API response data in nested JSON object
}
```
以下是一个返回格式的样例：

| 字段 | 数据类型  | 描述       |
| ---- | -----  | ---------- |
| code | int | 0：成功，其他失败 |
| msg  | string | 状态或错误描述 |
| data | object | 业务数据  |


## 错误信息

**HTTP状态码**

HTTP常见的错误码如下：
- 400 Bad Request – Invalid request forma 请求格式无效

- 401 signature failed – Invalid API Key 无效的API Key

- 404 service not found 没有找到请求

- 429 too many visits 请求太频繁被系统限流

- 500 internal server error – 服务器内部错误

**业务状态码**

如果失败，response msg 带有错误描述信息, 对应的状态码描述如下：

| 状态码 | 说明                             | 备注                       |
| ------ | -------------------------------- | -------------------------- |
| 0      | 成功                             | code=0 成功， code >0 失败 |

# 基础信息


## <span id="1">币种币对信息</span>

获取币种币对信息

### HTTP请求: 
- GET /api/configs/public


> 响应 

```json
{
	"data": {
		"contracts": [
			{
				"contractId": 100300034,
				"currencyId": 30,
				"name": "BTCUSDT",
				"displayName": "BTC/USDT",
				"baseAsset": "BTC",
				"quoteAsset": "USDT",
				"marginAsset": "USDT",
				"tickSize": 0.5,
				"priceScale": 1,
				"maxOrderSize": 20000,
				"minOrderSize": 1,
				"takerFeeRate": 0.0006,
				"makerFeeRate": 0.00025,
				"contractSize": 0.001,
				"minMaintRate": 0.003,
				"fundingInterval": 3,
				"tags": "",
				"weight": 0,
				"riskLimits": [
					{
						"maxSize": 40000,
						"maintRate": 0.005,
						"leverage": 100
					},
					{
						"maxSize": 80000,
						"maintRate": 0.01,
						"leverage": 50
					},
					{
						"maxSize": 100000,
						"maintRate": 0.025,
						"leverage": 20
					},
					{
						"maxSize": 500000,
						"maintRate": 0.05,
						"leverage": 10
					},
					{
						"maxSize": 1000000,
						"maintRate": 0.12,
						"leverage": 5
					},
					{
						"maxSize": 2000000,
						"maintRate": 0.15,
						"leverage": 5
					},
					{
						"maxSize": 5000000,
						"maintRate": 0.15,
						"leverage": 4
					},
					{
						"maxSize": 99999999,
						"maintRate": 0.25,
						"leverage": 2
					}
				]
			}
		],
		"currencies": [
			{
				"currencyId": 30,
				"name": "USDT",
				"disableTransferIn": 0,
				"disableTransferOut": 0
			}
		],
		"version": 3
	},
	"code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |   |   |  |

### 响应数据

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int      |  0：成功，其他失败           |
| msg | String  |  错误信息 |
| data | Object   |   业务数据 |
|- currencies | List \<Currency>    | 币种列表 |
|- contracts  | List \<Contract>   | 币对列表 |
|- version  | Long   | 版本号 |


#### Currency

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| currencyId | int  | 币种id |
| name | String  | 币种名称 |
| disableTransferIn | boolean | 是否可以划转至合约  默认false 能划转 |
| disableTransferOut | boolean | 是否可以划转至现货  默认false 能划转 |

#### Contract

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- | 
| contractId | int  | 合约id | 
| currencyId | int  | 保证金货币id |
| name | String  | 合约名称 BTCUSDT | 
| displayName | String  | 合约显示名称 BTC/USDT |
| baseAsset | String  | 商品货币名称 BTC |   
| quoteAsset | String  | 计价货币名称 USDT | 
| marginAsset | String  | 保证金货币名称 USDT |  
| tickSize | BigDecimal  | 最小报价单位 | 
| priceScale | int  | 价格精度 | 
| maxOrderSize | int |  单笔下单最大张数，默认0，无限制 | 
| minOrderSize | int |  单笔下单最小张数，默认0，无限制 |  
| takerFeeRate | BigDecimal  | Taker手续费率 |  
| makerFeeRate | BigDecimal  | Maker手续费率 | 
| contractSize | BigDecimal  | 合约单位 |   
| minMaintRate | BigDecimal  | 最小维持保证金率 | 
|fundingInterval | int  | 互换频率 单位 秒  | 
|weight | int  |排序权重  倒叙 | 
| tags | String[] |  标签    mock, hot, new, ... |  
| riskLimits | RiskLimit[]  | 风险限额 |  

#### RiskLimit

|       code  |  type  |   comment    |
| ---- | ----- | ---------- | 
| maxSize | long|  风险限额张数 |
| maintRate | BigDecimal |  维持保证金率 |
| leverage | int |  杠杆倍数 |

# 账户相关


## <span id="1">资产余额</span>

获取用户资产余额

### HTTP请求: 
- POST  /api/future/queryAvail


> 响应 

```json
{
    "code": 0,
    "msg": "success",
    "data": [
        {
            "currencyId": 28,
            "totalBalance": "997901.568361825",
            "available": "997901.568361825",
            "frozenForTrade": "0",
            "initMargin": "0",
            "frozenInitMargin": "0",
            "closeProfitLoss": "-121.3079",
            "dailyProfitLoss": "0",
            "unRealizedProfit": "0",
            "accountEquity": "997901.568361825",
            "leverLevel": "0",
            "toBtc": "0",
            "accountId": 100023,
            "accountType": 1,
            "contractId": null,
            "accountEquityBigDecimal": 997901.568361825,
            "availableBigDecimal": 997901.568361825
        }
    ]
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |

### 响应数据

|       code        |  type  |                      comment                       |
| ---- | ----- | ---------- |
| code | int        |   0：成功，其他失败         |
| msg | String    | 错误信息 |
| data | Object []    |  业务数据 |
| - currencyId| string       |   保证金id  |
| - totalBalance| string      |  总资产   |
| - available| string        | 可用资产 |
| - frozenForTrade| string       | 委托冻结资产|
| - initMargin| string        | 已占用保证金|
| - frozenInitMargin| string       | 委托冻结保证金|
| - closeProfitLoss| string        | 已实现盈亏|
| - dailyProfitLoss| string        | 当日已实现盈亏|
| - closeProfitLoss| string        | 已实现盈亏|
| - unRealizedProfit| string        | 未实现盈亏|
| - accountEquity| string        | 净资产|
| - leverLevel| string        | 整体杠杆水平|
| - unRealizedProfit| string        | 未实现盈亏|
| - toBtc| string        | btc资产价值|
| - accountId| string        | 账户id|
| - accountType| string        | 账户类型  1  主账户 |

# 仓位相关

## <span id="1">仓位查询</span>

查询用户合约仓位

### HTTP请求: 
- GET  /api/future/queryPosi


> 响应 

```json
{
    "code": 0,
    "msg": "success",
    "data": [
        {
            "lastPrice": "3140.3",
            "markPrice": "3140.2",
            "initMarginRate": "0.01",
            "posiQty": "-1",
            "contractId": 100300027,
            "marginType": 1,
            "accountId": 100023,
            "accountType": 1,
            "available": "12.09855181610000241",
            "openAmt": "3.12765",
            "initMargin": "0.03315309",
            "posiStatus": 1,
            "closeProfitLoss": "0",
            "maintainMarginRate": "0.005",
            "frozenCloseQty": "0",
            "frozenOpenQty": "0",
            "contractUnit": "0.001",
            "openPrice": "3127.65",
            "unRealizedProfit": "-0.01255",
            "liquidationPrice": "15256.27",
            "forwardCross": true,
            "crossMargin": true,
            "extraMargin": "0"
        }
    ]
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |

### 响应数据

|       code        |  type  |     comment                       |
| ---- | ----- | ---------- |
| code | int        |  0：成功，其他失败          |
| msg | String    | 错误信息 |
| data | Object []    |  业务数据 |
| - currencyId| string     |   保证金id  |
| - posiQty| string       |  持仓量，大于0多头，小于0空头  |
| - openAmt| string       | 开仓金额 |
| - initMargin| string        | 初始保证金|
| - posiStatus| string        | 1正常，2等待强平|
| - marginType| string        | 保证金类型，1全仓2逐仓|
| - closeProfitLoss| string        | 已实现盈亏|
| - initMarginRate| string        | 初始保证率|
| - maintainMarginRate| string        | 维持保证金|
| - frozenCloseQty| string        | 平仓委托冻结量|
| - frozenOpenQty| string        | 开仓委托冻结量|
| - extraMargin| string        | 额外保证金   deprecated|
| - contractUnit| string        | 合约单位|
| - openPrice| string        | 开仓均价|
| - unRealizedProfit| string        | 未实现盈亏|
| - liquidationPrice| string        | 预估强评价|
| - lastPrice| string        | 最新成交价|
| - markPrice| string        | 标记价格 * 持仓数量 * 合约面值|
| - accountId| string        | 账户id|
| - accountType| string        | 账户类型  1  主账户|

## <span id="1">调整保证金</span>


### HTTP请求: 
- POST  /api/future/adjustMargin


> 响应 

```json
{
    "code": 0,
    "msg": "success",
```

### 请求体

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  contractId| int|Y| 交易对ID|
|   margin| int|Y|  保证金额（>0 : 增加保证金，<0 ：减少保证金）|

### 响应数据

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int        | 0：成功，其他失败           |
| msg | String    | 错误信息 |
| data | Object []    |  业务数据 |

## <span id="1">调整杠杆倍数</span>


### HTTP请求: 
- POST  /api/configs/adjustPositionConfig


> 响应 

```json
{
    "code": 0,
    "msg": "success",
```

### 请求体

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  contractId| int|Y| 交易对ID|
|   leverage| int|Y|  杠杆倍数, 0为自动模式  逐仓 不能为0|
|   isolated|boolean|Y| 保证金类型 默认 false全仓 true逐仓|

### 响应数据

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int        |  0：成功，其他失败          |
| msg | String    | 错误信息 |
| data | Object []    |  业务数据 |

# 订单相关

## <span id="1">创建订单</span>
创建订单

### HTTP请求: 
- POST /api/trade/order/place

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| accountType| int|N| 账号类型 |
| clientOrderId| string|N| 客户端委托ID,如果不传使用uuid |
| contractId| int|Y| 合约ID|
| marginRate| string|Y| 保证金率,marginRate=1/leverage 范围:0-1|
| leverage| int|Y| 保证金倍数|
| marginType| int|Y| 保证金类型，全仓1，逐仓2|
| orderSubType| int|Y|0（默认值），1（被动委托），2（最近价触发条件委托），3（指数触发条件委托），4（标记价触发条件委托）|
| orderType| int|Y| 委托类型，1（限价），3（市价）|
| positionEffect| int|Y| 开平标志，开仓1，平仓2  平仓单必须传2|
| price| string|N| 委托价格,order_type等于3（市价）时非必填|
| quantity| string|Y| 委托数量|
| side| int|Y| 买1，卖-1|
| stopPrice| string|optional| 止损价格，order_type等于3（市价）时非必填 //条件单时|                                                                


> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": "1712040078475777"
}
```                                 

### 响应数据

|       code        |  type  |                        comment                       |
| ----------------- | ------ |  -------------------------------------------------- |
| code            | int    |      0：成功，其他失败                                               |
| msg         | string    |     错误信息                                    |
| data            |  string   |   订单id                                               |



## <span id="2">获取当前订单</span>

获取当前订单

### HTTP请求: 

- GET /api/trade/order/active


> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": [
        {
            "side": 1,
            "initMarginRate": "0.01",
            "contractId": 100280034,
            "marginType": 2,
            "orderQty": "1",
            "orderPrice": "1",
            "stopPrice": "0",
            "orderType": 1,
            "orderSubType": 0,
            "positionEffect": 1,
            "accountId": 100023,
            "orderId": "1712035767779585",
            "timeInForce": 1,
            "accountType": 1,
            "contractUnit": "0.001",
            "applId": 2,
            "clOrderId": "c07fb0395d304590ac9277b71741aeaf",
            "orderTime": 1632724540258000,
            "orderStatus": 2,
            "matchQty": "0",
            "matchAmt": "0",
            "cancelQty": "0",
            "matchTime": 0,
            "feeRate": "0.00025",
            "avgPrice": "0",
            "fcOrderId": "",
            "stopCondition": 0,
            "minimalQuantity": null
        }
    ]
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |


### 响应数据

|       code        |  type  |                       comment                       |
| ----------------- | ------ |  -------------------------------------------------- |
| code            | int    | 0                     |     0：成功，其他失败                                               |
| msg         | string    |                 |    错误信息                                    |
| data            |  list  |      |
| - accountType| int  | 账号类型 |
| - clientOrderId| string  | 客户端委托ID,如果不传使用uuid |
| - contractId| int  | 合约ID|
| - initMarginRate| string  | 保证金率，全仓时>0，逐仓时>0|
| - marginType| int  | 保证金类型，全仓1，逐仓2|
| - minimalQuantity| string  | 最新成交数量，order_type等于3（市价）时非必填//前端不传|
| - orderSubType| int  |0（默认值），1（被动委托），2（最近价触发条件委托），3（指数触发条件委托），4（标记价触发条件委托）|
| - orderType| int  | 委托类型，1（限价），3（市价）|
| - positionEffect| int  | 开平标志，开仓1，平仓2|
| - orderPrice| string  | 委托价格,order_type等于3（市价）时非必填|
| - orderQty| string  | 委托数量|
| - side| int  | 买1，卖-1|
| - stopCondition| int  | 止损,order_type等于3（市价）时非必填；值域：1（止盈，未启用），2（止损，未启用），3（只减仓，未启用） //前端不传|
| - stopPrice| string  | 止损价格，order_type等于3（市价）时非必填 //条件单时|
| - symbol| string  | 合约名称|


## <span id="2">获取当前订单V2</span>

获取当前订单 V2 版本

#### 新接口的 API域名地址 `https://futures.coinstore.com`  调用支持ApiKey


### HTTP请求:

- GET /api/v2/trade/order/active


> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": [
          {
            "orderStatus": 2,
            "avgPrice": "0",
            "matchQty": "0",
            "matchAmt": "0",
            "accountType": 1,
            "orderPrice": "100",
            "timeInForce": 1,
            "orderType": 1,
            "clOrderId": "1626933997000",
            "positionEffect": 1,
            "marginType": 2,
            "stopCondition": 0,
            "orderSubType": 0,
            "marginLeverage": 10,
            "orderQty": "5000",
            "remainQty": "5000",
            "orderTime": 1626933997312000,
            "stopPrice": "0",
            "initMarginRate": "0.1",
            "side": 1,
            "orderId": "1705963943362561",
            "contractId": 2902104,
            "accountId": 29475591730
          }
    ]
}
```

### 请求参数

|    code    |  type   | required | comment |
| ---------- | ------- | -------- |---------|
|contractId| int    | N      | 合约id    |
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID |


### 响应数据

|       code        | type   | comment                                                           |
| ----------------- |--------|-------------------------------------------------------------------|
| code            | int    | 0                                                                 |     0：成功，其他失败                                               |
| msg         | string |                                                                   |    错误信息                                    |
| data            | list   |                                                                   |
| - accountType| int    | 账户类型 1：user 2:system 3:bonus  4:follow                            |
| - contractId| int    | 合约ID                                                              |
| - orderId| long   | 委托ID                                                              |
| - clOrderId| string | 客户端委托ID                                                           |
| - orderStatus| int    | 委托状态 0：未申报 1:正在申报 2:已申报未成交 3:部分成交 4: 全部成交 5：部分撤单 6： 全部撤单 7:撤单中 8:失效	                         |
| - timeInForce| int    | 成交限制类型：1:GTC, 2:IOC, 3:FOK	                                                           |
| - initMarginRate| string | 保证金率                                                              |
| - marginLeverage| int    | 保证金倍数	                                                            |
| - marginType| int    | 保证金类型，全仓1，逐仓2                                                     |
| - orderType| int    | 委托类型 1：LIMIT  2：条件单限价  3：MARKET  4：条件单市价	                         |
| - orderSubType| int    | 0（默认值），1（被动委托），2（最近价触发条件委托），3（指数触发条件委托），4（标记价触发条件委托）              |
| - positionEffect| int    | 开平标志，开仓1，平仓2                                                      |
| - orderPrice| string | 委托价格：委托价格,order_type等于3（市价）时非必填	                                  |
| - orderQty| string | 委托数量                                                              |
| - side| int    | 方向 1：做多  -1：做空	                                                   |
| - stopCondition| int    | 止损,order_type等于3（市价）时非必填；值域：1（止盈，未启用），2（止损，未启用），3（只减仓，未启用） //前端不传 |
| - stopPrice| string | 止损价格，order_type等于3（市价）时非必填 //条件单时                                 |
| - matchQty| string | 累积成交数量                                                            |
| - matchAmt	| string | 累积成交金额                                                            |
| - avgPrice| String | 成交均价                                                              |
| - remainQty| int    | 剩余数量                                                              |
| - orderTime| long   | 委托时间                                                              |


## <span id="2">获取订单信息V2</span>

获取订单信息 V2 版本

#### 新接口的 API域名地址 `https://futures.coinstore.com`  调用支持ApiKey

### HTTP请求:

- GET /api/v2/trade/order/orderInfo


> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": [
          {
            "accountType": 1,
            "contractId": 100100001,
            "orderId": 1736859343912961,
            "clOrderId": "ZznkeIhHR4yEHmAUF0J6wA#1018",
            "orderStatus": 6,
            "timeInForce": 1,
            "marginLeverage": 0,
            "marginType": 1,
            "orderType": 3,
            "positionEffect": 1,
            "orderPrice": "0",
            "orderQty": "7134",
            "side": 1,
            "stopCondition": "0",
            "stopPrice": "0",
            "matchQty": "0",
            "matchAmt": "0",
            "avgPrice": "0",
            "remainQty": "0",
            "profitAndLoss": "0",
            "fee": "0",
            "feeCurrencyName": "USDT",
            "orderUpdateTime": 1656398147046,
            "orderTime": 1656398147046
          }
    ]
}
```

### 请求参数

|    code    |  type   | required | comment |
| ---------- | ------- | -------- |---------|
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID |


### 响应数据

| code             | type   | comment                                                              |
|------------------|--------|----------------------------------------------------------------------|
| code             | int    | 0                                                                    |     0：成功，其他失败                                               |
| msg              | string |                                                                      |    错误信息                                    |
| data             | list   |                                                                      |
| - accountType    | int    | 账户类型 1：user 2:system 3:bonus  4:follow                               |
| - contractId     | int    | 合约ID                                                                 |
| - orderId        | long   | 委托ID                                                                 |
| - clOrderId      | string | 客户端委托ID                                                              |
| - orderStatus    | int    | 委托状态 0：未申报 1:正在申报 2:已申报未成交 3:部分成交 4: 全部成交 5：部分撤单 6： 全部撤单 7:撤单中 8:失效	 |
| - timeInForce    | int    | 成交限制类型：1:GTC, 2:IOC, 3:FOK	                                          |
| - marginLeverage | int    | 保证金倍数	                                                               |
| - marginType     | int    | 保证金类型，全仓1，逐仓2                                                        |
| - orderType      | int    | 委托类型 1：LIMIT  2：条件单限价  3：MARKET  4：条件单市价	                            |
| - positionEffect | int    | 开平标志，开仓1，平仓2                                                         |
| - orderPrice     | string | 委托价格：委托价格,order_type等于3（市价）时非必填	                                     |
| - orderQty       | string | 委托数量                                                                 |
| - side           | int    | 方向 1：做多  -1：做空	                                                      |
| - stopCondition  | string | 止损,order_type等于3（市价）时非必填；值域：1（止盈，未启用），2（止损，未启用），3（只减仓，未启用） //前端不传    |
| - stopPrice      | string | 止损价格，order_type等于3（市价）时非必填 //条件单时                                    |
| - matchQty       | string | 累积成交数量                                                               |
| - matchAmt	      | string | 累积成交金额                                                               |
| - avgPrice       | String | 成交均价                                                                 |
| - remainQty      | string | 剩余数量                                                                 |
| - profitAndLoss  | String | 盈亏                                                                   |
| - fee            | String | 累计手续费	                                                                 |
| - feeCurrencyName       | String | 交易手续费币种	                                                                 |
| - orderUpdateTime       | long   | 订单状态更新时间：若未成交，则系统返回“订单创建时间”；若已成交，则系统返回最近一笔成交的时间	                                                                 |
| - orderTime      | long   | 委托时间                                                                 |



##  <span id="3">取消委托单</span>
取消委托单

### HTTP请求: 
- POST /api/trade/order/cancel

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId     | int | Y   |合约号 |
| originalOrderId     | long     | Y    |原始委托号sb |

> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": "cancel order success"
}
```

### 响应数据

|       code        |  type  |                         comment                       |
| ----------------- | ------ |    -------------------------------------------------- |
| code              | int    |     0：成功，其他失败                                   |
| msg         | string    |    错误信息                             |
| data            |  string  |  返回数据                                             |

##  <span id="5">一键撤单</span>


### HTTP请求: 
- POST /api/trade/order/cancelAll

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId     | int | N | 不传撤销所有币对订单 |


> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": "cancel all order success"
}
```

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |  0：成功，其他失败                                               |
| msg         | string    |  错误信息                                    |
| data            |  string  |  返回数据                                             |


## <span id="13">批量下单</span>
批量下单
### HTTP请求: 
- POST /api/trade/order/placeBatch

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| orders        | list | Y      |
| - clientOrderId| string|N| 客户端委托ID,如果不传使用uuid |
| - contractId| int|Y| 合约ID|
| - initRate| string|Y| 保证金率,保证金率=1/leverage|
| - marginType| int|Y| 保证金类型，全仓1，逐仓2|
| - orderSubType| int|Y|0（默认值），1（被动委托）|
| - orderType| int|Y| 委托类型，1（限价），3（市价）|
| - positionEffect| int|Y| 开平标志，开仓1，平仓2  平仓单必须传2|
| - orderPrice| string|N| 委托价格,order_type等于3（市价）时非必填|
| - orderQty| string|Y| 委托数量|
| - side| int|Y| 买1，卖-1|
> body
```json
{"orders":[{
  "contractId": 100280034,
  "side": 1,
  "orderType": 1,
  "orderPrice": "1",
  "orderQty": "1",
  "positionEffect": 1,
  "marginType": 1,
  "initRate": 0,
  "orderSubType": 0
},{
  "contractId": 100280034,
  "side": 1,
  "orderType": 1,
  "orderPrice": "1",
  "orderQty": "1",
  "positionEffect": 1,
  "marginType": 1,
  "initRate": 0,
  "orderSubType": 0
}]}
```

> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": {
        "reject": [],
        "succ": [
            [
                "91642986-bccd-4980-9946-326c509f9715",
                "1712125158883589"
            ],
            [
                "eb83d3b4-dd6a-4699-be1c-8a5403a8a884",
                "1712125158883590"
            ]
        ]
    }
}

```

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |   0：成功，其他失败                                     |
| msg         | string    |错误信息                                    |
| data            |  Object  |  返回数据                                                 |
| - reject            | Object[]   |   失败订单  [客户端委托ID,错误code]          |
| - succ            | Object[]   |   成功订单     [客户端委托ID,订单id]         |

## <span id="27">根据订单id批量进行撤单</span>
根据订单id批量进行撤单

### HTTP请求: 
- POST /api/trade/orders/del

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| cancels        | list | Y      |
| - originalOrderId| string|Y| 客户端委托ID,如果不传使用uuid |
| - contractId| int|Y| 合约ID|

> body
```json
{"cancels":[{
  "contractId": 100280034,
  "originalOrderId": 1712125303587721
},{
  "contractId": 100280034,
  "originalOrderId": 1712125303587722
}
]}
```

> 响应

```json
{
    "code": 0,
    "msg": "success",
    "data": {
        "reject": [
                    [
                        "",
                        "1019",
                        "1712125303587721"
                    ],
                    [
                        "",
                        "1019",
                        "1712125303587722"
                    ]
                ],
        "succ": [
            [
                "",
                "1712125303587722"
            ],
            [
                "",
                "1712125303587721"
            ]
        ]
    }
}
```

### 响应数据

|       code        |  type  |                        comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code         | string    |    正常返回信息                                |
| msg         | string    |   错误信息                                   |
| data            | Object   |                                             |
| -succ|   Object[]   |    成功的订单id集合         [客户端委托ID,订单id]                                 |
| -reject  |   Object[]   | 失败的订单id集合        [客户端委托ID,错误code,订单id]                 |




## <span id="3">获取用户全部成交</span>
获取全部成交记录

### HTTP请求: 
- GET /api/future/queryHisMatch


> 响应

```json
{
  "code" : 0,
  "data" : [ {
    "applId" : 0,
    "askFee" : "string",
    "askInitRate" : "string",
    "askMarginType" : 0,
    "askMatchType" : 0,
    "askOrderId" : "string",
    "askPnl" : "string",
    "askPnlType" : 0,
    "askPositionEffect" : 0,
    "askUserId" : 0,
    "bidFee" : "string",
    "bidInitRate" : "string",
    "bidMarginType" : 0,
    "bidMatchType" : 0,
    "bidOrderId" : "string",
    "bidPnl" : "string",
    "bidPnlType" : 0,
    "bidPositionEffect" : 0,
    "bidUserId" : 0,
    "bonusAccountType" : 0,
    "contractId" : 0,
    "execId" : "string",
    "matchAmt" : "string",
    "matchPrice" : "string",
    "matchQty" : "string",
    "matchTime" : 0,
    "side" : 0,
    "takerSide" : 0,
    "updateTime" : 0
  } ],
  "msg" : "string"
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| 合约id|
| endTime| string|N| 结束时间 |
| startTime| string|N| 开始时间|
| pageNum| string|N| 页数|
| pageSize| string|N| 每页数量|
| side| string|N| 方向 -1上一页 1下一页|

### 响应数据

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0：成功，其他失败                                               |
| msg         | string    |                 |    错误信息                                    |
| data | object|    |
| -	applId	|	int	  |	期货ID,2	|
| -	matchTime	|	Long	  |	成交时间	|
| -	contractId	|	Long	  |	交易对ID、合约号	|
| -	execId	|	Long	  |	成交编号	|
| -	bidUserId	|	Long	  |	买方账号ID	|
| -	askUserId	|	Long	  |	卖方账号ID	|
| -	bidOrderId	|	Long	  |	买方委托号	|
| -	askOrderId	|	Long	  |	卖方委托号	|
| -	matchPrice	|	Double	  |	成交价	|
| -	matchQty	|	int	  |	成交数量	|
| -	matchAmt	|	Double	  |	成交金额	|
| -	bidFee	|	Double	  |	买方手续费	|
| -	askFee	|	Double	  |	卖方手续费	|
| -	takerSide	|	int	  |	Taker方向,1: 买方吃单，-1：卖方吃单	|
| -	side	|	int	  |	买卖方向	|
| -	updateTime	|	Long	  |	最近更新时间	|
| -	bidPositionEffect	|	int	  |	买方开平标志	|
| -	askPositionEffect	|	int	  |	卖方开平标志	|
| -	bidMarginType	|	int	  |	买方保证金类型	|
| -	askMarginType	|	int	  |	卖方保证金类型	|
| -	bidInitRate	|	Double	  |	买方初始保证金率	|
| -	askInitRate	|	Double	  |	卖方初始保证金率	|
| -	bidMatchType	|	int	  |	买方成交类型：0普通成交1强平成交2强减成交（破产方）3强减	|
| -	askMatchType	|	int	  |	卖方成交类型：0普通成交1强平成交2强减成交（破产方）3强减	|
| -	bidPnlType	|	int	  |	买方盈亏类型：0正常成交1正常平仓2强平3强减	|
| -	bidPnl	|	Double	  |	买方平仓盈亏	|
| -	askPnlType	|	int	  |	卖方盈亏类型：0正常成交1正常平仓2强平3强减	|
| -	askPnl	|	Double	  |	卖方平仓盈亏	|


## <span id="4">获取用户全部成交V2</span>
获取全部成交记录 V2 版本

#### 新接口的 API域名地址 `https://futures.coinstore.com`  调用支持ApiKey


### HTTP请求:
- GET /api/v2/trade/order/queryHisMatch


> 响应

```json
{
  "code" : 0,
  "data" : [ {
      "accountId": 3431,
      "contractId": 100300144,
      "orderId": 1762426675003393,
      "clOrdId": "c819f4d2282f486d9ed4711327d6381e",
      "matchId": 10763,
      "tradeId": 2177,
      "execSize": 6,
      "matchAmt": -0.672000000000000000,
      "avgPrice": 112.000000000000000000,
      "orderStatus": 4,
      "orderRole": "MAKER",
      "remainingSize": 0,
      "pnl": 0.0,
      "fee": 0.000168,
      "matchTime": 1680781089260
  } ],
  "msg" : "string"
}
```

### 请求参数

|    code    | type   | required | comment            |
| ---------- |--------| -------- |--------------------|
| contractId| int    |N| 合约id               |
| ordId| int      |N| 委托ID               |
| pageNum| string |N| 页数，如果不传参，则默认返回 1 页 |
| pageSize| string |N| 每页数量，默认20          |
| side| string |N| 方向 -1做空 1做多        |

### 响应数据

| code             | type     | comment                                                               |
|------------------|----------|-----------------------------------------------------------------------|
| code             | int      | 0                                                                     |     0：成功，其他失败                                               |
| msg              | string   |                                                                       |    错误信息                                    |
| data             | object   |                                                                       |
| -	accountId	     | 	int	    | 	账户id	                                                                |
| -	contractId	    | 	long	   | 	交易对ID、合约号	                                                           |
| -	orderId	       | 	Long	   | 	委托ID	                                                                |
| -	clOrdId	       | 	string	 | 	客户端委托ID	                                                             |
| -	matchId	       | 	Long	   | 	撮合ID	                                                                |
| -	tradeId	       | 	Long	   | 	成交ID	                                                                |
| -	execSize	      | 	int	    | 	成交数量	                                                                |
| -	matchAmt	      | 	Double	 | 	成交金额	                                                                |
| -	avgPrice	      | 	Double	 | 	成交均价	                                                                |
| -	orderStatus	   | 	int	    | 	委托状态 0：未申报 1:正在申报 2:已申报未成交 3:部分成交 4: 全部成交 5：部分撤单 6： 全部撤单 7:撤单中 8:失效	 |
| -	orderRole	     | 	string	 | 	角色：Taker,Maker		                                                     |
| -	remainingSize	 | 	int	    | 	剩余数量	                                                                |
| -	pnl	           | 	Double	 | 	盈亏	                                                                  |
| -	fee	           | 	Double	 | 	累积手续费	                                                               |
| -	matchTime	     | 	Long	   | 	成交时间	                           |


## <span id="3">用户强减记录</span>
获取全部成交记录

### HTTP请求: 
- GET /api/future/queryFlOrders


> 响应

```json
{
    "data": {
        "list": [
            {
                "timestamp": 1624882627001000,
                "accountType": 1,
                "userId": 100023,
                "contractId": 100280034,
                "uuid": "1703812925489153",
                "applId": 2,
                "side": -1,
                "bankruptcyPrice": 0,
                "closeQty": 799,
                "filledCurrency": 27509.410200000000000000,
                "filledQuantity": 799,
                "canceledQuantity": 0,
                "orderStatus": 4,
                "feeRatio": null,
                "marginType": 1,
                "bonusAccountType": null,
                "lossUserId": 100023,
                "profitUserId": null,
                "execId": null,
                "lossMarginType": 1,
                "profitMarginType": null,
                "closePrice": 34429.800000000000000000,
                "closeAmt": 34429.800000000000000000,
                "lossSubsidy": null,
                "profitSubsidy": null,
                "takerSide": -1
            }
        ]
    },
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| 合约id|
| endTime| string|N| 结束时间 |
| startTime| string|N| 开始时间|
| pageNum| string|N| 页数|
| pageSize| string|N| 每页数量|
| side| string|N| 方向 -1上一页 1下一页|

### 响应数据

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0：成功，其他失败                                               |
| msg         | string    |                 |    错误信息                                    |
| data | object|    |
| -	applId	|	int	  |	期货ID,2	|
| -	timestamp	|	Long	  |	委托时间	|
| -	contractId	|	Long	  |	交易对ID、合约号	|
| -	userId	|	Long	  |	用户ID	|
| -	uuid	|	String	  |	委托编号	|
| -	side	|	int	  |	买卖方向	|
| -	closeQty	|	int	  |	委托量	|
| -	closePrice	|	Double	  |	委托量	|
| -	closeAmt	|	Double	  |	委托量	|
| -	filledCurrency	|	Double	  |	成交金额	|
| -	filledQuantity	|	int	  |	成交量	|
| -	canceledQuantity	|	int	  |	撤单数量	|
| -	orderStatus	|	int	  |	委托状态 4 成交	|
| -	feeRatio	|	Double	  |	手续费	|
| -	marginType	|	int	  |	保证金类型	|
| -	lossUserId	|	int	  |	强减用户（亏损方）	|
| -	profitUserId	|	int	  |	 被强减用户（盈利方）	|
| -	execId	|	int	  |	成交编号	|
| -	lossMarginType	|	int	  |	强减用户（亏损方）保证金类型	|
| -	profitMarginType	|	int	  |	被强减用户（盈利方）保证金类型	|

## <span id="3">用户强平记录</span>
获取全部成交记录

### HTTP请求: 
- GET /api/future/queryFcOrders


> 响应

```json
{
    "data": {
        "list": [
            {
                "timestamp": 1625043178024000,
                "accountType": 1,
                "userId": 100023,
                "contractId": 100280034,
                "uuid": "1703981275414529",
                "applId": 2,
                "side": -1,
                "bankruptcyPrice": 0,
                "closeQty": 1,
                "filledCurrency": 35.108000000000000000,
                "filledQuantity": 1,
                "canceledQuantity": 0,
                "orderStatus": 4,
                "feeRatio": null,
                "marginType": 2,
                "bonusAccountType": null,
                "lossUserId": 100023,
                "profitUserId": null,
                "execId": null,
                "lossMarginType": 2,
                "profitMarginType": null,
                "closePrice": 34789.200000000000000000,
                "closeAmt": 34789.200000000000000000,
                "lossSubsidy": null,
                "profitSubsidy": null,
                "takerSide": -1
            }
        ]
    },
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| 合约id|
| endTime| string|N| 结束时间 |
| startTime| string|N| 开始时间|
| pageNum| string|N| 页数|
| pageSize| string|N| 每页数量|
| side| string|N| 方向 -1上一页 1下一页|

### 响应数据

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0：成功，其他失败                                               |
| msg         | string    |                 |    错误信息                                    |
| data | object|    |
| -	applId	|	int	  |	期货ID,2	|
| -	timestamp	|	Long	  |	委托时间	|
| -	contractId	|	Long	  |	交易对ID、合约号	|
| -	userId	|	Long	  |	用户ID	|
| -	uuid	|	String	  |	委托编号	|
| -	side	|	int	  |	买卖方向	|
| -	closeQty	|	int	  |	委托量	|
| -	closePrice	|	Double	  |	委托量	|
| -	closeAmt	|	Double	  |	委托量	|
| -	filledCurrency	|	Double	  |	成交金额	|
| -	filledQuantity	|	int	  |	成交量	|
| -	canceledQuantity	|	int	  |	撤单数量	|
| -	orderStatus	|	int	  |	委托状态 4 成交	|
| -	feeRatio	|	Double	  |	手续费	|
| -	marginType	|	int	  |	保证金类型	|
| -	lossUserId	|	int	  |	强减用户（亏损方）	|
| -	profitUserId	|	int	  |	 被强减用户（盈利方）	|
| -	execId	|	int	  |	成交编号	|
| -	lossMarginType	|	int	  |	强减用户（亏损方）保证金类型	|
| -	profitMarginType	|	int	  |	被强减用户（盈利方）保证金类型	|




# 行情相关
##  <span id="7">获取历史K线</span>

### HTTP请求: 
- GET /v1/futureQuot/queryCandlestick

>注意：返回结果是按时间正序

> 响应

```json
{
    "success": true,
    "data": {
        "lines": [
            [
                1632794100000,
                "43762",
                "43762",
                "43762",
                "43762",
                "0"
            ]
        ],
        "contractId": 100280034
    }
}
 ```


### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| 合约id|
| range| string|Y| K线类型      1Min : 60000  3Min : 180000  5Min : 300000  15Min : 900000  30Min : 1800000  1Hour : 3600000  2Hour : 7200000  4Hour : 14400000  6Hour : 21600000  12Hour : 43200000  1Day : 86400000  1Week : 604800000 |
| end| string|N| 截止时间 默认当前时间，传第一条记录的时间，可以实现滚动翻页|
| limit| string|N| 查询记录条数  默认 1440 |

### 响应数据: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| msg         | string    |    错误信息                                    |
| data | object|    |
| ├─lines| List[] | [时间戳，开仓价，最高价格，最低价格，收盘价，成交张数]   |

##  <span id="7">查询行情快照</span>

### HTTP请求: 
- GET /v1/futureQuot/querySnapshot

> 响应

```json
{
    "contractId": 100100000,
    "result": {
        "bids": [
            [
                "50999",
                "112"
            ]
        ],
        "asks": [
            [
                "51001",
                "67"
            ]
        ],
        "fr": "0.000033",
        "mp": "56673.9",
        "cid": 100100000,
        "pv": 2158,
        "ph": "51001",
        "tv": "116",
        "tt": "5916.096",
        "pl": "50999",
        "ip": "56673.8",
        "pfr": "0.000033",
        "pcr": "0.00003922",
        "lp": "51001"
    }
}
```


### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| 合约id|


### 响应数据: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| msg         | string    |    错误信息                                    |
| data | object|    |
| ├─cid| Long| 交易对id |
| ├─ip| Double|  指数价|
| ├─mp|  Double| 标记价 |
| ├─lp| Double| 最新价|
| ├─pv| Long| 持仓量 |
| ├─tv| Long|  24小时成交量 |
| ├─tt| Long| 24小时成交额 |
| ├─ph| Double|  24小时最高价 |
| ├─pl| Double|  24小时最低价 |
| ├─pcr |  Double| 24小时涨跌幅率 |
| ├─fr| Double|  资金费率 |
| ├─pfr| Double| 预测资金费率 |
| ├─bids| List | 买盘 [价格，张数] |
| ├─asks| List | 买盘 [价格，张数] |

##  <span id="7">获取最新成交</span>

### HTTP请求: 
- GET /v1/futureQuot/queryTickTrade

>注意：返回结果是按时间正序

> 响应

```json
{
    "success": true,
    "data": {
        "trades": [
            [
                1631779278982000,
                "51000",
                "1",
                1
            ],
            [
                1631779329355000,
                "50000",
                "1",
                -1
            ]
        ]
    }
}
```


### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| 合约id|

### 响应数据: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| msg         | string    |    错误信息                                    |
| data | object|    |
| ├─trades| List[] | [时间戳，成交价，成交张数，成交方向（1多 -1空）]   |

# Websocket行情数据

## 简介

### 接入URL

`
wss://ws-futures.coinstore.com/socket.io/?EIO=3&transport=websocket
`
### 鉴权
注：signature: 签名方式同restful

```
[
    "auth",
    {
        "header": {
            "type": 1001
        },
        "body": {
            "apiKey": "AccessKey",
            "expires": "expires",
            "signature": "signature"
        }
    }
]
```

### 订阅
根据是否需要鉴权之后来进行业务topic的订阅，下面式是订阅的标准格式，在相关ws接口说明里有详细的说明

```
[
    "subscribe",
    {
        "header": {
            "type": 1003
        },
        "body": {
            "topics": [
                {
                    "topic": "indicator",
                    "params": {
                        "symbols": [
                            {
                                "symbol": 0
                            }
                        ]
                    }
                },
                {
                    "topic": "future_snapshot_depth",
                    "params": {
                        "symbols": [
                            {
                                "symbol": 0
                            }
                        ]
                    }
                },
                {
                    "topic": "future_tick",
                    "params": {
                        "symbols": [
                            {
                                "symbol": 0
                            }
                        ]
                    }
                },
                {
                    "topic": "match"
                },
                {
                    "topic": "future_kline",
                    "params": {
                        "symbols": [
                            {
                                "symbol": 0,
                                "ranges": [
                                    "60000"
                                ]
                            }
                        ]
                    }
                }
            ]
        }
    }
]
```
### 订阅 Topic 列表
|topic	|	类型	|	描述	|	需要验签	|
|-----|----|----|----|
|	future_tick	|	SUB	|	获取逐笔成交	|	否	|
|	future_kline	|	SUB	|	获取K线	|	否	|
|	future_snapshot_depth	|	SUB	|	获取行情快照买卖档位	|	否	|
|	indicator	|	SUB	|	获取行情数据	|	否	|
|	match	|	SUB	|	获取当前委托、持仓、资产	|	是	|


## 获取K线

### 订阅topic: future_kline
数据格式中的symbol值为：合约ID

### ranges的取值范围： 

* 1Min = 60000
* 5Min = 300000
* 15Min = 900000
* 30Min = 1800000
* 1Hour = 3600000
* 4Hour = 14400000
* 12Hour = 43200000
* 1Day = 86400000
* 1Week = 604800000


> 消息订阅格式如下所示：

```
{
    "header": {
        "type": 1003
    },
    "body": {
        "topics": [
            {
                "topic": "future_kline",
                "params": {
                    "symbols": [
                        {
                            "symbol": 100,
                            "ranges": [
                                "60000"
                            ]
                        }
                    ]
                }
            }
        ]
    }
}
```
	

### 返回结构

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |合约ID               |
|range      | String    |  范围             |
|lines      |object []  |    [ [${时间戳}, ${开市价格}, ${最高价格}, ${最低价格}, ${闭市价格}, ${成交量}] ]   |

## 获取逐笔成交

#### 订阅topic: future_tick
数据格式中的symbol值为：合约ID

> 消息订阅格式如下所示：

```
{
    "header": {
        "type": 1003
    },
    "body": {
        "topics": [
            {
                "topic": "future_tick",
                "params": {
                    "symbols": [
                        {
                            "symbol": 100
                        }
                    ]
                }
            }
        ]
    }
}
```

### 返回结构

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |合约ID               |
|trades      |object []  |    [ [${时间戳}, ${成交价格}, ${成交数量}, ${方向(1 多； 2 空)}] ] ]   |

## 获取行情快照买卖档位

### 订阅topic: future_snapshot_depth
数据格式中的symbol值为：合约ID

> 消息订阅格式如下所示：

```
{
    "header": {
        "type": 1003
    },
    "body": {
        "topics": [
            {
                "topic": "future_snapshot_depth",
                "params": {
                    "symbols": [
                        {
                            "symbol": 100
                        }
                    ]
                }
            }
        ]
    }
}
```

### 返回结构

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |合约ID               |
|bids      |object []  | 买档位数据列表   [ [${价格}, ${数量}] ] ]   |
|asks      |object []  |  卖档位数据列表  [ [${价格}, ${数量}] ] ]   |

## 获取行情快照基础数据

### 订阅topic: indicator
数据格式中的symbol值为：合约ID

> 消息订阅格式如下所示：

```
{
    "header": {
        "type": 1003
    },
    "body": {
        "topics": [
            {
                "topic": "indicator",
                 "params": {
                       "symbols": [
                           {
                               "symbol": 100
                            }
                        ]
                 }
            }
        ]
    }
}
```

### 返回结构

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| indicators | List<Indicator>|  交易对行情 |
| markets |List<Market>|  行情概况 |
| account | List<Account>|  私有数据  |

**注意：未登录时 accounts为null**
**注意：订阅时 params.symbols 为空，则 indicators 为 null**
**注意：topic默认频率 1s, markets 默认频率 3s，即每3秒有2次推送消息 markets 为null**

#### Indicator

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid| Long| 交易对id |
| ip| BigDecimal| 指数价|
| mp|  BigDecimal| 标记价 |
| lp| BigDecimal| 最新价|
| pv| Long| 持仓量 |
| tv| Long| 24小时成交量 |
| tt| Long| 24小时成交额 |
| ph| BigDecimal| 24小时最高价 |
| pl| BigDecimal| 24小时最低价 |
| pcr |  BigDecimal| 24小时涨跌幅率 |
| fr| BigDecimal| 资金费率 |
| pfr| BigDecimal| 预测资金费率 |

#### Market

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid| Long| 交易对id |
| lp| BigDecimal| 最新价 |
| pcr |  BigDecimal|24小时涨跌幅率 |
| tv |  Long|  24小时成交量 |
| tt| Long| 24小时成交额 |
#### Account

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| aid | Long| 账号id 主账号或者子账号 |
| at| int| 账号类型  1 主账号  10 体验金账号 |
| assets| List<Asset>| 资产信息 |
| posis| List<Posi>| 仓位信息  |

#### Asset

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid | Long| 币种ID |
| avail| BigDecimal| 可用余额 |
| upnl| BigDecimal| 全仓未实现盈亏 |

#### Posi

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| pid | Long| 仓位ID |
| flp| BigDecimal| 强平价格 |
| upnl| BigDecimal| 未实现盈亏 |
| adl| int| ADL等级 |
| mp|  BigDecimal| 标记价 |

**注意：pid为 3012推送仓位的id**

## 获取私有数据
### 订阅topic: match

> 消息订阅格式如下所示：

```
{
    "header": {
        "type": 1003
    },
    "body": {
        "topics": [
            {
                "topic": "match"
            }
        ]
    }
}
```

### 返回结构

#### 资产

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | 消息类型 3002               |
|accountId      |long  | 用户ID  |
|currencyId      |long  |  币种ID  |
|totalBalance      |double  |  总资产  |
|available      |double  |  可用资产  |
|frozenInitMargin      |double  |  委托冻结保证金  |
|initMargin      |double  |  已占用保证金  |
|closeProfitLoss      |double  |  已实现盈亏  |

####持仓

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | 消息类型 3012               |
|accountId      |long  | 用户ID  |
|posis      |object[]  | 持仓列表 |
| ├─contractId      |long  |  合约ID  |
| ├─marginType      |int  |  保证金类型，1全仓2逐仓  |
| ├─initMarginRate      |String  |  初始保证率  |
| ├─maintainMarginRate      |String  |  维持保证金  |
| ├─initMargin      |String  | 初始保证金  |
| ├─extraMargin      |String  |  额外保证金  |
| ├─openAmt      |String  |  开仓金额  |
| ├─openPrice      |String  |  开仓均价  |
| ├─posiQty      |String  |  持仓量，大于0多头，小于0空头  |
| ├─contractUnit      |String  |  合约单位  |
| ├─closeProfitLoss      |String  |  已实现盈亏  |
| ├─liquidationPrice      |double  |  预估强平价  |
| ├─unRealizedProfit      |double  |  未实现盈亏  |

####持仓配置

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | 消息类型 3022               |
|accountId      |long  | 用户ID  |
|positionConfgs      |object[]  | 持仓配置列表 |
| ├─contractId      |long  |  合约ID  |
| ├─leverage      |int  |  杠杆倍数, 0为自动模式  |
| ├─isolated      |boolean  |  保证金类型 默认 false全仓 true逐仓 |

####当前委托

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | 消息类型 3004               |
|accountId      |long  | 用户ID  |
|contractId      |long  |  合约ID  |
|	orderId	|	string	|	委托号	|
|	clientOrderId	|	string	|	客户订单编号	|
|	price	|	string	|	委托价格	|
|	quantity	|	string	|	委托数量	|
|	leftQuantity	|	string	|	剩余数量	|
|	side	|	number	|	买卖方向（1：买，-1：卖）	|
|	placeTimestamp	|	number	|	委托时间	|
|	matchAmt	|	string	|	成交金额	|
|	matchQty	|	string	|	成交张数	|
|	orderType	|	number	|	合约委托类型（1：限价，3：市价）	|
|	positionEffect	|	number	|	开平标志（1：开仓，2：平仓）	|
|	marginType	|	number	|	保证金类型（1、全仓，2：逐仓）	|
|	initMarginRate	|	string	|	初始保证金率	|
|	fcOrderId	|	string	|	强平委托号，空时为强平委托	|
|	markPrice	|	string	|	标记价格	|
|	feeRate	|	string	|	手续费率	|
|	contractUnit	|	string	|	合约单位	|
|	orderStatus	|	int	|	委托状态 0-未申报,1-正在申报,2-已申报未成交,3-部分成交,4-全部成交,5-部分撤单,6-全部撤单,7-撤单中,8-失效,11-缓存高于条件的委托,12-缓存低于条件的委托	|



# 错误码

### 错误码:

| Code                                  | Comment                                |
|---------------------------------------| -------------------------------------- |
| signature-failed 401                  | 主账号冻结子账号登录权限                   |
| signature-failed 401                  | 子账号被删除                   |
| signature-failed 401                  | Api key 错误                   |
| unauthorized 1401                     | 请求ip不在IP白名单中                   |
| unauthorized 1401                     | 签名失败                   |
| unauthorized 1401                     | 用户登录 Token 失效                   |
| signature error 3005                  | 签名生成错误                         |
| symbol not found 3011                 | 该币对没有上线                      |
| trade-limit 3013                      | 您暂时不具备合约交易资质，如有疑问请咨询客服，给您造成不便敬请谅解。                      |
| user-trade-limit 3014                 | 当前子账号暂时不具备合约交易资质                      |
| user-part-trade-limit 3015            | 当前子账号已被限制交易当前合约币对                      |
| duplicate-order 3111                  | 重复订单                             |
| sub-user-operator-business-error 6001 | 子账号不允许参与该业务   |
