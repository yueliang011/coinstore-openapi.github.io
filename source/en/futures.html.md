---
title: Coinstore official API document

language_tabs: # must be one of https://git.io/vQNgJ
  

toc_footers:

includes:
  
language: English

other_language: 简体中文

present_url: /en

url: /cn

active: active

other_active: 

menu: Menu

create_api:  Create API KEY

spot_goods: Spot

spot_goods_url: 'index.html'

contract: Perpetual Swap

contract_active: active

contract_url: 'futures.html'

searchText: Search

search: true

code_clipboard: true
---

# Introduction

Welcome to Coinstore Developer Documentation. It is the only official documentation of Coinstore API.

This documentation provides an introduction to the use of related APIs.

RESTful API includes interfaces such as assets, orders and tickers.

Websocket provides ticker-related interface and push service.

The services provided by Coinstore API will be continuously updated here, so stay tuned for updates.



# Quick Start

## Access Preparation

To use API, please log into the webpage first, create an API key through [User Center] - [API Managment], and then develop and trade according to the details of this documentation.

You can click 'https://www.coinstore.com/#/user/bindAuth/ManagementAPI' to create an API Key.

Each user can create 5 groups of API Keys, and each group of API Keys can bind 5 different IP addresses. Once an API key binds an address, the API interface can only be called by using the API key from the bound IP address. For security reasons, it is strongly recommended that you bind the corresponding IP address for API key.

Please remember the following information upon successful creation:

- `API Key`  API access key
- `Secret Key` Key for encryption of signature authentication

## Interface Type
Coinstore provides users with two interfaces, and you can choose the appropriate way to query the ticker and trade according to your own usage scenarios and preferences.

**REST API**

REST, the abbreviation of Representational State Transfer, is a popular Internet transmission architecture. It has the characteristics of clear structure, standards compliance, easy understanding and expansion, and is being adopted by more and more websites with the benefits as follows:

- In RESTful architecture, each URL represents a resource;
- Some representation layer of this resource is transferred between the client and the server;
- The client operates the server-side resources through four HTTP instructions to realize "presentation layer state transformation". 

Developers are advised to use REST API for one-time operations such as trades or assets.

**WebSocket API**

WebSocket is a new HTML5 Protocol. It realizes full-duplex communication between client and server, and the connection between client and server can be established by a simple handshake. The server can actively push information to the client according to business rules.

Developers are advised to use WebSocket API to obtain market tickers, asks/bids depth and other information.

### Interface Authentication

The above two interfaces include public interface and private interface.

Public interface can be used to obtain basic information and ticker data. Public interfaces can be called without authentication.

Private interface can be used for trading management. Every private request must use your API Key for signature verification.

## Access URLs

**REST API**

`https://futures.coinstore.com/api`

**WebSocket**

`wss://ws-futures.coinstore.com/socket.io/?EIO=3&transport=websocket`

To ensure the stability of API service, it is recommended to access using Japanese AWS cloud server. If the client server in Chinese mainland is used, it would be difficult to guarantee the stability of the connection.


## <span id="a4"> Signature Authentication </span>

**Signature Description**

The API request is very likely to be tampered in the process of transmission through the Internet. In order to ensure that the request has not been changed, all private interfaces other than public interfaces (basic information, ticker data) must use your API Key for signature authentication to verify whether the parameters or parameter values have changed during transmission.

**Signature Algorithm**

HmacSHA256 hash function is used as signature function.
```
Mac hmacSha256 = Mac.getInstance("HmacSHA256");
```

**Signature Steps**
    
1. The signature valid string consists of request parameters and request body.
Note: The request parameters and request body are not sorted but directly spliced into a string as payload.

 ```java
  Example 1: GET Request Query String
 ?symbol=aaaa88&size=10
 
 String payload = "symbol=aaaa88&size=10"；
 ```



```java
Example 2: POST Request Body
{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}


 String payload = "{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}"；
```


```java
Example 3: Mixed Request
 ?symbol=aaaa88&size=10
{"symbol":"aaaa88","side":"SELL","ordType":"LIMIT","ordPrice":2,"ordQty":1,"timestamp":1627384801051}

 String payload = "symbol=aaaa88&size=10{"symbol":" aaaa88","side":" SELL","ordType":" LIMIT","ordPrice":2,"ordQty":1,
 "timestamp":1627384801051}"；
```




2. Use signature function to calculate hash value for timestamp

> Note: X-CS-EXPIRES is a 13-bit timestamp, which needs to be divided by 30000 to obtain a class timestamp. It is calculated by signature function to obtain the function value as the key of step 3 (the value of variable key in step 3).


```java
 String time = String.valueOf(X-CS-EXPIRES / 30000);
 hmacSha256.init(new SecretKeySpec(Secret_Key.getBytes(), "HmacSHA256"));
 byte[] hash = hmacSha256.doFinal(time.getBytes());
 String key = Hex.toHexString(hash);
```





3. Use signature function to calculate hash value for valid string

> Note: The value of key is the hash value calculated in step 2.

```java
 hmacSha256.reset();
 hmacSha256.init(new SecretKeySpec(key.getBytes(), "HmacSHA256"));
 hash = hmacSha256.doFinal(payload.getBytes());
 String sign= Hex.toHexString(hash);
```

# API Access Instructions

## <span id="a3"> Request Format </span>
All API requests are restful, and there are only two methods at present: GET and POST.
- GET request: All parameters are in path parameters
- POST request: Parameters can be set in the path, and they can be sent in JSON format to the request body. If there are no parameters,{} needs to be sent

