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

spot_goods_active: active

spot_goods_url: 'index.html'

contract: 永续合约

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

## 创建APIkey

如需使用API，请先登录网页端，通过【用户中心】-【API管理】创建一个API key，再据此文档详情进行开发和交易。
[您可以点击创建APIKey]: https://www.coinstore.com/#/user/bindAuth/ManagementAPI

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

`https://api.coinstore.com/api`

**WebSocket**

`wss://ws.coinstore.com/s/ws`

为保证API服务的稳定性，建议使用日本AWS云服务器进行访问。如使用中国大陆境内的客户端服务器，连接的稳定性将难以保证。

## 限频规则

**相同ip**

`相同ip每3秒最多300个请求`

**每个用户**

`相同用户3秒最多120个请求`

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
注意：请求参数和请求体不进行任何排序，直接拼接成字符串作为payload。

 ```java
 String payload = “symbol=aaaa88&size=10”；
 ```

实例1：GET请求查询字符串

 ?symbol=aaaa88&size=10

```java
 String payload = “{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；
 ```

实例2 :Post请求体

 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}

```java
 String payload = “symbol=aaaa88&size=10{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}”；
 ```


实例3：混合请求
 
 ?symbol=aaaa88&size=10
 {"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}


```java
 String time = String.valueOf(X-CS-EXPIRES / 30000);
 hmacSha256.init(new SecretKeySpec(Secret_Key.getBytes(), "HmacSHA256"));
 byte[] hash = hmacSha256.doFinal(time.getBytes());
 String key = Hex.toHexString(hash);
```

2、使用签名函数对时间戳计算哈希值

>注意： X-CS-EXPIRES为13位时间戳，需要除以30000获取一个类时间戳，对其进行签名函数计算，获得函数值作为第三步的秘钥(第3步中的变量key的值）




```java
 hmacSha256.reset();
 hmacSha256.init(new SecretKeySpec(key.getBytes(), "HmacSHA256"));
 hash = hmacSha256.doFinal(payload.getBytes());
 String sign= Hex.toHexString(hash);
```

3、使用签名函数对有效字符串计算哈希值
 

>注意： key的值为第2步计算出来的哈希值。

**Python示例**

 https://coinstore-sg-encryption.s3.ap-southeast-1.amazonaws.com/filesUpload/ex1/public/coinstore.py
 
 `推荐python sdk使用3.9版本，使用如果版本低于3.9版本，可能会造成兼容性或者签名计算有误的情况。`

# API接入说明

## <span id="a3">请求格式</span>
所有的API请求都是restful，目前只有两种方法：GET和POST。
- GET请求：所有的参数都在路径参数里
- POST请求: 路径里可以设置参数，参数可以以JSON格式发送在请求主体（body）里，没有参数的需要传{}

一个合法的请求由以下几部分组成：
- 方法请求地址：即访问服务器地址api.coinstore.com，比如https://api.coinstore.com/api/trade/order/place
- 必须和可选参数。
- X-CS-APIKEY： 即用户申请的API Key。
- X-CS-EXPIRES：您发出请求的时间戳。如：1629291143107。
- X-CS-SIGN：签名计算得出的字符串，用于确保签名有效和未被篡改。

**注意：X-CS-APIKEY ，X-CS-EXPIRES ，X-CS-SIGN 三个参数都在请求头中，另外需要设置'Content-Type':'application/json'。**

## <span id="a3">返回格式</span>

所有的接口返回都是JSON格式。在JSON的第一层有3个字段code，msg和data。前两个字段表示请求状态和描述，实际的业务数据在data字段里。

```json
{
    "code": "0",
    "message": "suc",
    "data": // per API response data in nested JSON object
}
```
以下是一个返回格式的样例：

| 字段 | 数据类型  | 描述       |
| ---- | -----  | ---------- |
| code | int | 返回状态 |
| message  | string | 状态或错误描述 |
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

如果失败，response message 带有错误描述信息, 对应的状态码描述如下：

| 状态码 | 说明                             | 备注                       |
| ------ | -------------------------------- | -------------------------- |
| 0      | 成功                             | code=0 成功， code >0 失败 |


# 基础信息

## <span id="1">现货币种币对信息</span>

获取现货币种币对信息

### HTTP请求:
- POST /v2/public/config/spot/symbols

### 请求参数

| param | type | required | comment |
|-------|------|----------|---------|
|   symbolCodes    | 数组   | 非必需      | 币对Code  |
|   symbolIds    | 数组   | 非必需      | 币对ID    |

> 请求

```json
{
  "symbolCodes":["BTCUSDT"],
  "symbolIds":[10]
}
```
### 响应数据

| code                | type      | comment        |
|---------------------|-----------|----------------|
| code                | int       | 0：成功，其他失败      |
| message             | String    | 错误信息           |
| data                | Object [] | 业务数据           |
| - symbolId          | Long      | 币对ID           |
| - symbolCode        | String    | 币对Code         |
| - tradeCurrencyCode | String    | 交易货币币种           |
| - quoteCurrencyCode | String    | 计价货币币种             |
| - openTrade         | boolean   |是否开放交易：true是，false否 |
| - onLineTime        | Long      |上线日期：Unix时间戳的毫秒数格式，如 1597026383085 |
| - tickSz            | Integer   |下单价格精度（tickSz） |
| - lotSz             | Integer   |下单数量精度（lotSz）|
| - minLmtPr          | String   |限价最小交易最小价（minLmtPr） |
| - minLmtSz          | String   |限价最小交易数量（minLmtSz） |
| - minMktVa         | String   |市价最小交易最小额（minMktVa） |
| - minMktSz         | String   |市价最小交易数量（minMktSz） |
| - makerFee         | String   |Maker 手续费 |
| - takerFee         | String   |Taker 手续费 |

> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests
url = "https://api.coinstore.com/api/v2/public/config/spot/symbols"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'exch-language': 'en_US',
 'Content-Type': 'application/json',
 'Accept': '*/*',
 # 'Host': 'https://api.coinstore.com',
 'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```

> 响应

```json
{
    "code": "0",
    "message": "Success",
    "data": [
        {
            "symbolId": 1,
            "symbolCode": "BTCUSDT",
            "tradeCurrencyCode": "btc",
            "quoteCurrencyCode": "usdt",
            "openTrade": true,
            "onLineTime": 1609813531019,
            "tickSz": 0,
            "lotSz": 4,
            "minLmtPr": "0.0002",
            "minLmtSz": "0.2",
            "minMktVa": "0.1",
            "minMktSz": "0.1",
            "makerFee": "0.006",
            "takerFee": "0.003"
        }
    ]
}
```
## <span id="1">查询币种信息</span>

获取币种详细信息

### HTTP请求:
- GET /fi/v1/common/currency

### 请求参数

| param | type | required | comment |
|-------|------|----------|---------|
|   currencyCode    | string   | Y        | 币对Code  |

> 请求

```json
{
  "currencyCode":"ETH"
}
```

### 响应数据

| code                | type    | comment       |
|---------------------|---------|---------------|
| code                | int     | 0：成功，其他失败     |
| message             | String  | 错误信息          |
| data                |         | 业务数据          |
|-	name |string| 币种名称|
|-   code |string| 币种代码|
|-   showPrecision |string| 币种精度|
|-   chainDataList |Object []| 链信息|
|--		currencyCode |string| 币种code|
|--		chainName |string| 链名称|
|--    chainProtocol  |string|  链协议|
|--		contractAddress |string|币种合约地址|
|--		depositOpen |int|是否开放充值，0关闭充值，1开放充值|
|--		withdrawOpen |int|是否开启提现，0关闭提现，1开启提现|
|--		withdrawFee |string|提币手续费|
|--		withdrawMin |string|最小提币数量|
|--		withdrawMax |string|最大提币数量|
|--		depositMin |string|最小充币数量|
|--		withdrawFeePlatform |string|提币平台手续费比例|
|--		burningState |int|是否为燃烧币 是1 否0|
|--		withdrawFeeOnChain |string|提币链上燃烧比例|
|--		tagType |int|是否需要标签 0不需要 1需要 2可选|
|--		withdrawFeeMin |string|提现最少手续费|

> python

```python
import hashlib
import hmac
import json
import math
import time
import requests

# api查询币种信息

url = "https://api.coinstore.com/api/fi/v1/common/currency?currencyCode=ETH"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = "currencyCode=ETH"
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()

headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'Content-Type': 'application/json'
}
response = requests.request("GET", url, headers=headers)
print(response.text)
```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "name": "eth",
    "code": "eth",
    "showPrecision": 6,
    "chainDataList": [
      {
        "currencyCode": "eth",
        "chainName": "eth",
        "contractAddress": "23412374127384ghgdwq",
        "depositOpen": 0,
        "withdrawOpen": 1,
        "withdrawFee": "100.00000000",
        "withdrawMin": "1.00000000",
        "withdrawMax": "0.00000000",
        "depositMin": "1.00000000",
        "withdrawFeePlatform": "0.00000000",
        "burningState": 0,
        "withdrawFeeOnChain": "0.00000000",
        "tagType": 0,
        "withdrawFeeMin": "0.00000000"
      }
    ]
  }
}
```

# 账户相关


## <span id="1">资产余额</span>

获取用户资产余额

### HTTP请求: 
- POST /spot/accountList

> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests
url = "https://api.coinstore.com/api/spot/accountList"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'exch-language': 'en_US',
 'Content-Type': 'application/json',
 'Accept': '*/*',
 # 'Host': 'https://api.coinstore.com',
 'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```