A licit request consists of the following parts:
- method request address: Access server address futures.coinstore.com，e.g. https://futures.coinstore.com/api/future/place
- Required and optional parameters.
- X-CS-APIKEY: API Key applied by the user.
- X-CS-EXPIRES: Timestamp when you issued the request. For example:1629291143107.
- X-CS-SIGN: A string calculated from the signature, which is used to ensure that the signature is valid and not tampered with.

**Note: X-CS-APIKEY, X-CS-EXPIRES and X-CS-SIGN are all in the request header, and 'Content-Type':'application/json' 
needs to be set.**

## <span id="a3">Return Format</span>

All interface returns are in JSON format. There are three fields in the first layer of JSON: code, message and data. The first two fields indicate the request status and description, and the actual business data is in the data field.

```json
{
    "code": "0",
    "message": "suc",
    "data": // per API response data in nested JSON object
}
```
The following is an example of a return format:

| field| data type  | description       |
| ---- | -----  | ---------- |
| code | int | 0：success, other: failure     |
| message  | string | status or error description |
| data | object | transaction data  |


## Error Message

**HTTP Status Code**

Common error codes for HTTP are as follows:
- 400 Bad Request – Invalid request format

- 401 signature failed – Invalid API Key

- 404 service not found

- 429 too many visits

- 500 internal server error

**Transaction Status Code**

In case of failure, the response message carries error description information, and the corresponding status code is described as follows:

|status code| description                     | remarks                        |
| ------    | --------------------------------| --------------------------     |
| 0         | success                         | code=0 success, code >0 failure|

# Basic Information


## <span id="1">Currency & contract information</span>

Get currency & contract information

### HTTP Request: 
- GET /api/configs/public


> Response 

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |   |   |  |

### Response Data

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int      |  0：success, other: failure           |
| message | String  |  error message |
| data | Object   |   transaction data |
|- currencies | List \<Currency>    | currency list |
|- contracts  | List \<Contract>   | contract list |
|- version  | Long   | version |


#### Currency

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| currencyId | int  | currency id |
| name | String  | currency name |
| disableTransferIn | boolean | whether transfer to futures, default: false, transfer possible |
| disableTransferOut | boolean | whether transfer to spot, default: false, transfer possible |

#### Contract

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- | 
| contractId | int  | contract id | 
| currencyId | int  | margin currency id |
| name | String  | contract name BTCUSDT | 
| displayName | String  | contract display name BTC/USDT |
| baseAsset | String  | base asset name BTC |   
| quoteAsset | String  | quote asset name USDT | 
| marginAsset | String  | margin asset name USDT |  
| tickSize | BigDecimal  | tick size | 
| priceScale | int  | price decimal | 
| maxOrderSize | int |  maximum order size, default 0, unlimited | 
| minOrderSize | int |  minimum order size, default 0, unlimited |  
| takerFeeRate | BigDecimal  | Taker fee rate |  
| makerFeeRate | BigDecimal  | Maker fee rate | 
| contractSize | BigDecimal  | contract size |   
| minMaintRate | BigDecimal  | minimum maintenance margin rate | 
|fundingInterval | int  | funding interval, unit: second  | 
|weight | int  |ranking weight  flashback | 
| tags | String[] |  tags    mock, hot, new, ... |  
| riskLimits | RiskLimit[]  | risk limit |  

#### RiskLimit

|       code  |  type  |   comment    |
| ---- | ----- | ---------- | 
| maxSize | long|  risk limits |
| maintRate | BigDecimal |  maintenance margin rate |
| leverage | int |  leverage rate |

# Account Related


## <span id="1">Assets Balance</span>

Get user assets balance

### HTTP Request: 
- POST  /api/future/queryAvail


> Response 

```json
{
    "code": 0,
    "message": "success",
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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |

### Response Data

|       code        |  type  |                      comment                       |
| ---- | ----- | ---------- |
| code | int        |   0: success, other: failure         |
| message | String    | error information |
| data | Object []    |  transaction data |
| - currencyId| string       |   margin id  |
| - totalBalance| string      |  total balance   |
| - available| string        | available balance |
| - frozenForTrade| string       | order frozen balance|
| - initMargin| string        | occupied margin|
| - frozenInitMargin| string       | order frozen margin|
| - closeProfitLoss| string        | realized P&L|
| - dailyProfitLoss| string        | daily realized P&L|
| - closeProfitLoss| string        | realized P&L|
| - unRealizedProfit| string        | unrealized P&L|
| - accountEquity| string        | net asset|
| - leverLevel| string        | overall leverage level|
| - unRealizedProfit| string        | unrealized P&L|
| - toBtc| string        | btc asset value|
| - accountId| string        | account id|
| - accountType| string        | account type  1  main account |

# Position Related

## <span id="1">Position Query</span>

Query user futures position

### HTTP Request: 
- GET  /api/future/queryPosi


> Response 

```json
{
    "code": 0,
    "message": "success",
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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |

### Response Data

|       code        |  type  |     comment                       |
| ---- | ----- | ---------- |
| code | int        |  0: success, other: failure          |
| message | String    | error information |
| data | Object []    |  transaction data |
| - currencyId| string     |   margin id  |
| - posiQty| string       |  positions, >0: long, <0: short  |
| - openAmt| string       | open amount |
| - initMargin| string        | initial margin|
| - posiStatus| string        | 1: normal, 2: to be liquidated|
| - marginType| string        | margin type, 1: cross, 2: isolated|
| - closeProfitLoss| string        | realized P&L|
| - initMarginRate| string        | initial margin rate|
| - maintainMarginRate| string        | maintenance margin|
| - frozenCloseQty| string        | frozen close position|
| - frozenOpenQty| string        | frozen open position|
| - extraMargin| string        | extra margin   deprecated|
| - contractUnit| string        | contract unit|
| - openPrice| string        | average open price|
| - unRealizedProfit| string        | unrealized P&L|
| - liquidationPrice| string        | estimated liquidated price|
| - lastPrice| string        | the latest trade price|
| - markPrice| string        | marked price * total cont * futures face value|
| - accountId| string        | account id|
| - accountType| string        | account type  1  main account|

## <span id="1">Adjusted Margin</span>


### HTTP Request: 
- POST  /api/future/adjustMargin


> Response 

```json
{
    "code": 0,
    "message": "success"
}
```

### Request Body

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  contractId| int|Y| symbol ID|
|   margin| int|Y|  margin amount (>0 : increase margin, <0: decease margin)|

### Response Data

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int        | 0: success, other: failure           |
| message | String    | error information |
| data | Object []    |  transaction data |

## <span id="1">Adjusted leverage rate</span>


### HTTP Request: 
- POST  /api/configs/adjustPositionConfig


> Response 

```json
{
    "code": 0,
    "message": "success",
```

### Request Body


|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  contractId| int|Y| symbol ID|
|   leverage| int|Y|  leverage rate, 0 represents automatic mode, isolated mode can’t be 0|
|   isolated| boolean|Y|  margin type, default false:cross, true:isolated|

### Response Data

|       code        |  type  |                       comment                       |
| ---- | ----- | ---------- |
| code | int        |  0: success, other: failure          |
| message | String    | error information |
| data | Object []    |  transaction data |

# Order Related

## <span id="1">Create order</span>
Create order

### HTTP Request: 
- POST /api/trade/order/place

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| accountType| int|N| account type |
| clientOrderId| string|N| client order ID, uuid is used if not passed |
| contractId| int|Y| contract ID|
| marginRate| string|N| margin rate, cross>0, isolated>0 marginRate=1/levarage,range:0~1|
| leverage| int|Y| margin rate|
| marginType| int|Y| margin type, 1: cross, 2: isolated|
| orderSubType| int|Y|0 (default value), 1 (passive order), 2 (latest price triggering conditional order), 3 (index triggering conditional order), 4 (marked price triggering conditional order)|
| orderType| int|Y| order type, 1 (limit), 3 (market)|
| positionEffect| int|Y| position effect, 1 (open), 2 (close), 2 must be passed for close orders|
| price| string|N| order price, non required when order_type is 3 (market)|
| quantity| string|Y| order quantity|
| side| int|Y| buy: 1, sell: -1|
| stopPrice| string|optional| stop price, not required when order_type is 3 (market) //in case of conditional order|                                                                



> Response

```json
{
    "code": 0,
    "message": "success",
    "data": "1712040078475777"
}
```                                 

### Response Data

|       code        |  type  |                        comment                       |
| ----------------- | ------ |  -------------------------------------------------- |
| code            | int    |      0: success, other: failure                                               |
| message         | string    |     error information                                    |
| data            |  string   |   order id                                               |



## <span id="2">Get active order</span>

Get active order

### HTTP Request: 

- GET /api/trade/order/active


> Response

```json
{
    "code": 0,
    "message": "success",
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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|    |  |


### Response Data

|       code        |  type  |                       comment                       |
| ----------------- | ------ |  -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data            |  list  |      |
| - accountType| int  | account type |
| - clientOrderId| string  | client order ID, uuid is used if not passed |
| - contractId| int  | contract ID|
| - marginRate| string  | margin rate, cross>0, isolated>0|
| - leverage| int  | margin rate|
| - marginType| int  | margin type, 1: cross, 2: isolated|
| - minimalQuantity| string  | the latest trade volume, not required when order_type is 3 (market)//not passed at front-end|
|| - orderSubType| int  |0 (default value), 1 (passive order), 2 (latest price triggering conditional order), 3 (index triggering conditional order), 4 (marked price triggering conditional order) |
| - orderType| int  | order type, 1 (limit), 3 (market) |
| - positionEffect| int  | position effect, 1 (open), 2 (close)|
| - price| string  | non required when order_type is 3 (market)|
| - quantity| string  | order quantity|
| - side| int  | buy: 1, sell: -1|
| - stopCondition| int  | stop loss, non required when order_type is 3 (market); range: 1 (take profit, not enabled), 2 (stop loss, not enabled), 3 (only reduction, not enabled) //not passed at front-end|
| - stopPrice| string  | stop price, not required when order_type is 3 (market) //in case of conditional order|
| - symbol| string  | contract name|


## <span id="2">Get active order V2</span>

Get active order V2 version

#### The new interface API domain name address `https://futures.coinstore.com`  Call support for ApiKey

### HTTP Request:

- GET /api/v2/trade/order/active


> Response

```json
{
    "code": 0,
    "message": "success",
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

### Request Parameters

|    code    |  type   | required | comment            |
| ---------- | ------- | -------- |--------------------|
|contractId| int    | N      | contract id        |
|ordId| int    | N      | Delegate ID        |
|clOrdId| String | N      | Client delegate ID |


### Response Data

|       code        |  type  | comment                                                                                                                                                                           |
| ----------------- | ------ |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code            | int    | 0                                                                                                                                                                                 |     0: success, other: failure                                               |
| message         | string    |                                                                                                                                                                                   |    error message                                    |
| data            |  list  |                                                                                                                                                                                   |
| - accountType| int    | account type 1：user 2:system 3:bonus  4:follow                                                                                                                                    |
| - contractId| int    | contract id                                                                                                                                                                       |
| - orderId| long   | Delegate ID                                                                                                                                                                       |
| - clOrderId| string | Client delegate ID                                                                                                                                                                |
| - orderStatus| int    | order status 0：NOT_DECLARE 1:IN_DECLARE 2:NOT_MATCH 3:PORTION_MATCH 4: ALL_MATCH 5：PORTION_CANCEL 6： ALL_CANCEL 7:IN_CANCEL 8:INVALID	                                                                                                      |
| - timeInForce| int    | Type of transaction restriction：1:GTC, 2:IOC, 3:FOK	                                                                                                                              |
| - initMarginRate| string | Margin ratio                                                                                                                                                                      |
| - marginLeverage| int    | Margin multiple	                                                                                                                                                                  |
| - marginType| int    | margin type, 1: cross, 2: isolated                                                                                                                                                |
| - orderType| int    | order type 1：LIMIT  2：Condition of single price  3：MARKET  4：Conditional market price	                                                                                            |
| - orderSubType| int    | 0 (default value), 1 (passive order), 2 (latest price triggering conditional order), 3 (index triggering conditional order), 4 (marked price triggering conditional order)        |
| - positionEffect| int    | position effect, 1 (open), 2 (close)                                                                                                                                              |
| - orderPrice| string | non required when order_type is 3 (market)	                                                                                                                                       |
| - orderQty| string | order quantity                                                                                                                                                                    |
| - side| int    | buy: 1, sell: -1	                                                                                                                                                                 |
| - stopCondition| int    | stop loss, non required when order_type is 3 (market); range: 1 (take profit, not enabled), 2 (stop loss, not enabled), 3 (only reduction, not enabled) //not passed at front-end |
| - stopPrice| string | stop price, not required when order_type is 3 (market) //in case of conditional order                                                                                             |
| - matchQty| string | match quantity                                                                                                                                                                    |
| - matchAmt	| string | match amount                                                                                                                                                                      |
| - avgPrice| String | Average transaction price                                                                                                                                                         |
| - remainQty| int    | remain   quantity                                                                                                                                                                 |
| - orderTime| long   | order time                                                                                                                                                                        |


## <span id="2">get order info V2</span>

get order info V2 version

#### The new interface API domain name address `https://futures.coinstore.com`  Call support for ApiKey

### HTTP Request:

- GET /api/v2/trade/order/orderInfo


> response

```json
{
    "code": 0,
    "msg": "success",
    "data": [
          {
            "accountType": 10,
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

### request paramters

|    code    |  type   | required | comment |
| ---------- | ------- | -------- |---------|
|ordId| int    | N      | Order id    |
|clOrdId| String | N      | Client order number |


### response parameters

| code             | type   | comment                                                                                                                                                                           |
|------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| code             | int    | 0                                                                                                                                                                                 |     0：成功，其他失败                                               |
| msg              | string |                                                                                                                                                                                   |    错误信息                                    |
| data             | list   |                                                                                                                                                                                   |
| - accountType| int    | account type 1：user 2:system 3:bonus  4:follow                                                                                                                                    |
| - contractId| int    | contract id                                                                                                                                                                       |
| - orderId| long   | Delegate ID                                                                                                                                                                       |
| - clOrderId| string | Client delegate ID                                                                                                                                                                |
| - orderStatus| int    | order status 0：NOT_DECLARE 1:IN_DECLARE 2:NOT_MATCH 3:PORTION_MATCH 4: ALL_MATCH 5：PORTION_CANCEL 6： ALL_CANCEL 7:IN_CANCEL 8:INVALID	                                            |
| - timeInForce| int    | Type of transaction restriction：1:GTC, 2:IOC, 3:FOK	                                                                                                                              |
| - initMarginRate| string | Margin ratio                                                                                                                                                                      |
| - marginLeverage| int    | Margin multiple	                                                                                                                                                                  |
| - marginType| int    | margin type, 1: cross, 2: isolated                                                                                                                                                |
| - orderType| int    | order type 1：LIMIT  2：Condition of single price  3：MARKET  4：Conditional market price	                                                                                            |
| - orderSubType| int    | 0 (default value), 1 (passive order), 2 (latest price triggering conditional order), 3 (index triggering conditional order), 4 (marked price triggering conditional order)        |
| - positionEffect| int    | position effect, 1 (open), 2 (close)                                                                                                                                              |
| - orderPrice| string | non required when order_type is 3 (market)	                                                                                                                                       |
| - orderQty| string | order quantity                                                                                                                                                                    |
| - side| int    | buy: 1, sell: -1	                                                                                                                                                                 |
| - stopCondition| String    | stop loss, non required when order_type is 3 (market); range: 1 (take profit, not enabled), 2 (stop loss, not enabled), 3 (only reduction, not enabled) //not passed at front-end |
| - stopPrice| string | stop price, not required when order_type is 3 (market) //in case of conditional order                                                                                             |
| - matchQty| string | match quantity                                                                                                                                                                    |
| - matchAmt	| string | match amount                                                                                                                                                                      |
| - avgPrice| String | Average transaction price                                                                                                                                                         |
| - remainQty| int    | remain   quantity                                                                                                                                                                 |
| - profitAndLoss  | String | Profit and loss                                                                                                                                                                   |
| - fee            | String | Accumulated commission	                                                                                                                                                           |
| - feeCurrencyName       | String | Transaction fee currency	                                                                                                                                                         |
| - orderUpdateTime       | long   | The time of the last transaction or Order time	                                                                                                                                   |
| - orderTime      | long   | order time                                                                                                                                                                        |


## <span id="3">Cancel orders</span>
Cancel orders

### HTTP Request: 
- POST /api/trade/order/cancel

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId     | int | Y   |contract Id |
| originalOrderId     | long     | Y    |original order Id sb |

> Response

```json
{
    "code": 0,
    "message": "success",
    "data": "cancel order success"
}
```

### Response Data

|       code        |  type  |                         comment                       |
| ----------------- | ------ |    -------------------------------------------------- |
| code              | int    |     0: success, other: failure                                   |
| message         | string    |    error message                             |
| data            |  string  |  return data                                             |

##  <span id="5">One-click cancellation</span>


### HTTP Request: 
- POST /api/trade/order/cancelAll

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId     | int | N | cancel all contract orders if not passed |


> Response

```json
{
    "code": 0,
    "message": "success",
    "data": "cancel all order success"
}
```

### Response Data

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |  0: success, other: failure                                               |
| message         | string    |  error message                                    |
| data            |  string  |  return data                                             |


## <span id="13">Place batch</span>
Place batch
### HTTP Request: 
- POST /api/trade/order/placeBatch

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| orders        | list | Y      |
| - clientOrderId| string|N| client order ID, uuid is used if not passed |
| - contractId| int|Y| contract ID|
| - initRate| string|Y| margin rate, cross>0, isolated>0,initRate=1/leverage,range:0~1|
| - marginType| int|Y| margin type, 1: cross, 2: isolated|
| - orderSubType| int|Y|0 (default value), 1 (passive order)|
| - orderType| int|Y| order type, 1 (limit), 3 (market)|
| - positionEffect| int|Y| position effect, 1 (open), 2 (close), 2 must be passed for close orders|
| - orderPrice| string|N| order price, non required when order_type is 3 (market)|
| - orderQty| string|Y| order quantity|
| - side| int|Y| buy: 1, sell: -1|

> Request Body

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

> Response

```json
{
    "code": 0,
    "message": "success",
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

### Response Data

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |   0: success, other: failure                                     |
| message         | string    |error message                                    |
| data            |  Object  |  return data                                                 |
| - reject            | Object[]   |   rejected order  [client order ID, error code]          |
| - succ            | Object[]   |   successful order     [client order ID, order id]         |

## <span id="27">Batch cancellation according to order id</span>
Batch cancellation according to order id

### HTTP Request: 
- POST /api/trade/orders/del

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| cancels        | list | Y      |
| - originalOrderId| string|Y| client order ID, uuid is used if not passed |
| - contractId| int|Y| contract ID|

> Request Body

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

> Response

```json
{
    "code": 0,
    "message": "success",
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

### Response Data

|       code        |  type  |                        comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code         | string    |    normal return information                                |
| message         | string    |   error message                                   |
| data            | Object   |                                             |
| -succ|   Object[]   |    set of successful order ids         [client order ID, order id]                                 |
| -reject  |   Object[]   | set of rejected order ids        [client order ID, error code, order id]                 |




## <span id="3">Get user's all trade records</span>
Get all trade records

### HTTP Request: 
- GET /api/future/queryHisMatch


> Response

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
  "message" : "string"
}
```

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| contract id|
| endTime| string|N| end time |
| startTime| string|N| start time|
| pageNum| string|N| page number|
| pageSize| string|N| page size|
| side| string|N| side -1 Previous, 1 Next|

### Response Data

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | object|    |
| -	applId	|	int	  |	futures ID, 2	|
| -	matchTime	|	Long	  |	trade time	|
| -	contractId	|	Long	  |	symbol ID, contract ID	|
| -	execId	|	Long	  |	trade ID	|
| -	bidUserId	|	Long	  |	bid user ID	|
| -	askUserId	|	Long	  |	ask user ID	|
| -	bidOrderId	|	Long	  |	bid order ID	|
| -	askOrderId	|	Long	  |	ask order ID	|
| -	matchPrice	|	Double	  |	trade price	|
| -	matchQty	|	int	  |	trade volume	|
| -	matchAmt	|	Double	  |	trade amount	|
| -	bidFee	|	Double	  |	bid fee	|
| -	askFee	|	Double	  |	ask fee	|
| -	takerSide	|	int	  |	Taker side, 1: bid Taker, -1: ask Taker	|
| -	side	|	int	  |	bid/ask side	|
| -	updateTime	|	Long	  |	last update time	|
| -	bidPositionEffect	|	int	  |	bid position effect	|
| -	askPositionEffect	|	int	  |	ask position effect	|
| -	bidMarginType	|	int	  |	bid margin type	|
| -	askMarginType	|	int	  |	ask margin type	|
| -	bidInitRate	|	Double	  |	bid initial margin rate	|
| -	askInitRate	|	Double	  |	ask initial margin rate	|
| -	bidMatchType	|	int	  |	bid trade type: 0: ordinary trade, 1: liquidation trade, 2: forced deduction trade (bankrupt party) 3: forced deduction	|
| -	askMatchType	|	int	  |	ask trade type: 0: ordinary trade, 1: liquidation trade, 2: forced deduction trade (bankrupt party) 3: forced deduction	|
| -	bidPnlType	|	int	  |	bid P&L type: 0: ordinary trade, 1: ordinary close, 2: liquidation, 3: forced deduction	|
| -	bidPnl	|	Double	  |	bid close P&L	|
| -	askPnlType	|	int	  |	ask P&L type: 0: ordinary trade, 1: ordinary close, 2: liquidation, 3: forced deduction	|
| -	askPnl	|	Double	  |	ask close P&L	|


## <span id="3">Get user's all trade records V2</span>
Get all trade records V2 version

#### The new interface API domain name address `https://futures.coinstore.com`  Call support for ApiKey

### HTTP Request:
- GET /api/v2/trade/order/queryHisMatch


> Response

```json
{
  "code" : 0,
  "data" : [
      {
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
      }
  ],
  "message" : "string"
}
```

### Request Parameters

|    code    | type   | required | comment             |
| ---------- |--------| -------- |---------------------|
| contractId| int    |N| contract id         |
| ordId| long      |N| order id             |
|pageNum| int | N | Current page number,default 1 |
|pageSize| int | N | Quantity per page,default  20 |
| side| int    |N| side -1 sell, 1 buy |

### Response Data

| code                 |  type  | comment                                                                                                                                 |
|----------------------| ------ |-----------------------------------------------------------------------------------------------------------------------------------------|
| code                 | int    | 0                                                                                                                                       |     0: success, other: failure                                               |
| message              | string    |                                                                                                                                         |    error message                                    |
| data                 | object|                                                                                                                                         |
| -	accountId	         | 	int	    | 	account id	                                                                                                                            |
| -	contractId	        | 	long	   | 	symbol ID, contract ID	                                                                                                                |
| -	orderId	           | 	Long	   | 	order ID	                                                                                                                              |
| -	clOrdId	           | 	string	 | 	Client delegate ID	                                                                                                                    |
| -	matchId	           | 	Long	   | 	match ID	                                                                                                                              |
| -	tradeId	           | 	Long	   | 	deal ID	                                                                                                                               |
| -	execSize	          | 	int	    | 	Transaction quantity	                                                                                                                  |
| -	matchAmt	          | 	Double	 | 	Transaction amount	                                                                                                                    |
| -	avgPrice	          | 	Double	 | 	Average transaction price	                                                                                                             |
| -	orderStatus	       | 	int	    | 	order status 0：NOT_DECLARE 1:IN_DECLARE 2:NOT_MATCH 3:PORTION_MATCH 4: ALL_MATCH 5：PORTION_CANCEL 6： ALL_CANCEL 7:IN_CANCEL 8:INVALID	 |
| -	orderRole	         | 	string	 | 	role：Taker,Maker		                                                                                                                     |
| -	remainingSize	     | 	int	    | 	remain quantity	                                                                                                                       |
| -	pnl	               | 	Double	 | 	Profit and loss	                                                                                                                       |
| -	fee	               | 	Double	 | 	Accumulated commission	                                                                                                                |
| -	matchTime	         | 	Long	   | 	match time	                                                                                                                            |


## <span id="3">User forced reduction</span>
Get all trade records

### HTTP Request: 
- GET /api/future/queryFlOrders


> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| contract id|
| endTime| string|N| end time |
| startTime| string|N| start time|
| pageNum| string|N| page number|
| pageSize| string|N| page size|
| side| string|N| side -1 Previous, 1 Next|

### Response Data

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | object|    |
| -	applId	|	int	  |	futures ID,2	|
| -	timestamp	|	Long	  |	order time	|
| -	contractId	|	Long	  |	symbol ID, contract ID	|
| -	userId	|	Long	  |	user ID	|
| -	uuid	|	String	  |	order ID	|
| -	side	|	int	  |	bid/ask side	|
| -	closeQty	|	int	  |	order quantity	|
| -	closePrice	|	Double	  |	order quantity	|
| -	closeAmt	|	Double	  |	order quantity	|
| -	filledCurrency	|	Double	  |	trade amount	|
| -	filledQuantity	|	int	  |	filled volume	|
| -	canceledQuantity	|	int	  |	canceled quantity	|
| -	orderStatus	|	int	  |	order status, 4: trade	|
| -	feeRatio	|	Double	  |	fee	|
| -	marginType	|	int	  |	margin type	|
| -	lossUserId	|	int	  |	loss user	|
| -	profitUserId	|	int	  |	 profit user	|
| -	execId	|	int	  |	trade ID	|
| -	lossMarginType	|	int	  |	loss user margin type	|
| -	profitMarginType	|	int	  |	profit user margin type	|

## <span id="3">User liquidation record</span>
Get all trade records

### HTTP Request: 
- GET /api/future/queryFcOrders


> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|N| contract id|
| endTime| string|N| end time |
| startTime| string|N| start time|
| pageNum| string|N| page number|
| pageSize| string|N| page size|
| side| string|N| side -1 Previous, 1 Next|

### Response Data

|       code        |  type  |                     comment                       |
| ----------------- | ------ |-------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | object|    |
| -	applId	|	int	  |	futures ID,2	|
| -	timestamp	|	Long	  |	order time	|
| -	contractId	|	Long	  |	symbol ID, contract ID	|
| -	userId	|	Long	  |	user ID	|
| -	uuid	|	String	  |	order ID	|
| -	side	|	int	  |	bid/ask side	|
| -	closeQty	|	int	  |	order quantity	|
| -	closePrice	|	Double	  |	order quantity	|
| -	closeAmt	|	Double	  |	order quantity	|
| -	filledCurrency	|	Double	  |	trade amount	|
| -	filledQuantity	|	int	  |	filled volume	|
| -	canceledQuantity	|	int	  |	canceled quantity	|
| -	orderStatus	|	int	  |	order status 4: trade	|
| -	feeRatio	|	Double	  |	fee	|
| -	marginType	|	int	  |	margin type	|
| -	lossUserId	|	int	  |	loss user	|
| -	profitUserId	|	int	  |	 profit user	|
| -	execId	|	int	  |	trade ID	|
| -	lossMarginType	|	int	  |	loss user margin type	|
| -	profitMarginType	|	int	  |	profit user margin type	|




# Ticker Related
## <span id="7">Get historical K-line</span>

### HTTP Request: 
- GET /v1/futureQuot/queryCandlestick

>Note: The return result is in time positive sequence

> Response

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


### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| contract id|
| range| string|Y| K-line type      1Min : 60000  3Min : 180000  5Min : 300000  15Min : 900000  30Min : 1800000  1Hour : 3600000  2Hour : 7200000  4Hour : 14400000  6Hour : 21600000  12Hour : 43200000  1Day : 86400000  1Week : 604800000 |
| end| string|N| deadline defaults to the current time, time of transmitting the first record, scrolling page turn|
| limit| string|N| number of query records, default: 1440 |

### Response Data: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0: success, other: failure                                               |
| message         | string    |    error message                                    |
| data | object|    |
| ├─lines| List[] | [timestamp, open price, high price, low price, close price, trade volume]   |

##  <span id="7">Query ticker snapshot</span>

### HTTP Request: 
- GET /v1/futureQuot/querySnapshot

> Response

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


### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| contract id|


### Response Data: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0: success, other: failure                                               |
| message         | string    |    error message                                    |
| data | object|    |
| ├─cid| Long| symbol id |
| ├─ip| Double|  index price|
| ├─mp|  Double| marked price |
| ├─lp| Double| latest price|
| ├─pv| Long| positions |
| ├─tv| Long|  24-hour trade volume |
| ├─tt| Long| 24-hour trade amount |
| ├─ph| Double|  24-hour high price |
| ├─pl| Double|  24-hour low price |
| ├─pcr |  Double| 24-hour change rate |
| ├─fr| Double|  fund rate |
| ├─pfr| Double| predicted fund rate |
| ├─bids| List | bids [price, conts] |
| ├─asks| List | bids [price, conts] |

##  <span id="7">Get latest trade</span>

### HTTP Request: 
- GET /v1/futureQuot/queryTickTrade

>Note: The return result is in time positive sequence

> Response

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


### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| contractId| int|Y| contract id|

### Response Data: 

|       code        |  type  |                      comment                       |
| ----------------- | ------ | -------------------------------------------------- |
| code            | int    |     0: success, other: failure                                               |
| message         | string    |    error message                                    |
| data | object|    |
| ├─trades| List[] | [timestamp, price, transaction quantity, side (1 long; -1 short)]   |

# Websocket Ticker Data

## Introduction

### Access URL

`
wss://ws-futures.coinstore.com/socket.io/?EIO=3&transport=websocket
`

### Authentication
Note: signature: in the same way as restful

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


### Subscription
The topic service is subscribed according to whether authentication is required. The following formula is the standard format of subscription, which is explained in detail in the relevant ws interface description

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
### Subscribe to Topic List
|topic	|	type	|	description	|	signature required	|
|-----|----|----|----|
|	future_tick	|	SUB	|	get trade data by transaction	|	No	|
|	future_kline	|	SUB	|	get K-line	|	No	|
|	future_snapshot_depth	|	SUB	|	get ticker snapshot and bid/ask level	|	No	|
|	indicator	|	SUB	|	get ticker data	|	No	|
|	match	|	SUB	|	get open orders, positions and assets	|	Yes	|


## Get K-line

### Subscribe to topic: future_kline
symbol value in the data format is: contract ID

### Ranges: 

* 1Min = 60000
* 5Min = 300000
* 15Min = 900000
* 30Min = 1800000
* 1Hour = 3600000
* 4Hour = 14400000
* 12Hour = 43200000
* 1Day = 86400000
* 1Week = 604800000

> The message subscription format is as follows:

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
	

### Return Structure
|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |contract ID               |
|range      | String    |  range             |
|lines      |object []  |    [ [${timestamp}, ${open price}, ${high price}, ${low price}, ${close price}, ${volume}] ]   |

## Get trade data by transaction

### Subscribe to topic: future_tick
Symbol value in the data format is: contract ID

> The message subscription format is as follows:

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

### Return Structure

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |contract ID               |
|trades      |object []  |    [ [${timestamp}, ${price}, ${volume}, ${side(1 long; 2 short)}] ] ]   |

## Get ticker snapshot and bid/ask level

### Subscribe to topic: future_snapshot_depth
Symbol value in the data format is: contract ID

> The message subscription format is as follows:

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

### Return Structure

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|contractId |long       |contract ID               |
|bids      |object []  | bid level data list   [ [${price}, ${volume}] ] ]   |
|asks      |object []  |  ask level data list  [ [${price}, ${volume}] ] ]   |