> 响应 

```json
{
    "data": [
        {
            "uid": 315,
            "accountId": 1134971,
            "currency": "USDT",
            "balance": "0.05001",
            "type": 4,
            "typeName": "FROZEN"
        },
        {
            "uid": 315,
            "accountId": 1134971,
            "currency": "USDT",
            "balance": "13297.94999",
            "type": 1,
            "typeName": "AVAILABLE"
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  |  |  |  |

### 响应数据

|       code        |  type  |                  comment                       |
| ---- | ----- |---------- |
| code | int      |   0：成功，其他失败         |
| message | String  | 错误信息 |
| data | Object []  |  业务数据 |
|- userId | Long  | 用户id |
|- accountId | Long  | 账户id |
|- currencyId | int  | 币种id |
|- balance | Decimal  | 金额 |
|- type | Decimal |账户状态 1:可用 4:冻结 |

# 资金相关
## <span id="2">获取充币地址</span>
获取充币地址信息

### HTTP请求:
- POST /fi/v3/asset/deposit/do

### 请求参数

|  code |  type  |  required  |comment|
| ---- | ----- |---------- |-----|
|currencyCode	|string|	Y|	币种code|
|chain	|string|	Y|	链名称|

> 请求

```json
{
  "currencyCode": "USDTTRX",
  "chain": "TRX"
}
```

### 响应数据

|       code        |  type  |                  comment                       |
| ---- | ----- |---------- |
|code	|int|	0：成功，其他失败|
|message	|string|	错误信息|
|data	|object|	|
|-	address	|string|	地址|
|-	tag	|string|	标签|
|-	depositMin	|string|	最小充币数量|

> python

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/fi/v3/asset/deposit/do"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "currencyCode": "USDTTRX",
    "chain": "TRX"
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'exch-language': 'en_US',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)
```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "address": "TW4……",
    "tag": "",
    "depositMin": 0.00010000
  }
}
```

## <span id="2">获取充币记录</span>
获取用户充币记录

### HTTP请求:
- POST /fi/v3/asset/deposit/record/list

### 请求参数

|  code |  type  | required |comment|
| ---- | ----- |----------|-----|
|currencyCode |string| N        | 币种对|
|startDate |string| N        | 起始时间 日期型字符串 格式"yyyy-MM-dd HH:mm:ss"等|
|endDate |string| N        | 结束时间 日期型字符串 格式"yyyy-MM-dd HH:mm:ss"等|
|fromId |long| N        | 从id开始查询，缺省从头开始|
|limit |int| N        |  条数 默认500，最大1000|
|externalId |string| N        | 外部id|
|label |string| N        | 备注标签|


> 请求

```json
{
  "currencyCode": "",
  "startDate": "2024-01-01",
  "endDate": "2024-03-08",
  "fromId": "",
  "limit": "10",
  "externalId": "",
  "label": ""
}

```

### 响应数据

|       code        |  type  |                  comment                      |
| ---- | ----- |--------- |
|code	|type|	comment|
|code	|int|	0：成功，其他失败|
|message	|string|	错误信息|
|data	| | |
|	externalId |string| 外部id|
|	label |string| 备注标签|
|	withdrawList |Object[]|	业务数据|
|--	 id	|long|	ID|
|--	 symbol	|string|	币种|
|--	 amount	|string|	金额|
|--	 fee	|string|	手续费|
|--	 createdAt	|string|	时间|
|--	 status	|int|	提现状态: 0 未审核，1 审核通过，2 审核拒绝，3 支付中已经打币，4 支付失败，5 已完成，6 已撤销|
|--	 explorer	|string|	区块链浏览器|
|--	 txId	|string|	交易hash|
|--	 addressTo	|string|	提币地址|
|--	 chainName	|string|	链名称|
|--	 chainProtocol	|string|	链协议|
|--	 internalType	|int|	站内提币类型 0：非站内 1：站内|

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/fi/v3/asset/deposit/record/list"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "currencyCode": "USDT",
    "startDate": "2024-03-01",
    "endDate": "2024-03-08",
    "fromId": "",
    "limit": "10",
    "externalId": "",
    "label": ""
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'exch-language': 'en_US',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)

```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "depositList": [
      {
        "id": 38948,
        "symbol": "doge",
        "chainName": "doge",
        "chainProtocol": "doge",
        "amount": "10.61",
        "createdAt": "2024-02-27 17:36:38",
        "status": 1,
        "explorer": "19ceb820c143ceae503cf6ce5a0c9064721f438099529d041026e062e99a0bdc8",
        "txId": "9ceb820c143ceae503cf6ce5a0c9064721f438099529d041026e062e99a0bdc8",
        "internalType": 0,
        "lockAccount": false,
        "noDepositMsg": null
      }
    ],
    "externalId": null,
    "label": null
  }
}
```

## <span id="2">获取提币记录</span>
获取用户提币记录

### HTTP请求:
- POST /fi/v3/asset/withdraw/record/list

### 请求参数

|  code |  type  | required   |comment|
| ---- | ----- |------------|-----|
|param	|type| 	required	 |comment|
|currencyCode |string| N          | 币种对|
|startDate |string| N          | 起始时间|
|endDate |string| N          | 结束时间|
|fromId |long| N          |从id开始查询，缺省从头开始|
|limit |int| N          |条数 默认500，最大1000|
|externalId |string| N          | 外部id|
|label |sring| N          | 标签|

> 请求

```json
{
  "currencyCode": "",
  "startDate": "2024-03-01",
  "endDate": "2024-03-08",
  "fromId": "",
  "limit": "10",
  "externalId": "",
  "label": ""
}