## Get basic data of ticker snapshot

### Subscribe to topic: indicator
Symbol value in the data format is: contract ID

> The message subscription format is as follows:

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

### Return Structure

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| indicators | List<Indicator>|  symbol ticker |
| markets |List<Market>|  ticker overview |
| account | List<Account>|  private data  |

**Note: Accounts is null when not logged in**
**Note: If params.symbols is null when subscribing, indicators is null**
**Note: Default interval of topic is 1s, and default interval of markets is 3s, that is, there are 2 push messages every 3 seconds in which markets is null**

#### Indicator

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid| Long| symbol id |
| ip| BigDecimal| index price|
| mp|  BigDecimal| marked price |
| lp| BigDecimal| latest price|
| pv| Long| positions |
| tv| Long| 24-hour trade volume |
| tt| Long| 24-hour trade amount |
| ph| BigDecimal| 24-hour high price |
| pl| BigDecimal| 24-hour low price |
| pcr |  BigDecimal| 24-hour change rate |
| fr| BigDecimal| fund rate |
| pfr| BigDecimal| predicted fund rate |

#### Market

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid| Long| symbol id |
| lp| BigDecimal| latest price |
| pcr |  BigDecimal|24-hour change rate |
| tv |  Long|  24-hour trade volume |
| tt| Long| 24-hour trade amount |
#### Account

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| aid | Long| account id  master account or sub-account |
| at| int| account type  1: master account  10: trial fund account |
| assets| List<Asset>| assets information |
| posis| List<Posi>| positions information  |

#### Asset

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| cid | Long| currency ID |
| avail| BigDecimal| available balance |
| upnl| BigDecimal| unrealized P&L in cross mode |

#### Posi

|    code    |  type   |      comment         |
|------------|---------|----------------------|
| pid | Long| position ID |
| flp| BigDecimal| liquidation price |
| upnl| BigDecimal| unrealized P&L |
| adl| int| ADL grade |
| mp|  BigDecimal| marked price |

**Note: pid is the id of 3012 push position**

## Get private data
### Subscribe to topic: match

> The message subscription format is as follows:

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

### Return Structure

#### Asset

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | message type 3002               |
|accountId      |long  | user ID  |
|currencyId      |long  |  currency ID  |
|totalBalance      |double  |  total balance  |
|available      |double  |  available balance  |
|frozenInitMargin      |double  |  order frozen margin  |
|initMargin      |double  |  occupied margin  |
|closeProfitLoss      |double  |  realized P&L  |

#### Positions

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | message type 3012               |
|accountId      |long  | user ID  |
|posis      |object[]  | position list |
| ├─contractId      |long  |  contract ID  |
| ├─marginType      |int  |  margin type, 1: cross, 2: isolated  |
| ├─initMarginRate      |String  |  initial margin rate  |
| ├─maintainMarginRate      |String  |  maintenance margin  |
| ├─initMargin      |String  | initial margin  |
| ├─extraMargin      |String  |  extra margin  |
| ├─openAmt      |String  |  open amount  |
| ├─openPrice      |String  |  average open price  |
| ├─posiQty      |String  |  positions, >0: long, <0: short  |
| ├─contractUnit      |String  |  contract unit  |
| ├─closeProfitLoss      |String  |  realized P&L  |
| ├─liquidationPrice      |double  |  estimated liquidation price  |
| ├─unRealizedProfit      |double  |  unrealized P&L  |