```

### 响应数据

|       code        |  type  |                  comment                      |
| ---- | ----- |--------- |
|code	|type|	comment|
|code	|int|	0：成功，其他失败|
|message	|String|	错误信息|
|data|	|  |
|	externalId |string| 外部id|
|	label |string| 备注标签|
|	depositList |Object []|	业务数据|
|--	 id	|long|	ID|
|--	 symbol	|string|	币种|
|--	 chainName	|string|	链名|
|--	 chainProtocol	|string|	链协议|
|--	 amount	|string|	数量|
|--	 createdAt	|string|	时间|
|--	 status	|int|	充值状态: 0 未确认，1 已完成，2 异常，3审核中，4驳回|
|--	 explorer	|string|	区块浏览器|
|--	 txId	|string|	交易hash|
|--	 internalType	|string|	站内充币类型 0：非站内 1：站内|
|--	 lockAccount	|boolean|	是否锁定|

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/fi/v3/asset/withdraw/record/list"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "currencyCode": "",
    "startDate": "2024-01-01",
    "endDate": "2024-03-08",
    "fromId": "",
    "limit": "10",
    "externalId": "",
    "label": ""
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'exch-language': 'en_US',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)

```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "withdrawList": [
      {
        "id": 4886,
        "symbol": "DOGE",
        "amount": "0.3",
        "fee": "0",
        "createdAt": "2024-02-27 18:20:31",
        "status": 5,
        "showStatus": 2,
        "explorer": "https://eth.btc.com/txinfo/",
        "txId": "7b06acd90f839914f2676693927d994c264a0561893d3a5461a2a910d318acba",
        "addressTo": "DFXm3XC49n3SBNqELip9LZHAo1nBaiDbdG",
        "tag": null,
        "chainName": "DOGE",
        "chainProtocol": "DOGE",
        "internalType": 0
      }
    ],
    "externalId": null,
    "label": null
  }
}
```

## <span id="2">提币</span>
用户提币。普通子账户不支持提币;API只能提币到免认证地址/账户上，通过 WEB/APP 可以设置免认证地址。

### HTTP请求:
- POST /fi/v3/asset/doWithdraw

### 请求参数

|  code |  type  | required   |comment|
| ---- | ----- |------------|-----|
|param	|type|	required|	comment|
|currencyCode |string| Y| 划转币种|
|amount |string| Y| 划转金额|
|address |string| Y| 提币地址|
|tag |string| N| 标签 如需要|
|chainType |string| Y| 链协议 币种详情中chainProtocol 例如:trc20 bnbbsc erc20 sol |


> 请求

```json
{
  "currencyCode": "USDT",
  "amount": "1000",
  "address": "TU4vEruvZwLLkSfV9bNw12EJTPvNr7Pvaa",
  "tag": "",
  "chainType": "trc20"
}
```

### 响应数据

|       code        |  type  |                  comment                      |
| ---- | ----- |--------- |
|code	|type|	comment|
|code	|int|	0：成功，其他失败|
|message	|String|	错误信息|
|data|||	
|--	 id	|Long|	提币ID|

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/fi/v3/asset/doWithdraw"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "currencyCode": "USDT",
    "amount": "1000",
    "address": "TU4vEruvZwLLkSfV9bNw12EJTPvNr7Pvaa",
    "tag": "698347",
    "chainType": "TRX"
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'Content-Type': 'application/json'}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)

```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "id": 4886
  }
}
```

## <span id="2">撤销提币</span>
用户撤销提币

### HTTP请求:
- POST /fi/v3/asset/cancelWithdraw

### 请求参数

|  code |  type  | required   |comment|
| ---- | ----- |------------|-----|
|withdrawId |long| Y| 提现ID|


> 请求

```json
{
  "withdrawId": 4886
}
```

### 响应数据

|       code        | type   |                  comment                      |
| ---- |--------|--------- |
|code	| type   |	comment|
|code	| int    |	0：成功，其他失败|
|message	| String |	错误信息|
|data	|        ||
|--	 id	| Long   |	提币ID|

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/fi/v3/asset/cancelWithdraw"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "withdrawId": 4886
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'exch-language': 'en_US',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)

```
> 响应

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "id": 4886
  }
}
```

## <span id="2">划转资产</span>
划转用户资产

#### 目前支持合约<一>现货划转, API域名地址 `https://futures.api.coinstore.com/api`  调用支持ApiKey


### HTTP请求:
- POST /v1/future/transfer

### 请求参数

|  code |  type  | required   | comment                                                                                  |
| ---- | ----- |------------|------------------------------------------------------------------------------------------|
|param	|type|	required| 	comment                                                                                 |
|type |string| Y| 0, 类型 划转类型 0：账户内划转（适用于母子账户APIKey） <br/>1：母账户转子账户(仅适用于母账户APIKey) <br/>2：子账户转母账户(仅适用于母账户APIKey) <br/>默认是0 |
|currencyCode |string| Y| "USDT", 划转币种，如 USDT                                                                      |
|amount |string| Y | 划转数量                                                                                     |
|from |int| Y | 转出账户 1：币币账户 2：合约账户                                                                       |
|to |int| Y | 转入账户 1：币币账户 2：合约账户                                                                       |
|subAccount |string| N| 子账户登录名 当type为1/2时，该字段必填                                                                  |


> 请求

```json
{
  "type": "0",
  "currencyCode": "USDT",
  "amount": "1",
  "from": "1",
  "to": "2",
  "subAccount": ""
}
```

### 响应数据

|       code        | type   |                  comment                      |
| ---- |--------|--------- |
|code	|type|	comment|
|code	|int|	0：成功，其他失败|
|message	|string|	错误信息|

```python
python

import hashlib
import hmac
import json
import math
import time
import requests

url = "https://api.coinstore.com/api/v1/future/transfer"
api_key = b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "type": "0",
    "currencyCode": "USDT",
    "amount": "1",
    "from": "1",
    "to": "2",
    "subAccount": ""
})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
    'X-CS-APIKEY': api_key,
    'X-CS-SIGN': signature,
    'X-CS-EXPIRES': str(expires),
    'exch-language': 'en_US',
    'Content-Type': 'application/json',
    'Accept': '*/*',
    'Connection': 'keep-alive'}
response = requests.request("POST", url, headers=headers, data=payload, verify=False)
print(response.text)

```
> 响应

```json
{
  "code": "0",
  "message": "Success"
}
```
> 响应数据:失败参数示例:

```json
{
  "code": 101040502,
  "msg": "request field missing",
  "data": null
}
```

### 其他响应code说明

|       code       |  remark                      |
| ---- | ----- |
| 200| 划转成功 |
| 102150400| 划转数量不正确 |
| 101040502| 可能币种名称、自定义ID、划转类型不正确、划转币种不支持 |
| 102150808| 可能余额不足、重复订单 |
| 102150400| 失败 |



# 订单相关

## <span id="2">获取当前订单</span>
获取当前订单

### HTTP请求: 
- GET /trade/order/active

> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests 
url = "https://api.coinstore.com/api/trade/order/active"
api_key=b'your api_key'
secret_key = b'your secret_key' 
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = ''
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {'X-CS-APIKEY':api_key,'X-CS-SIGN':signature,'X-CS-EXPIRES':str(expires),'Content-Type':'application/json'}
response = requests.request("GET", url, headers=headers, data=payload)
print(response.text)
```

> 响应

```json
{
    "data": [
        {
            "symbol": "LUFFYUSDT",
            "baseCurrency": "LUFFY",
            "quoteCurrency": "USDT",
            "timestamp": 1642402062196,
            "side": "BUY",
            "timeInForce": "GTC",
            "accountId": 1138204,
            "ordPrice": 9.1E-10,
            "cumAmt": "0",
            "cumQty": "0",
            "leavesQty": "0",
            "clOrdId": "b1b2ea5e00a84b888d419cb73f8eb203",
            "ordAmt": "0",
            "ordQty": "1",
            "ordId": "1722183748419690",
            "ordStatus": "SUBMITTED",
            "ordType": "LIMIT"
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  |  |  |  |


### 响应数据

|       code        |  type  |                         comment                       |
| ----------------- | ------ |------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |         错误信息                                    |
| data            |  list  |    |
| ├─ ordId            | long      订单id            |
| ├─ clOrdId            | string     |   客户端订单id            |
| ├─symbol | string |   币对 |
| ├─baseCurrency| string |   base币|
| ├─quoteCurrency| string |   quot币|
| ├─ordPrice | string |    订单价格|
| ├─ordQty| string |   订单数量|
| ├─ordAmt| string |   订单金额|
| ├─side | string |    BUY,SELL|
| ├─cumAmt| string | 成交金额  |
| ├─cumQty| string |  成交数量 |
| ├─leavesQty| string |  剩余数量  |
| ├─ordStatus | string |  订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─ordType | string |   MARKET,LIMIT,POST_ONLY|
| ├─timeInForce| string | GTC,IOC,FOK |
| ├─timestamp| long | 时间戳 |


## <span id="3">获取当前订单V2</span>

获取当前订单 v2 版本

#### 新接口的 API域名地址 `https://api.coinstore.com`  调用支持ApiKey

### HTTP请求:
- GET /api/v2/trade/order/active

> python 

```python
import hashlib
import hmac
import math
import time
import requests
url = "https://api.coinstore.com/api/v2/trade/order/active"
api_key=b'your api_key'
secret_key = b'your secret_key'  
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = 'symbol=btcusdt'
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {'X-CS-APIKEY':api_key,'X-CS-SIGN':signature,'X-CS-EXPIRES':str(expires),'Content-Type':'application/json'}
response = requests.request("GET", url, headers=headers, data=payload)
print(response.text)
```

> 响应

```json
{
    "data": [
        {
          "baseCurrency": "A123",
          "quoteCurrency": "USDT",
          "side": "BUY",
          "cumQty": "0",
          "ordId": 1758065956225025,
          "clOrdId": "tLCGVA4g19zuEBwsITXi9g3U624Al0Bw",
          "ordType": "LIMIT",
          "ordQty": "10",
          "cumAmt": "0",
          "accountId": 1134912,
          "timeInForce": "GTC",
          "ordPrice": "100",
          "leavesQty": "10",
          "avgPrice": "0",
          "ordStatus": "SUBMITTED",
          "symbol": "A123USDT",
          "timestamp": 1676622290389
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    | type   | required | comment |
| ---------- |--------|--------|---------|
|symbol | string | N      | 交易对     |
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID  |


### 响应数据

|       code    |  type  | comment                                 |
| ------------- | ------ |-----------------------------------------|
| code          | int    | 0：成功，其他失败                               |
| message       | string    | 错误信息                                    |
| data          |  list  |                                         |
| ├─baseCurrency| string | base币                                   |
| ├─quoteCurrency| string | quot币                                   |
| ├─timeInForce| string | GTC,IOC,FOK                             |
| ├─ordStatus | string | 订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 成交均价	                                   |
| ├─ordPrice | string | 订单价格                                    |
| ├─leavesQty| string | 剩余数量                                    |
| ├─ordType | string | 委托类型 MARKET,LIMIT,POST_ONLY             |
| ├─ordQty| string | 订单数量                                    |
| ├─cumAmt| string | 成交金额                                    |
| ├─side | string | BUY,SELL                                |
| ├─cumQty| string | 成交数量                                    |
| ├─ ordId      | long  |     订单id                                |
| ├─ clOrdId      | string     | 客户端订单id                                 |
| ├─symbol | string | 币对                                      |
| ├─timestamp| long | 时间戳                                     |

## <span id="3">获取用户最新成交</span>
获取全部成交记录

### HTTP请求: 
- GET /trade/match/accountMatches

> python 

```python
import hashlib
import hmac
import math
import time
import requests
url = "https://api.coinstore.com/api/trade/match/accountMatches?symbol=tipusdt"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time()* 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = 'symbol=tipusdt'
payload=payload.encode("utf-8")
signature = hmac.new(key,payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'Content-Type': 'application/json'
}
response = requests.request("GET", url, headers=headers)
print(response.text)
```




> 响应

```json
{
    "data": [
        {
            "id": 3984987,
            "remainingQty": 0E-18,
            "matchRole": 1,
            "feeCurrencyId": 44,
            "acturalFeeRate": 0.002000000000000000,
            "role": 1,
            "accountId": 1138204,
            "instrumentId": 15,
            "baseCurrencyId": 44,
            "quoteCurrencyId": 30,
            "execQty": 40.900000000000000000,
            "orderState": 50,
            "matchId": 258338866,
            "orderId": 1717384395096065,
            "side": 1,
            "execAmt": 8.887161000000000000,
            "selfDealingQty": 0E-18,
            "tradeId": 11523732,
            "fee": 0.081800000000000000,
            "matchTime": 1637825389,
            "seq": null
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    |  type   | required | comment                |
| ---------- | ------- | -------- |------------------------|
|symbol | string | Y | 交易对                    |
|ordId| int | N | 委托ID                   |
|pageNum| int | N | 当前页码，如果不传参，则默认返回 第 1 页 |
|pageSize| int | N | 每页数量，如果不传参，则默认返回 20 条  |
|side| int | N | 方向 -1卖出 1买入            |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |---------------------------------- |
| code            | int    |  0：成功，其他失败                                               |
| message         | string    |   错误信息                                    |
| data | object|   |
| ├─accountId| long| 账户id |
| ├─instrumentId| int | 币对id |
| ├─baseCurrencyId| int | 基础货币 BTC |
| ├─quoteCurrencyId| int | 计价货币USDT |
| ├─matchRole| int | 撮合角色 TAKER(1),MAKER(-1) |
| ├─feeCurrencyId| int | 手续费币种id |
| ├─role| String | 角色 TAKER,MAKER |
| ├─execQty| String | 成交数量 |
| ├─orderState| int | 订单状态 |
| ├─matchId| long | 撮合id |
| ├─orderId| long | 订单id |
| ├─side| int | 订单方向 | BUY(1), SELL(-1) |
| ├─execAmt| String | 成交额 |
| ├─selfDealingQty| String | 自成交额 |
| ├─tradeId| long | 成交id |
| ├─fee| String | 手续费 |
| ├─matchTime| long  | 撮合时间 |
| ├─remainingQty| String | 剩余数量 |


##  <span id="4">取消委托单</span>
取消委托单

### HTTP请求: 
- POST /trade/order/cancel

> 请求体

```json
{
  "ordId": 1722183748419690,
  "symbol":"LUFFYUSDT"
}
```

> python 

```python
import hashlib
import hmac
import math
import time
import requests
import json
url = "https://api.coinstore.com/api/trade/order/cancel"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({"symbol":"btcusdt","orderId":1782425964251297})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {'X-CS-APIKEY':api_key,'X-CS-SIGN':signature,'X-CS-EXPIRES':str(expires),'Content-Type':'application/json'}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```


> 响应

```json
{
    "code": 0,
    "data": {
        "clientOrderId": "b1b2ea5e00a84b888d419cb73f8eb203",
        "state": "CANCELED",
        "ordId": 1722183748419690
    }
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | true | |
| ordId     | long     | true  |  |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |--------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |   错误信息                                    |
| data            |    |   返回数据                    |
| ├─state| string|   |
| ├─ordId| long |   |

##  <span id="5">一键撤单</span>


### HTTP请求: 
- POST /trade/order/cancelAll

> 请求体

```json
{
  "symbol":"LUFFYUSDT"
}
```


> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests
url = "https://api.coinstore.com/api/trade/order/cancelAll"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
  "symbol": "BTCUSDT"
 })
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'exch-language': 'en_US',
 'Content-Type': 'application/json',
 'Accept': '*/*',
 # 'Host': 'https://api.coinstore.com',
 'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```



> 响应

```json
{
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | true| 撤销指定交易对所有订单 |

### 响应数据

|       code        |  type  |                      comment                       |
| ----------------- | ------ |--------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |


## <span id="6">创建订单</span>
创建订单

#### 普通用户的账号同一时间只允许持有 50 笔活动委托，做市账号不受此限制。如果您是做市商但有账号未添加做市账号权限，请联系 Coinstore 交付部门。

> 请求体

```json
{
	"symbol": "LUFFYUSDT",
	"side": "BUY",
	"ordType": "LIMIT",
	"ordPrice": 9.2e-10,
	"ordQty": "12323231243",
	"timestamp": 1642407805168
}
```

> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests
url = "https://api.coinstore.com/api/trade/order/place"
api_key=b'your api_key'
secret_key = b'your secret key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "ordPrice": "30000",
 "ordQty": "1",
 # "clOrdId": "8vdpfHC0LmhojVIffOlkBc9bV9992",
 "symbol": "BTCUSDT",
 "side": "BUY",
 "ordType": "LIMIT"
 })
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'exch-language': 'en_US',
 'Content-Type': 'application/json',
 'Accept': '*/*',
 # 'Host': 'https://api.coinstore.com',
 'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```

> 响应

```json
{
    "code": "0",
    "data": {
        "order_id":11594964764657880
    }
}
``` 

### HTTP请求: 
- POST /trade/order/place

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| clOrdId | string | false    |   客户端订单ID。客户自定义的唯一订单ID。 如果未发送，则自动生成  |
| symbol        | string | true    |  |
| side   | string    | true    |   BUY,SELL    |
| ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| timeInForce    | string     | false    | defaultValue = "GTC"   |
| ordPrice    | long     | false    | 限价单必选 |
| ordQty    | long     | false    | 限价单必选，市价卖单必选 |
| ordAmt    | long     | false    | 市价买单必选 |
| timestamp    | long     | true    |时间戳  |                                                                    


                                

### 响应数据

|       code        |  type  |                    comment                       |
| ----------------- | ------ |------------------------------- |
| code            | int     |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |
| data            |     |   返回数据                                                 |
| ├─ ordId            | long   |  订单id                                                 |

## <span id="13">批量下单</span>
批量下单

#### 普通用户的账号同一时间只允许持有 50 笔活动委托，做市账号不受此限制。如果您是做市商但有账号未添加做市账号权限，请联系 Coinstore 交付部门。

> 请求体

```json
{
	"symbol": "LUFFYUSDT",
	"orders": [
		{
			"side": "BUY",
			"ordType": "LIMIT",
			"ordPrice": 9.2e-10,
			"ordQty": "1"
		},
		{
			"side": "BUY",
			"ordType": "LIMIT",
			"ordPrice": 9.2e-10,
			"ordQty": "1"
		}
	],
	"timestamp": 1642574848089
}
```

> python 

```python
import hashlib
import hmac
import json
import math
import time
import requests
url = "https://api.coinstore.com/api/trade/order/placeBatch"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({
    "symbol":"BTCUSDT",
 "orders":[
        {
            "side":"BUY",
 "ordType":"LIMIT",
 "ordPrice":28000,
 "ordQty":"2"
 },
 {
            "side":"BUY",
 "ordType":"LIMIT",
 "ordPrice":30000,
 "ordQty":"1"
 }
    ],
 "timestamp":expires}
)
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'exch-language': 'en_US',
 'Content-Type': 'application/json',
 'Accept': '*/*',
 # 'Host': 'https://api.coinstore.com',
 'Connection': 'keep-alive'
}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```


> 响应

```json
{
    "code": 0,
    "data": [
        {
            "ordId": 1722364585836585,
            "clOrdId": "c09e68b49a724c8a95ef76526bf6bc05"
        },
        {
            "ordId": 1722364585836586,
            "clOrdId": "d0344830639d4f6d94be4110bf98c759"
        }
    ]
}
```

### HTTP请求: 
- POST /trade/order/placeBatch

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol        | string | true    |  |
| timestamp    | long     | true    |  |
| orders        | list | true    |  |
| ├─ clOrdId | string | false    |   客户端订单ID。客户自定义的唯一订单ID。 如果未发送，则自动生成  |
| ├─ side   | string    | true    |   BUY,SELL    |
| ├─ ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| ├─ timeInForce    | string     | false    | defaultValue = "GTC"   |
| ├─ ordPrice    | long     | false    | 限价单必选 |
| ├─ ordQty    | long     | false    | 限价单必选，市价卖单必选 |
| ├─ ordAmt    | long     | false    | 市价买单必选 |


### 响应数据

|       code        |  type  |                     comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |    0：成功，其他失败                                               |
| message         | string        |    错误信息                                    |
| data            |  list    |   返回数据                                                 |
| ├─ ordId            | long     |   订单id            |
| ├─ clOrdId            | string     |   客户端订单id            |
| ├─errno| int|   |  |
| ├─errMsg| string| | |

## <span id="27">根据订单id批量进行撤单</span>
根据订单id批量进行撤单

### HTTP请求: 
- POST trade/order/cancelBatch

> 请求体

```json
{
  "orderIds": [1722364585836586,1722364585836585],
  "symbol":"LUFFYUSDT"
}
```

> python 

```python
import hashlib
import hmac
import math
import time
import requests
import json
url = "https://api.coinstore.com/api/trade/order/cancelBatch"
api_key=b'your api_key'
secret_key = b'secret_key'
expires = int(time.time() * 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = json.dumps({"symbol":"btcusdt","orderId":[1782428898165698]})
payload = payload.encode("utf-8")
signature = hmac.new(key, payload, hashlib.sha256).hexdigest()
headers = {'X-CS-APIKEY':api_key,'X-CS-SIGN':signature,'X-CS-EXPIRES':str(expires),'Content-Type':'application/json'}
response = requests.request("POST", url, headers=headers, data=payload)
print(response.text)
```

> 响应

```json
{
    "data": {
        "success": [
            1722364585836586,
            1722364585836585
        ],
        "reject": []
    },
    "code": 0
}
```

### 请求参数

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | Y |  "BTCUSDT"|
|orderIds| LIst| Y |[1,2,3]  |



### 响应数据

|       code        |  type  |          comment                       |
| ----------------- | ------ |---------------------------------------- |
| code         | string    |   0：成功，其他失败                               |
| message         | string    |    错误信息                                   |
| data            | Object   |                                      |
| --success|   List   |     成功的订单id集合                                         |
| --reject  |    List     |  失败的订单id集合                      |


## <span id="14">获取订单信息</span>
获取订单信息

> python 

```python
import hashlib
import hmac
import math
import time
import requests
url = "https://api.coinstore.com/api/trade/order/orderInfo?ordId=1780715084580128"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time()* 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = 'ordId=1780715084580128'
payload=payload.encode("utf-8")
signature = hmac.new(key,payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'Content-Type': 'application/json'
}
response = requests.request("GET", url, headers=headers)
print(response.text)
```


> 响应

```json
{
    "data": {
        "baseCurrency": "LUFFY",
        "quoteCurrency": "USDT",
        "symbol": "LUFFYUSDT",
        "timestamp": 1642574848089,
        "side": "BUY",
        "accountId": 1138204,
        "ordId": 1722364585836586,
        "clOrdId": "d0344830639d4f6d94be4110bf98c759",
        "ordType": "LIMIT",
        "ordState": "CANCELED",
        "ordPrice": "0.00000000092",
        "ordQty": "1",
        "ordAmt": "0.00000000092",
        "cumAmt": "0",
        "cumQty": "0",
        "leavesQty": "0",
        "avgPrice": "0",
        "feeCurrency": "LUFFY",
        "timeInForce": "GTC"
    },
    "code": 0
}
```

### HTTP请求: 
- GET trade/order/orderInfo

### 请求参数:

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|ordId | long | true  | 订单id |

### 响应数据:

|       code        |  type  |               comment                       |
| ----------------- | ------ | -------------------------------------- |
| code            | int     |     0：成功，其他失败                                               |
| message         | string    |    错误信息                                    |
| data | object|  | 
| ├─ordId| long | 订单ID |
| ├─clOrdId| String | 客户端订单ID |
| ├─accountId| long | 账户ID |
| ├─symbol | String |   | 交易对名称 |
| ├─baseCurrency| String | 基础货币 |
| ├─quoteCurrency| String | 计价货币 |
| ├─ordPrice | String | 价格 |
| ├─ordQty| String | 数量 |
| ├─ordAmt| String | 金额 |
| ├─side | Stirng | 方向 BUY,SELL|
| ├─cumAmt| String | 累计成交额 |
| ├─cumQty| String | 累计成交量 |
| ├─leavesQty| String | 剩余数量 |
| ├─ordState | Stirng | 状态 NOT_FOUND, REJECTED, SUBMITTING, SUBMITTED, PARTIAL_FILLED, REPLACING, REPLACED, CANCELING, CANCELED, EXPIRED, STOPPED, FILLED |
| ├─ordType | Stirng | 订单类型 MARKET,LIMIT,POST_ONLY  |
| ├─flags| Stirng | 订单类型 POST_ONLY,REDUCE_ONLY,HIDDEN |
| ├─timeInForce| Stirng | 成交限制类型 GTC,IOC,FOK |
| ├─timestamp| long | 订单时间 毫秒 |
| ├─avgPrice| long | 成交均价  |



## <span id="15">获取订单信息V2</span>
获取订单信息V2

#### 新接口的 API域名地址 `https://api.coinstore.com`  调用支持ApiKey


### HTTP请求:
- GET /api/v2/trade/order/orderInfo


> python 

```python
import hashlib
import hmac
import math
import time
import requests
url = "https://api.coinstore.com/api/v2/trade/order/orderInfo?ordId=1780715084580128"
api_key=b'your api_key'
secret_key = b'your secret_key'
expires = int(time.time()* 1000)
expires_key = str(math.floor(expires / 30000))
expires_key = expires_key.encode("utf-8")
key = hmac.new(secret_key, expires_key, hashlib.sha256).hexdigest()
key = key.encode("utf-8")
payload = 'ordId=1780715084580128'
payload=payload.encode("utf-8")
signature = hmac.new(key,payload, hashlib.sha256).hexdigest()
headers = {
   'X-CS-APIKEY': api_key,
 'X-CS-SIGN': signature,
 'X-CS-EXPIRES': str(expires),
 'Content-Type': 'application/json'
}
response = requests.request("GET", url, headers=headers)
print(response.text)
```

> 响应

```json
{
    "data": [
        {
          "baseCurrency": "A123",
          "quoteCurrency": "USDT",
          "side": "BUY",
          "cumQty": "0",
          "ordId": 1758065956225025,
          "clOrdId": "tLCGVA4g19zuEBwsITXi9g3U624Al0Bw",
          "ordType": "LIMIT",
          "ordQty": "10",
          "cumAmt": "0",
          "accountId": 1134912,
          "timeInForce": "GTC",
          "ordPrice": "100",
          "leavesQty": "10",
          "avgPrice": "0",
          "ordStatus": "SUBMITTED",
          "symbol": "A123USDT",
          "orderUpdateTime": 1676622290389,
          "timestamp": 1676622290389
        }
    ],
    "code": 0
}
```

### 请求参数

|    code    | type   | required | comment |
| ---------- |--------|--------|---------|
|ordId| int    | N      | 委托ID    |
|clOrdId| String | N      | 客户端委托ID  |


### 响应数据

|       code    |  type  | comment                                 |
| ------------- | ------ |-----------------------------------------|
| code          | int    | 0：成功，其他失败                               |
| message       | string    | 错误信息                                    |
| data          |  list  |                                         |
| ├─baseCurrency| string | base币                                   |
| ├─quoteCurrency| string | quot币                                   |
| ├─timeInForce| string | GTC,IOC,FOK                             |
| ├─ordStatus | string | 订单状态 NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 成交均价	                                   |
| ├─ordPrice | string | 订单价格                                    |
| ├─leavesQty| string | 剩余数量                                    |
| ├─ordType | string | 委托类型 MARKET,LIMIT,POST_ONLY             |
| ├─ordQty| string | 订单数量                                    |
| ├─cumAmt| string | 成交金额                                    |
| ├─side | string | BUY,SELL                                |
| ├─cumQty| string | 成交数量                                    |
| ├─ ordId      | long  |     订单id                                |
| ├─ clOrdId      | string     | 客户端订单id                                 |
| ├─symbol | string | 币对                                      |
| ├─orderUpdateTime| long | 订单状态更新时间：若未成交，则系统返回“订单创建时间”；若已成交，则系统返回最近一笔成交的时间             |
| ├─timestamp| long | 时间戳                                     |


# 行情相关

##  <span id="7">Ticker</span>
 市场所有交易对的Ticker
 
### HTTP请求: 
- GET /v1/market/tickers

> 响应

```json
{
  "data": [
    {
      "channel": "ticker",
      "bidSize": "454.2",
      "askSize": "542.6",
      "count": 41723,
      "volume": "24121351.85",
      "amount": "1693799.10798965",
      "close": "0.067998",
      "open": "0.071842",
      "high": "0.072453",
      "low": "0.067985",
      "bid": "0.0679",
      "ask": "0.0681",
      "symbol": "TRXUSDT",
      "instrumentId": 2
    }
  ],
  "code": 0
}
```

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─count | Integer  |24h滚动交易笔数 |
| ├─volume|String  |24h以报价币种计量的交易量 |    
| ├─amount|  String  |24h以基础币种计量的交易量 |           
| ├─close|  String  | 末一笔成交价| 
| ├─open|  String  | 第一笔成交价| 
| ├─high|  String  | 最高一笔成交价| 
| ├─low|  String  | 最低一笔成交价| 
| ├─bid|  String  | 买一价| 
| ├─bidSize|  String  | 买一量| 
| ├─ask|  String  | 卖一价| 
| ├─askSize|  String  | 卖一量| 
                     



 
##  <span id="7">获取深度</span>
获取深度数据

> 响应

```json
{
	"data": {
		"channel": "4@depth@5",
		"level": 5,
		"a": [
			[
				"41995",
				"0.018144",
				-1
			],
			[
				"41995.5",
				"0.008279",
				-1
			]
		],
		"b": [
			[
				"41991.5",
				"0.548567",
				1
			],
			[
				"41991",
				"0.013168",
				1
			]
		],
		"symbol": "BTCUSDT",
		"instrumentId": 4
	},
	"code": 0
}
 ```
 
### HTTP请求: 
- GET /v1/market/depth/{symbol}

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，如“BTCUSDT” |
|depth | int | false | 深度的数量，如“5, 10, 20, 50, 100”,默认20 |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─level| Integer  |数量 |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| string  |交易对 |
| ├─a | List  |卖盘，[${价格}, ${数量}, ${方向}] ] |
| ├─b | List  |买盘，[${价格}, ${数量}, ${方向}] ] |


##  <span id="7">获取K线</span>
获取交易K线

> 响应

```json
{
	"data": {
		"channel": "4@kline@week_1",
		"item": [
			{
				"endTime": 1641744059,
				"amount": "98630624.103511453",
				"interval": "week_1",
				"startTime": 1641744000,
				"firstTradeId": 14547884,
				"lastTradeId": 14799534,
				"volume": "2307.742209",
				"close": "43196.0002",
				"open": "41628.2582",
				"high": "52250.3884",
				"low": "39654.039"
			}
		],
		"symbol": "BTCUSDT",
		"instrumentId": 4
	},
	"code": 0
}
 ```
 
### HTTP请求: 
- GET /v1/market/kline/{symbol}

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | ------------------ |
|symbol | string | true | 交易对，如“BTCUSDT” |
|period | String | false | 数据粒度，如“1min, 5min, 15min, 30min, 60min, 4hour,12hour, 1day, 1week”,默认20 |
|size | Integer | false | 数据条数，[1,2000] |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─item | List  |K 线集合 |
| ├─├─interval | String  |K线间隔 |
| ├─├─endTime| Long |结束时间，单位 s |         
| ├─├─amount|String  |成交额 |               
| ├─├─startTime|Long  |开始时间，单位 s|     
| ├─├─firstTradeId|Long |第一笔成交ID |    
| ├─├─lastTradeId|Long  |末一笔成交ID |    
| ├─├─volume|  String  |成交量 |           
| ├─├─close|  String  | 末一笔成交价|            
| ├─├─open|   String  | 第一笔成交价|            
| ├─├─high|  String  |最高成交价 |             
| ├─├─low|   String  |最低成交价 |             



##  <span id="7">最新成交</span>
获取最新成交记录

### HTTP请求: 
- GET /v1/market/trade/{symbol}

> 响应

```json
 {
 	"data": [
 		{
 			"channel": "4@trade",
 			"time": 1642495112,
 			"volume": "0.011102",
 			"price": "41732.3305",
 			"tradeId": 14867136,
 			"takerSide": "BUY",
 			"seq": 14867136,
 			"ts": 1642495112000,
 			"symbol": "BTCUSDT",
 			"instrumentId": 4
 		}
 	],
 	"code": 0
 }
```

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，如“BTCUSDT” |
|size | Integer | false | 数据条数，[1,100] |

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | Object|   |
| ├─channel| String |标识  |
| ├─instrumentId| Integer  |交易对ID |
| ├─symbol| String  |交易对 |
| ├─interval | String  |K线间隔 |
| ├─time| Long |时间，单位 s |         
| ├─ts|Long  |时间，单位 ms |                  
| ├─tradeId|Long |第一笔成交ID |    
| ├─takerSide|String  |方向 "BUY"，"SELL" |    
| ├─volume|  String  |成交量 |           
| ├─price|  String  | 末一笔成交价|                       



##  <span id="7">获取所有交易对最新价格</span>
获取所有交易对最新价格

> 响应

```json
{
  "code": 0,
  "message": "",
  "data": [
    {
      "id": 1,
      "symbol": "BTCUSDT",
      "price": "400"
    },
    {
      "id": 4,
      "symbol": "AUTUSDT",
      "price": "1"
    }
  ]
}
 ```
 
### HTTP请求: 
- GET /v1/ticker/price

### 请求参数: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | 交易对，区分大小写，逗号分割 |

**默认：查询所有交易对**
**查询指定交易对: /v1/ticker/price;symbol=BTCUSDT,EOSUSDT,AUTUSDT**

### 响应数据: 

|       code        |  type   |                      comment                       |
| ----------------- | ------ | ---------------------------------------- |
| code            | int    |     0：成功，其他失败                                               |
| message         | string       |    错误信息                                    |
| data | List[]|   |
| ├─id| long |  |
| ├─symbol| string  |交易对 |
| ├─price| string  |价格 |

  
  
# Websocket行情数据

## **简介**

### 接入URL
wss://ws.coinstore.com/s/ws

1. 所有wss接口的 baseurl为: wss://<host:port>/s/ws

2. stream名称中所有交易对均为 小写

3. 每个链接有效期不超过24小时，请妥善处理断线重连。

4. 每3分钟，服务端会发送ping帧，客户端应当在10分钟内回复pong帧，否则服务端会主动断开链接。允许客户端发送不成对的pong帧(即客户端可以以高于10分钟每次的频率发送pong帧保持链接)。


### 服务端推送数据类型说明

> Format Schema

```lang=json
{
    "S": 1, // session 级别的 response 消息序号，session 重连后重置，可用来判断是否漏消息
    "T": "echo|resp", // 响应类型，详见 `Types` 部分
    "C": 200,  // code
    "M": "sub.channel.success",  // meaning of this code
    "sid": "3ae985ff-cadd-9c96-7a42-4ed29e77f83b",
    "echo": {},  // client request command
    "payload": "external msg"
}
```

#### Types

##### request-response 类型的消息

* `echo`: 服务器接受到到每一个消息都会以该种形式返回一个message，确认消息已经被接收
* `resp`: 服务器接受到到每一个消息都会以该种形式返回一个处理结果的message，标记消息处理，返回信息中包含处理结果信息


##### subscribe 消息类型
以下类型的消息，从执行对应 channel 的 `SUB` 命令开始，到执行 `UNSUB` 结束，如果服务器端数据发生变化则推送，最小推送间隔 100ms
```lang=json
{
    "S": 1,
    "T": "kline|ticker|depth|trade|account",
    ...:  // 详细见下面到每种数据订阅格式 
}
```
* `kline`

* `ticker`

* `depth`

* `trade`

* `account`

* `order`



### Pong
服务器端支持 websocket pong frame 和 pong message 两种形式端 pong 响应

> Pong message format

```lang=json
{
  "op": "pong",
  "epochMillis": 1604040975
}
```

#### Websocket Pong frame

1. https://tools.ietf.org/html/rfc6455#section-5.5.3 


> Example

```lang=shell
$>wscat -c 'ws://127.0.0.1:8080/s/ws'
< {"S":1,"T":"resp","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"established"}
> {"op":"SUB","channel":["80004@trade"],"id":1}
< {"S":2,"T":"echo","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"command.received","echo":{"op":"SUB","channel":[{"name":"80004@trade","type":{"name":"trade","auth":"PUB"},"instrumentId":"80004","lifeStartTime":1604040964,"msgCount":0,"pub":true}],"id":1}}
< {"S":3,"T":"resp","sid":"3ae985ff-cadd-9c96-7a42-4ed29e77f83b","C":200,"M":"sub.channel.success"}
< {"S":4,"T":"trade","channel":"80004@trade","time":1604040975,"price":"9811.7494086","takerSide":"SELL","tradeId":26461,"volume":"7.505","symbol":"EOSUSD","instrumentId":80004}
```

## **实时订阅/取消数据流**

> 订阅一个信息流

```lang=json
{
    "op": "SUB",
    "channel": [
        "btcusdt@ticker",
        "btcusdt@depth"
    ],
    "id": 1
}
```

> 取消订阅一个信息流

```lang=json
{
    "op": "UNSUB",
    "channel": [
        "btcusdt@depth"
    ],
    "id": 312
}
```

> 已订阅信息流

```lang=json
{
    "op": "LIST",
    "channel":[],
    "id": 3
}
```

* 以下数据可以通过websocket发送以实现订阅或取消订阅数据流。示例如下。
* 响应内容中的id是无符号整数，作为往来信息的唯一标识。
* 如果相应内容中的 result 为 null，表示请求发送成功。

### Client `op` Types

1. `SUB`
2. `UNSUB`
3. `LOGIN`
4. `REQ`
5. `LIST`



> Respon

```lang=json
{
    "S": 12,
    "T": "resp",
    "subs": [
        {
            "name": "80004@trade",
            "type": {
                "name": "trade",
                "auth": "PUB"
            },
            "instrumentId": "80004",
            "lifeStartTime": 1604043594,
            "msgCount": 212
        }
    ],
    "sid": "b1beeb73-7c84-862f-3d03-619de1e1a5dc"
}
```

> NOTE: `<symbol>` 参数暂时请使用 交易对的id，后续支持使用交易对的名称

> IMPORTANT:  请优先使用  `symbol` 字段，` instrumentId` 标记为 `Deprecated`

> NOTE:  所有返回数据的时间，单位都是 `秒`


## **逐笔交易**

> Send

```lang=json
 {
 	"op": "SUB",
 	"channel": [
 		"28@trade"
 	],
 	"param": {
 		"size": 2
 	},
 	"id": 1
 }
```
> Receive全量数据：

```lang=json
 {
   "S": 15,
   "T": "trade",
   "data": [
     {
       "channel": "28@trade",
       "price": "384.32",
       "tradeId": 21704,
       "seq": 10157,
       "ts": 1612685697087,
       "takerSide": "BULL",
       "time": 1612685697,
       "volume": "2",
       "symbol": "BTCJJ",
       "instrumentId": 28
     },
     {
       "channel": "28@trade",
       "price": "644.96",
       "tradeId": 21705,
       "seq": 10158,
       "ts": 1612685698157,
       "takerSide": "SELL",
       "time": 1612685698,
       "volume": "1.1932988",
       "symbol": "BTCJJ",
       "instrumentId": 28
     }
   ]
 }
```
> Receive增量数据：

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",    // 交易对
  "tradeId": 12345,       // 交易ID
  "takerSide": "BUY",    // taker side
  "price": "0.001",     // 成交价格
  "volume": "100",       // 成交数量
  "time": 123456785,   // 成交时间 单位 s
  "ts": 1612685313400, // 单位 ms
  "seq": 9935         // 唯一自增序号
}
```

### Stream Name
`<symbol>@trade`

eg: `4@trade`
 
### param
 `param":{"size":2}`
 




## **K线 Streams**
K线stream逐秒推送所请求的K线种类(最新一根K线)的更新。

> Send

```lang=json
 {"op":"SUB","channel":["4@kline@min_1"],"id":1}
```
> Receive

```lang=json
{
   "instrumentId": 88066, 
   "startTime": 1603732500,  // 这根 k线的开始时间，单位 s
    "endTime": 1603732559,  // 这根 k线的开始时间，单位 s
    "symbol": "USDTBTC",  // 交易对
    "interval": "1m",      // K线间隔
    "firstTradeId": 100,       // 这根K线期间第一笔成交ID
    "lastTradeId": 200,       // 这根K线期间末一笔成交ID
    "open": "0.0010",  // 这根K线期间第一笔成交价
    "close": "0.0020",  // 这根K线期间末一笔成交价
    "high": "0.0025",  // 这根K线期间最高成交价
    "low": "0.0015",  // 这根K线期间最低成交价
    "volume": "1000",    // 这根K线期间成交量
    "amount": "1.0000",  // 这根K线期间成交额
}
```

### Stream Name
`<symbol>@kline@<interval>`
  
eg: `4@kline@min_1`

### interval 可选值
* min_1
* min_5
* min_15
* min_30
* hour_1
* hour_4
* hour_12
* day_1
* week_1



## **K线 Request**
请求历史k线

> Send

```lang=json
 {
 	"op": "REQ",
 	"param": {
 		"channel": "4@kline@min_1",
 		"endTime": null,
 		"limit": 200
 	},
 	"id": 1
 }
```
> Receive

```lang=json
{
    "channel": "88066@kline@min_1",
    "symbol": "USDTBTC",
    "instrumentId": 88066,
    "item": [
        {
            "interval": "min_1",
            "startTime": 1603766280,
            "endTime": 1603766339,
            "lastTradeId": 4739656,
            "firstTradeId": 4739656,
            "close": "4.91",
            "open": "4.91",
            "high": "4.91",
            "low": "4.91",
            "volume": "8.488",
            "amount": "41.67608"
        },
        ...
    ]
}
```

###  param

* `channel`: `<symbol>@kline@<interval>` ,eg:88066@kline@min_1
* `limit`: 最大不超过 200
* `endTime`: 可选，exclusive





## **按 Symbol 的 Ticker信息**
按Symbol刷新的最近24小时精简 ticker 信息

> Send

```lang=json
 {"op":"SUB","channel":["4@ticker"],"id":1}
```
> Receive

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",      // 交易对
  "open": "0.0010",      // 整整24小时前，向后数的第一次成交价格
  "high": "0.0025",      // 24小时内最高成交价
  "low": "0.0010",      // 24小时内最低成交加
  "close": "0.0025",      // 最新成交价格
  "volume": "10000",       // 24小时内成交量
  "amount": "18"          // 24小时内成交额
}
```

### Stream 名称
`<symbol>@ticker`, eg:  `88066@ticker`



## **深度信息**
每秒或每100毫秒推送有限档深度信息。

> Send

```lang=json
 {"op":"SUB","channel":["4@depth@20"],"id":1}
```

> Receive

```lang=json
{
  "level": 5,
  "instrumentId": 88066,
  "symbol": "BTCUSDT"
  "b": [             // Bids to be updated
    [
      "0.0024",         // Price level to be updated
      "10"              // Quantity
    ]
  ],
  "a": [             // Asks to be updated
    [
      "0.0026",         // Price level to be updated
      "100"            // Quantity
    ]
  ]
}
```

### Stream Names
`<symbol>@depth@<levels>` 

 eg:  `4@depth@50`
 
### levels 可选值
levels表示几档买卖单信息, 可选 5/10/20/50/100档

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
| part-trade-limit 3012                 | 您已被限制交易当前现货币对，如有疑问请咨询客服，给您造成不便敬请谅解。                      |
| trade-limit 3013                      | 您暂时不具备币币交易资质，如有疑问请咨询客服，给您造成不便敬请谅解。                      |
| user-trade-limit 3014                 | 当前子账号暂时不具备币币交易资质                      |
| user-part-trade-limit 3015            | 当前子账号已被限制交易当前现货币对                      |
| account-insufficient 3113             | 账户余额不足                     |
| duplicate-order 3111                  | 重复订单                             |
| order-not-found 3103                  | 订单未找到或者该订单不属于当前账号   |
| sub-user-operator-business-error 6001 | 子账号不允许参与该业务   |



## **字典**

### OrderState

* REJECTED
* SUBMITTING
* SUBMITTED
* PARTIAL_FILLED
* CANCELING
* CANCELED
* EXPIRED
* STOPPED
* FILLED

### OrderType

* MARKET 
* LIMIT 
* POST_ONLY

### TimeInForce

* IOC
* GTC

### MatchRole

* TAKER
* MAKER

### Side

* BUY
* SELL