#### Position Configuration

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | message type 3022               |
|accountId      |long  | user ID  |
|positionConfgs      |object[]  | position configuration list |
| ├─contractId      |long  |  contract ID  |
| ├─leverage      |int  |  leverage rate, 0 represents automatic mode  |
| ├─isolated      |boolean  |  margin type, default false:cross, true:isolated|

#### Open Orders

|    code    |  type   |      comment         |
|------------|---------|----------------------|
|messageType |int       | message type 3004               |
|accountId      |long  | user ID  |
|contractId      |long  |  contract ID  |
|	orderId	|	string	|	order ID	|
|	clientOrderId	|	string	|	clients order ID	|
|	price	|	string	|	order price	|
|	quantity	|	string	|	order quantity	|
|	leftQuantity	|	string	|	remaining quantity	|
|	side	|	number	|	bid/ask side (1: bid, -1: ask）	|
|	placeTimestamp	|	number	|	order time	|
|	matchAmt	|	string	|	trade amount	|
|	matchQty	|	string	|	transaction quantity	|
|	orderType	|	number	|	contract order type (1: limit, 3: market）	|
|	positionEffect	|	number	|	position effect (1: open, 2: close）	|
|	marginType	|	number	|	margin type (1: cross, 2: isolated）	|
|	initMarginRate	|	string	|	initial margin rate	|
|	fcOrderId	|	string	|	liquidation order ID, liquidation order when null	|
|	markPrice	|	string	|	marked price	|
|	feeRate	|	string	|	fee rate	|
|	contractUnit	|	string	|	contract unit	|
|	orderStatus	|	int	|	order status 0-undeclared, 1-declaring, 2-declared but not filled, 3-partially filled, 4-fully filled, 5-partially canceled, 6-fully canceled, 7-canceling, 8-failure, 11-buffer the orders higher than limit, 12-buffer the orders lower than limit	|


# Error Code

### error Code:

| Code                                  | Comment                                    |
|---------------------------------------| ------------------------------------------ |
| signature-failed 401                  | Main account freezes sub-account login access                   |
| signature-failed 401                  | Sub-account has been deleted                   |
| signature-failed 401                  | Api key error                   |
| unauthorized 1401                     | Request IP is not in the IP whitelist      |
| unauthorized 1401                     | Signature failure                   |
| unauthorized 1401                     | User login Token expired                   |
| signature error 3005                  | Signature generation error                 |
| symbol not found 3011                 | The trading pair is not listed             |
| trade-limit 3013                      | You do not currently have the qualification for contract trading. If you have any questions, please contact customer support. We apologize for any inconvenience this may cause.                      |
| user-trade-limit 3014                 | The current sub-account temporarily lacks futures trading qualifications.                      |
| user-part-trade-limit 3015            | The current sub-account is restricted from trading the current futures currency pairs.                      |
| duplicate-order 3111                  | Duplicate order                            |
| sub-user-operator-business-error 6001 | Sub-account is not allowed to participate in this business   |
