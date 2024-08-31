---
title: Official API Document Of Coinstore

language_tabs: # must be one of https://git.io/vQNgJ
  

toc_footers:

includes:
  
language: English

other_language: 简体中文

present_url: /en

url: /cn

active: active

other_active: 

menu: 菜单

create_api: Create API KEY

spot_goods: Spot

spot_goods_active: active

spot_goods_url: 'index.html'

contract: Perpetual Swap

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

## Create APIkey

To use API, please log into the webpage first, create an API key through [User Center] - [API Managment], and then develop and trade according to the details of this documentation.
[You can click here to create an API Key]: https://www.coinstore.com/#/user/bindAuth/ManagementAPI

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

`https://api.coinstore.com/api`

**WebSocket**

`wss://ws.coinstore.com/s/ws`

To ensure the stability of API service, it is recommended to access using Japanese AWS cloud server. If the client server in Chinese mainland is used, it would be difficult to guarantee the stability of the connection.

## Rate Limit

**Same IP**

`A maximum of 300 requests is allowed every 3 seconds for the same IP.`

**Same User**

`For the same user, a maximum of 120 requests is allowed every 3 seconds.`

## <span id="a4">签名认证</span>

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

**Python Example**

 https://coinstore-sg-encryption.s3.ap-southeast-1.amazonaws.com/filesUpload/ex1/public/coinstore.py

 `It is recommended to use Python SDK version 3.9,using a version lower than 3.9 may lead to compatibility issues or inaccuracies in signature calculation.`


# API Access Instructions

## <span id="a3"> Request Format </span>
All API requests are restful, and there are only two methods at present: GET and POST.
- GET request: All parameters are in path parameters
- POST request: Parameters can be set in the path, and they can be sent in JSON format to the request body. If there are no parameters,{} needs to be sent

A licit request consists of the following parts:
- method request address: Access server address api.coinstore.com，e.g. https://api.coinstore.com/api/trade/order/place
- Required and optional parameters.
- X-CS-APIKEY: API Key applied by the user.
- X-CS-EXPIRES: Timestamp when you issued the request. For example:1629291143107.
- X-CS-SIGN: A string calculated from the signature, which is used to ensure that the signature is valid and not tampered with.

**Note: X-CS-APIKEY, X-CS-EXPIRES and X-CS-SIGN are all in the request header, and 'Content-Type':'application/json' 
needs to be set.**

## <span id="a3">Return Format</span>

All interface returns are in JSON format. There are three fields in the first layer of JSON: code, msg and data. The first two fields indicate the request status and description, and the actual business data is in the data field.

```json
{
    "code": "0",
    "msg": "suc",
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

| status code | description                             | remarks                       |
| ------ | -------------------------------- | -------------------------- |
| 0      | success                             | code=0 success, code >0 failure |


# Basic Information


## <span id="1">Currency & spot information</span>

Get currency & spot information

### HTTP Request:
- POST /v2/public/config/spot/symbols

### Request Params

| param | type  | required | comment     |
|-------|-------|----------|-------------|
|   symbolCodes    | array | no       | symbol Code |
|   symbolIds    | array    | no       | symbol ID   |

> Request

```json
{
  "symbolCodes":["BTCUSDT"],
  "symbolIds":[10]
}
```
### Response Data

| code                | type      | comment                            |
|---------------------|-----------|------------------------------------|
| code                | int       | 0：success, other: failure      |
| message             | String    | Error message           |
| data                | Object [] | Transaction data          |
| - symbolId          | Long      | Symbol  id           |
| - symbolCode        | String    | Symbol name         |
| - tradeCurrencyCode | String    | Base currency, e.g. BTC in BTC-USDT           |
| - quoteCurrencyCode | String    | Quote currency, e.g. USDT in BTC-USDT            |
| - openTrade         | boolean   |open trade:true, off trade:false |
| - onLineTime        | Long      |Listing time, Unix timestamp format in milliseconds, e.g. 1597026383085 |
| - tickSz            | Integer   |Tick size, e.g. 0 |
| - lotSz             | Integer   |Lot size, e.g. 1 |
| - minLmtPr          | String   |The minimum order price of the limit order |
| - minLmtSz          | String   |The minimum order quantity of the limit order |
| - minMktVa         | String   |The minimum order volume of the market order|
| - minMktSz         | String   |The minimum order quantity of the market order |
| - makerFee         | String   |Maker fee |
| - takerFee         | String   |Taker fee |


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

> response

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
## <span id="1">Basic information：</span>

Get currency information

### HTTPRequest:
- GET /fi/v1/common/currency

### Response Data

| param | type | required | comment       |
|-------|------|----------|---------------|
|   currencyCode    | string   | Y        | Currency Code |

> Request

```json
{
  "currencyCode":"ETH"
}
```

### response

| code                | type     | comment       |
|---------------------|----------|---------------|
| code                | int      | 0：success, other: failure    |
| message             | String   | Error message          |
| data                |          |           |
|-	name | String   | Ccurrency name|
|-   code | String   | Currency code|
|-   showPrecision | String   |    Lot size|
|-   chainDataList | Object[] | Chain List|
|--		currencyCode | String   | Currency code|
|--		chainName | String   | Chain name|
|--     chainProtocol  | String   |  Chain protocol|
|--		contractAddress | String   | Currency contract address|
|--		depositOpen  | String   | The availability to deposit from chain，0:not avaliable，1:avaliable|
|--		withdrawOpen | String   | The availability to withdraw to chain，0:not avaliable，1:avaliable|
|--		withdrawFee  | String   | The withdrawal fee|
|--		withdrawMin  | String   | The minimum on-chain withdrawal amount of currency in a single transaction|
|--		withdrawMax  | String   | The maximum amount of currency on-chain withdrawal in a single transaction|
|--		depositMin  | String   | The minimum deposit amount of currency in a single transaction|
|--		withdrawFeePlatform | String   | Platform fee rate|
|--		burningState | String   | If the currency is burncurrency,0:no,1:yes|
|--		withdrawFeeOnChain | String   | Burn rate on chain|
|--		tagType   | String   |   Whether tag/memo information is required for withdrawal, 0:no required,1:required,2:optional|
|--		withdrawFeeMin | String   | The minimum withdrawal fee|

> python

```python
import hashlib
import hmac
import json
import math
import time
import requests

# api get currency information

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
> response

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


# Account Related


## <span id="1"> Assets Balance </span>

Get user assets balance

### HTTP Request: 
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

> Response 

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

### Request Parameters

| param         | type   | required | comment      |
|---------------|--------|----------|--------------|
| symbolCodes   | array  | no | Currency pair code |
| symbolIds     | array  | no | Currency pair ID   |


### Response Data

|       code        |  type  |        example        |                      comment                       |
| ---- | ----- | ---------- |---------- |
| code | String      | |            |
| message | String  | | Error Message |
| data | Object []  | |  Transaction Data |
|- userId | Long  |  |User id |
|- accountId | Long  | | Account id |
|- currencyId | Integer  | | Currency id |
|- balance | Decimal  | | Amoun |
|- type | Decimal | |Account status 1: available 4: deactivated |


# Funding Related

## <span id="2">Get deposit address</span>
Get deposit address information

### HTTP request:
- POST /fi/v3/asset/deposit/do

### Request Params

|  code |  type  |  required  |comment|
| ---- | ----- |---------- |-----|
|currencyCode	|string|	Y|	Currency code|
|chain	|string|	Y|	Chain name|

> Request

```json
{
  "currencyCode": "USDTTRX",
  "chain": "TRX"
}
```

### Response Data

|       code        |  type  |                  comment                       |
| ---- | ----- |---------- |
|code	|int|	0：success, other: failure|
|message	|string|	Error message|
|data	|object|	|
|-	address	|string|	Deposit address|
|-	tag	|string|	Tag/memo|
|-	depositMin	|string|	The minimum deposit amount of currency in a single transaction|

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
> Response Data

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


## <span id="2">Get deposit history</span>
Get deposit history

### HTTP request:
- POST /fi/v3/asset/deposit/record/list

### Request

|  code |  type  |  required  | comment                                                                |
| ---- | ----- |---------- |------------------------------------------------------------------------|
|currencyCode |string| Y| Currency code                                                          |
|startDate |string| N| Start Time, format ‘yyyy-MM-dd HH:mm:ss’                               |
|endDate |string| N| End Time, format ’yyyy-MM-dd HH:mm:ss‘                                 |
|fromId |long| N| Deposit ID，default starts from 1                                       |
|limit |int| N| Number of results per request. The maximum is 1000; The default is 500 |
|externalId |string| N| External Id                                                            |
|label |string| N| Label                                                                  |


> Request

```json
{
  "currencyCode": "USDT",
  "startDate": "2024-03-01",
  "endDate": "2024-03-08",
  "fromId": "",
  "limit": "10",
  "externalId": "",
  "label": ""
}

```

### Response Data

|       code        |  type  |                  comment                      |
| ---- | ----- |--------- |
|code	|type|	comment|
|code	|int|	0：success, other: failure|
|message	|String|	Error message|
|data|	|  |
|	externalId |string| ExternalId|
|	label |string| Label|
|	depositList |Object []|	Data|
|--	 id	|long|	ID|
|--	 symbol	|string|	Currency code|
|--	 chainName	|string|	Chain name|
|--	 chainProtocol	|string|	Chain protocol|
|--	 amount	|string|	Amount|
|--	 createdAt	|string|	Time|
|--	 status	|int|	Deposit status, 0:Pending，1:Completed，2:Abnormal，3:Under Review，4:Rejected|
|--	 explorer	|string|	The website of explorer|
|--	 txId	|string|	TxID|
|--	 internalType	|string|	Deposit type 0：On-chain 1：Internal transfer|
|--	 lockAccount	|boolean|	Lockup period|

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
> Response Data

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
## <span id="2">Get withdrawal history</span>
Get withdrawal history

### HTTP Request:
- POST /fi/v3/asset/withdraw/record/list

### Request Data

|  code |  type  | required |comment|
| ---- | ----- |----------|-----|
|currencyCode |string| N        | Currency code|
|startDate |string| N        | Start Time, format ‘yyyy-MM-dd HH:mm:ss’|
|endDate |string| N        | End Time, format ’yyyy-MM-dd HH:mm:ss‘|
|fromId |long| N        | Withdrawal ID，default starts from 1|
|limit |int| N        | Number of results per request. The maximum is 1000; The default is 500|
|externalId |string| N        | ExternalId|
|label |string| N        | Label|


> Request

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

### Response Data

|       code        |  type  | comment                                                                                                                         |
| ---- | ----- |---------------------------------------------------------------------------------------------------------------------------------|
|code	|type| 	comment                                                                                                                        |
|code	|int| 	0：success, other: failure                                                                                                      |
|message	|string| 	Error message                                                                                                                  |
|data	| |                                                                                                                                 |
|	externalId |string| ExternalId                                                                                                                      |
|	label |string| Label                                                                                                                           |
|	withdrawList |Object[]| 	Data                                                                                                                           |
|--	 id	|long| 	ID                                                                                                                             |
|--	 symbol	|string| 	Currency code                                                                                                                  |
|--	 amount	|string| 	Amount                                                                                                                         |
|--	 fee	|string| 	Fee                                                                                                                            |
|--	 createdAt	|string| 	Time                                                                                                                           |
|--	 status	|int| 	Withdrawal status, 0： Not reviewed, 1:Approved, 2:Rejected, 3:Payment in progress, 4:Payment failed, 5:Completed, 6:Cancelled. |
|--	 explorer	|string| 	The website of explorer                                                                                                        |
|--	 txId	|string| 	TxID                                                                                                                           |
|--	 addressTo	|string| 	Receiving address                                                                                                              |
|--	 chainName	|string| 	Chain Name                                                                                                                     |
|--	 chainProtocol	|string| 	Chain protocol                                                                                                                 |
|--	 internalType	|int| Withdrawal type 0：On-chain 1：Internal transfer                                                                                                                                |

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
> Response

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

## <span id="2">Withdrawal</span>
Withdrawal of tokens. Common sub-account does not support withdrawal;The API can only make withdrawal to verified addresses/account, and verified addresses can be set by WEB/APP.


### HTTP Request:
- POST /fi/v3/asset/doWithdraw

### Request Data

|  code |  type  | required   |comment|
| ---- | ----- |------------|-----|
|param	|type|	required|	comment|
|currencyCode |string| Y| Currency code|
|amount |string| Y| Withdrawal amount|
|address |string| Y| Should be a trusted address|
|tag |string| N| Tag/memo|
|chainType |string| Y| Chain protocol eg:trc20 bnbbsc erc20 sol|


> Request

```json
{
  "currencyCode": "USDT",
  "amount": "1000",
  "address": "TU4vEruvZwLLkSfV9bNw12EJTPvNr7Pvaa",
  "tag": "",
  "chainType": "trc20"
}
```

### Response

|       code        |  type  |                  comment                     |
| ---- | ----- |--------- |
|code	|type|	comment|
|code	|int|	0：success, other: failure|
|message	|String|	Error message|
|data|||	
|--	 id	|Long|	Withdrawal ID|

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
> Response

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "id": 4886
  }
}
```

## <span id="2">Cancel withdrawal</span>
Cancel withdrawal

### HTTP Request:
- POST /fi/v3/asset/cancelWithdraw

### Request

|  code |  type  | required   |comment|
| ---- | ----- |------------|-----|
|withdrawId |long| Y| Withdrawal ID|


> Request

```json
{
  "withdrawId": 4886
}
```

### Response Data

|       code        | type   |                  comment                      |
| ---- |--------|--------- |
|code	| type   |	comment|
|code	| int    |	0：success, other: failure|
|message	| String |	Error message|
|data	|        ||
|--	 id	| Long   |	Withdrawal ID|

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
> Response

```json
{
  "code": "0",
  "message": "Success",
  "data": {
    "id": 4886
  }
}
```

## <span id="2">Fund transfer</span>
Fund transfer

#### Currently supporting contract<->spot transfer, API domain name address https://futures.api.coinstore.com/api Call support for ApiKey


### HTTP Request:
- POST /v1/future/transfer

### Request

|  code |  type  | required   | comment                                                                                  |
| ---- | ----- |------------|------------------------------------------------------------------------------------------|
|param	|type|	required| 	comment                                                                                 |
|type |string| Y| Transfer type <br/>0：Transfer within account<br/>1：Master account to sub-account (Only applicable to API Key from master account) <br/>2：Sub-account to master account (Only applicable to API Key from master account)<br/>The default is 0 |
|currencyCode |string| Y| Currrency code                                                                   |
|amount |string| Y | Transfer amount                                                                                     |
|from |int| Y | The remitting account , 1：Spot account 2：Futures account                                                                      |
|to |int| Y | The beneficiary account , 1：Spot account 2：Futures account                                                                       |
|subAccount |string| N| Name of the sub-account,When type is 1/2, this parameter is required.                                                                  |


> Request

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

### Response

|       code        | type   |                  comment                      |
| ---- |--------|--------- |
|code	|type|	comment|
|code	|int|	0：success, other: failure|
|message	|string|	Error message|

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
> Response

```json
{
  "code": "0",
  "message": "Success"
}
```
> Response Data Fail:

```json
{
  "code": 101040502,
  "msg": "request field missing",
  "data": null
}
```

#### Other response code description


|       code       | remark                                                                                              |
| ---- |-----------------------------------------------------------------------------------------------------|
| 200| Transfer succeeded                                                                                  |
| 102150400| Incorrect transfer quantity                                                                         |
| 101040502| Possible incorrect currency name, user-defined ID, transfer type, and unsupported transfer currency |
| 102150808| Possible insufficient balance, duplicate orders                                                     |
| 102150400| Transfer Fail                                                                                       |




# Order Related

## <span id="2"> Get current orders </span>

Get current order

### HTTP Request: 

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

> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|  |  |  |  |


### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data            |  list  |    | |
| ├─ ordId            | long   | 11594964764657880     |   order id            |
| ├─ clOrdId            | string   | 7980b2ebf32042ba9dbfbddc555a3985     |   client order id            |
| ├─symbol | string |  |  token pair |
| ├─baseCurrency| string |  | base currency|
| ├─quoteCurrency| string |  | quot currency|
| ├─ordPrice | string |  |  order price|
| ├─ordQty| string |  | order quantity|
| ├─ordAmt| string |  | order amount|
| ├─side | string |  |  BUY,SELL|
| ├─cumAmt| string |  | |
| ├─cumQty| string |  | |
| ├─leavesQty| string |  | |
| ├─ordStatus | string |  | NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─ordType | string |  |  MARKET,LIMIT,POST_ONLY|
| ├─timeInForce| string | GTC | GTC,IOC,FOK |
| ├─timestamp| long |  | |


## <span id="2"> Get current orders V2 </span>

Get current order v2 version

#### The new interface API domain name address `https://api.coinstore.com`  Call support for ApiKey


### HTTP Request:

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



> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | N      | token pair     |
|ordId| int    | N      | Delegate ID    |
|clOrdId| String | N      | Client delegate ID  |


### Response Data

|       code        |  type  | example | comment                                                       |
| ----------------- | ------ |---------|---------------------------------------------------------------|
| code            | int    | 0       | 0: success, other: failure                                    |
| message         | string    |         | error message                                                 |
| data            |  list  |         |                                                               |
| ├─baseCurrency| string |         | base currency                                                 |
| ├─quoteCurrency| string |         | quot currency                                                 |
| ├─timeInForce| string |   GTC      | GTC,IOC,FOK                                                   |
| ├─ordStatus | string |   SUBMITTED      | NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 	       | Average transaction price                                     |
| ├─ordPrice | string |         | order price                                                   |
| ├─leavesQty| string |         |
| ├─ordType | string | MARKET  | MARKET,LIMIT,POST_ONLY                                        |
| ├─ordQty| string |         | order quantity|
| ├─cumAmt| string |         | Transaction amount|
| ├─side | string | BUY      | BUY,SELL|
| ├─cumQty| string |         | Transaction quantity|
| ├─ ordId      | long  |         | order id|
| ├─ clOrdId      | string     |         | Client delegate ID|
| ├─symbol | string |         | token pair|
| ├─timestamp| long |         |


## <span id="3"> Get user's latest trade </span>
Get all trading records

### HTTP Request: 
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

> Response

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

### Request Parameters

|    code    |  type   | required | comment                       |
| ---------- | ------- | -------- |-------------------------------|
|symbol | string | Y | symbol                        |
|ordId| int | N | Delegate ID                   |
|pageNum| int | N | Current page number,default 1 |
|pageSize| int | N | Quantity per page,default  20 |
|side| int | N | direction -1:SELL 1:BUY       |

### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | object|  | |
| ├─accountId| long| | |
| ├─instrumentId| int | | |
| ├─baseCurrencyId| int | | |
| ├─quoteCurrencyId| int | | |
| ├─matchRole| int | | TAKER(1),MAKER(-1) |
| ├─feeCurrencyId| int | | |
| ├─role| string | | TAKER,MAKER |
| ├─execQty| string | | |
| ├─orderState| int | | |
| ├─matchId| long | | |
| ├─orderId| long | | |
| ├─side| int | | BUY(1), SELL(-1) |
| ├─execAmt| string | | |
| ├─selfDealingQty| string | | |
| ├─tradeId| long | | |
| ├─fee| string | | |
| ├─matchTime| long  | | |
| ├─remainingQty| string | | |


##  <span id="4"> Cancel orders </span>
Cancel orders

### HTTP Request: 
- POST /trade/order/cancel

> Request Body

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

> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | true | |
| ordId     | long     | true  |  |

### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data            |    |      |   return data                                                 |
| ├─state| string|  | |
| ├─ordId| long |  | |

##  <span id="5"> One-click cancellation </span>


### HTTP Request: 
- POST /trade/order/cancelAll

> Request Body

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

> Response

```json
{
    "code": 0
}
```

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol     | string | true| Cancel all orders for the specified trading pair |

### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |


## <span id="6"> Create order </span>
Create order

#### Regular user accounts are only allowed to hold up to 50 active orders at the same time, while market-making accounts are not subject to this restriction.If you are a market maker but do not have permission to add a market-making account, please contact the Coinstore delivery department.

> Request Body

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


> Response

```json
{
    "code": "0",
    "data": {
        "order_id":11594964764657880
    }
}
```

### HTTP Request: 
- POST /trade/order/place

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| clOrdId | string | false    |   client order ID. Unique customer-defined order ID. If not sent, it is automatically generated  |
| symbol        | string | true    |  |
| side   | string    | true    |   BUY,SELL    |
| ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| timeInForce    | string     | false    | defaultValue = "GTC"   |
| ordPrice    | long     | false    | limit order required |
| ordQty    | long     | false    | limit order required, market price sell order required |
| ordAmt    | long     | false    | market price buy order required |
| timestamp    | long     | true    |  |



### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data            |     |      |   return data                                                 |
| ├─ ordId            | long   | 11594964764657880     |   order id                                                 |

## <span id="13"> Batch ordering </span>
Batch ordering


#### Regular user accounts are only allowed to hold up to 50 active orders at the same time, while market-making accounts are not subject to this restriction.If you are a market maker but do not have permission to add a market-making account, please contact the Coinstore delivery department.


> Request Body

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
        {"side":"BUY","ordType":"LIMIT","ordPrice":28000,"ordQty":"2"},
 {"side":"BUY","ordType":"LIMIT","ordPrice":30000,"ordQty":"1"}],
"timestamp":expires})
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


> Response

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

### HTTP Request: 
- POST /trade/order/placeBatch

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
| symbol        | string | true    |  |
| timestamp    | long     | true    |  |
| orders        | list | true    |  |
| ├─ clOrdId | string | false    |   client order ID. Unique customer-defined order ID. If not sent, it is automatically generated  |
| ├─ side   | string    | true    |   BUY,SELL    |
| ├─ ordType     | string    | true    |  MARKET,LIMIT,POST_ONLY   |
| ├─ timeInForce    | string     | false    | defaultValue = "GTC"   |
| ├─ ordPrice    | long     | false    | limit order required |
| ├─ ordQty    | long     | false    | limit order required, market price sell order required |
| ├─ ordAmt    | long     | false    | market price buy order required |



### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data            |  list  |      |   return data                                                 |
| ├─ ordId            | long   | 11594964764657880     |   order id            |
| ├─ clOrdId            | string   | 7980b2ebf32042ba9dbfbddc555a3985     |   client order id            |
| ├─errno| int|   |  |
| ├─errMsg| string| | |

## <span id="27"> Batch cancellation according to order id </span>
Batch cancellation according to order id.

### HTTP Request: 
- POST trade/order/cancelBatch

> Request Body

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
payload = json.dumps({"symbol": "BTCUSDT"})
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


> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | Y |  "BTCUSDT"|
|orderIds| LIst| Y |[1,2,3]  |




### Response Data

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code         | string    |0                |    normal return information                                |
| message         | string    |order.id-required                 |    error message                                   |
| data            | Object   |                    |                                             |
| --success|   List   | [1,2,3]                    |     set of successful order ids                                         |
| --reject  |    List   | [1,2,3]                                  |  set of failed order ids.                      |


## <span id="14">Get order information </span>
Get order information


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


> Response

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

### HTTP Request: 
- GET trade/order/orderInfo

### Request Parameters:

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|ordId | long | false  | order id |

### Response Data:

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | object|  | |
| ├─ordId| long |  | |
| ├─clOrdId| string |  | |
| ├─accountId| long |  | |
| ├─symbol | string |  |   |
| ├─baseCurrency| string |  | |
| ├─quoteCurrency| string |  | |
| ├─ordPrice | string |  | |
| ├─ordQty| string |  | |
| ├─ordAmt| string |  | |
| ├─side | string |  | BUY,SELL|
| ├─cumAmt| string |  | |
| ├─cumQty| string |  | |
| ├─leavesQty| string |  | |
| ├─ordState | string |  | NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─ordType | string |   | MARKET,LIMIT,POST_ONLY |
| ├─timeInForce| string | GTC |   GTC,IOC,FOK |
| ├─timestamp| long |  | |
| ├─avgPrice| long |  | |


## <span id="14">Get order information V2 </span>
Get order information v2

#### The new interface API domain name address `https://api.coinstore.com`  Call support for ApiKey


### HTTP Request:

- GET /api/v2/trade/order/orderInfo


> python 

```python
import hashlib
import hmac
import math
import time
import requests
url = "https://api.coinstore.com/api/v2/trade/order/orderInfo?ordId=1780715084580128"
api_key=b'da2b7ed9aeee00744193d251bc99a09c'
secret_key = b'28fd41afcb4bdcb752634b1549ad9aa7'
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

> Response

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

### Request Parameters

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|ordId| int    | N      | Order id (client order number 1 of 2)   |
|clOrdId| String | N      | Client order number (order id optional)  |


### Response Data

|       code        |  type  | example       |                      comment                       |
| ----------------- | ------ |---------------| -------------------------------------------------- |
| code            | int    | 0             |     0: success, other: failure                                               |
| message         | string    |               |    error message                                    |
| data            |  list  |               | |
| ├─baseCurrency| string |        | base currency                                                 |
| ├─quoteCurrency| string |        | quot currency                                                 |
| ├─timeInForce| string |   GTC  | GTC,IOC,FOK                                                   |
| ├─ordStatus | string |   SUBMITTED | NOT_FOUND,SUBMITTING,SUBMITTED,PARTIAL_FILLED,CANCELED,FILLED |
| ├─avgPrice | string | 	      | Average transaction price                                     |
| ├─ordPrice | string |        | order price                                                   |
| ├─leavesQty| string |        |
| ├─ordType | string | MARKET | MARKET,LIMIT,POST_ONLY                                        |
| ├─ordQty| string |        | order quantity|
| ├─cumAmt| string |        | Transaction amount|
| ├─side | string | BUY    | BUY,SELL|
| ├─cumQty| string |        | Transaction quantity|
| ├─ ordId      | long  |        | order id|
| ├─ clOrdId      | string     |        | Client delegate ID|
| ├─symbol | string |        | token pair|
| ├─orderUpdateTime| long |        | The time of the last transaction or Order time|
| ├─timestamp| long |        | |


# Ticker Related

## <span id="7">Ticker</span>
 Ticker for all trading pairs in the market

> Response
 
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

### HTTP request:
- GET /v1/market/tickers

### Request parameters:

| code | type | required | comment |
| ---------- | ------- | -------- | -------------------- |

### Response data:

| code | type | comment |
| ----------------- | ------ | ---------------------------------------- |
| code | int | 0: success, other failure |
| message | string | Error message |
| data | Object| |
| ├─channel| String |Identifier |
| ├─instrumentId| Integer |Pair ID |
| ├─symbol| String |Trading pair |
| ├─count | Integer |24h rolling transaction count |
| ├─volume|String |24h transaction volume measured in quote currency |
| ├─amount| String |24h transaction volume measured in base currency |
| ├─close| String | Last transaction price|
| ├─open| String | First transaction price|
| ├─high| String | The highest transaction price|
| ├─low| String | The lowest transaction price|
| ├─bid| String | Buy one price|
| ├─bidSize| String | Buy a quantity|
| ├─ask| String | Sell one price|
| ├─askSize| String | Amount to sell|



 
## <span id="7">Get depth</span>
Get depth data

> Response

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
 
### HTTP request:
- GET /v1/market/depth/{symbol}

### Request parameters:

| code | type | required | comment |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | trading pair, such as "BTCUSDT" |
|depth | int | false | The number of depths, such as "5, 10, 20, 50, 100", default 20 |

### Response data:

| code | type | comment |
| ----------------- | ------ | ---------------------------------------- |
| code | int | 0: success, other failure |
| message | string | Error message |
| data | Object| |
| ├─channel| String |Identifier |
| ├─level| Integer |Quantity |
| ├─instrumentId| Integer |Pair ID |
| ├─symbol| string |Trading pair |
| ├─a | List | Sell order, [${price}, ${quantity}, ${direction}] ] |
| ├─b | List | Buy order, [${price}, ${quantity}, ${direction}] ] |


## <span id="7">Get K line</span>
Get trading K line

> Response

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
 
### HTTP request:
- GET /v1/market/kline/{symbol}

### Request parameters:

| code | type | required | comment |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | trading pair, such as "BTCUSDT" |
|period | String | false | Data granularity, such as "1min, 5min, 15min, 30min, 60min, 4hour, 12hour, 1day, 1week", the default is 20 |
|size | Integer | false | Number of data bars, [1,2000] |

### Response data:

| code | type | comment |
| ----------------- | ------ | ---------------------------------------- |
| code | int | 0: success, other failure |
| message | string | Error message |
| data | Object| |
| ├─channel| String |Identifier |
| ├─instrumentId| Integer |Pair ID |
| ├─symbol| String |Trading pair |
| ├─item | List |K-line collection |
| ├─├─interval | String |K line interval |
| ├─├─endTime| Long |End time, unit s |
| ├─├─amount|String |Amount |
| ├─├─startTime|Long |Start time, unit s|
| ├─├─firstTradeId|Long |First Trade ID |
| ├─├─lastTradeId|Long |Last Trade ID |
| ├─├─volume| String |Volume|
| ├─├─close| String | Last transaction price|
| ├─├─open| String | First transaction price|
| ├─├─high| String |Highest transaction price |
| ├─├─low| String |Lowest transaction price |



## <span id="7">Latest trades</span>
Get the latest trades record

> Response
 
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
  
### HTTP request:
- GET /v1/market/trade/{symbol}

### Request parameters:

| code | type | required | comment |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | trading pair, such as "BTCUSDT" |
|size | Integer | false | Number of data bars, [1,100] |

### Response data:

| code | type | comment |
| ----------------- | ------ | ---------------------------------------- |
| code | int | 0: success, other failure |
| message | string | Error message |
| data | Object| |
| ├─channel| String |Identifier |
| ├─instrumentId| Integer |Pair ID |
| ├─symbol| String |Trading pair |
| ├─interval | String |K line interval |
| ├─time| Long |Time, unit s |
| ├─ts|Long |Time, unit ms |
| ├─tradeId|Long |The ID of the first transaction |
| ├─takerSide|String |Direction "BUY", "SELL" |
| ├─volume| String |Volume |
| ├─price| String | Last transaction price|                      
 
 

  
##  <span id="7"> Get the latest price of all symbols </span>
Get the latest price of all symbols

### HTTP Request: 
- GET /v1/ticker/price


> Response

```json
{
  "code": 0,
  "message": "",
  "data": [
    {
      "id": 1,
      "symbol": "btcusdt",
      "price": "400"
    },
    {
      "id": 4,
      "symbol": "autusdt",
      "price": "1"
    }
  ]
}
```


### Request Parameters: 

|    code    |  type   | required |       comment        |
| ---------- | ------- | -------- | -------------------- |
|symbol | string | true | Trading pairs, case-sensitive, comma-separated |

**default: Query all symbols**

**query the specified symbol: /v1/ticker/price;symbol=btcusdt,eosusdt,autusdt**

### Response Data: 

|       code        |  type  |        example        |                      comment                       |
| ----------------- | ------ | --------------------- | -------------------------------------------------- |
| code            | int    | 0                     |     0: success, other: failure                                               |
| message         | string    |                 |    error message                                    |
| data | List[]|  | |
| ├─id| long |  | |
| ├─symbol| string |  |symbol|
| ├─price| string |  |price |



#Websocket Ticker Data

##  **Introduction**

### Access URL
wss://ws.coinstore.com/s/ws

1. The baseurl of all wss interfaces is: wss://<host:port>/s/ws

2. All symbols in stream name are lowercase

3. Each link is valid for no more than 24 hours. Please handle the disconnection and reconnection properly.

4. The server will send a ping frame every 3 minutes, and the clients should reply to the pong frame within 10 minutes, otherwise, the server will actively disconnect the link. Clients are allowed to send unpaired pong frames (that is, clients can send pong frames at a frequency higher than 10 minutes each time to maintain the link).



### Description of Server Push Data Type

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

##### Request-response message type

* `echo`: The server will return a message for each message received in the same form to confirm that the message has been received.
* `resp`: The server will return a message of processing result for each message received in the same form, marking the message processing. The return message includes the processing result.

##### subscribe message type
the following types of messages start from executing the `SUB` command of the corresponding channel and end with executing the `UNSUB` command, and the messages are pushed if the data on the server side changes, with a minimum push interval of 100ms.
```lang=json
{
    "S": 1,
    "T": "kline|ticker|depth|trade|account",
    ...:  // see below for details on each data subscription format 
}
```
* `kline`

* `ticker`

* `depth`

* `trade`

* `account`

* `order`


### Pong
the server supports two forms of end pong response, namely websocket pong frame and pong message.

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

## **Subscribe/unsubscribe data stream in real time**

> Subscribe an information stream

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

> Unsubscribe an information stream

```lang=json
{
    "op": "UNSUB",
    "channel": [
        "btcusdt@depth"
    ],
    "id": 312
}
```

> Subscribed information stream

```lang=json
{
    "op": "LIST",
    "channel": [],
    "id": 3
}
```

* The following data can be sent via websocket to subscribe or unsubscribe the data stream. Examples are as follows.
* The id in the response content is an unsigned integer, which is the unique identifier of the current information.
* If the result in the corresponding content is null, it means that the request was sent successfully.

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

> NOTE: `<symbol>` Please use the id of the symbol for the parameter temporarily, and the name of the symbol can be used later.

> IMPORTANT:  Please use the symbol' field first, and the instrumentId' is marked as Deprecated'.

> NOTE:  All the time to return data is in `seconds'.


## **Trade by Trade**

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
> full-volume data:

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
> incremental data:

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",    // symbol
  "tradeId": 12345,       // trading ID
  "takerSide": "BUY",    // taker side
  "price": "0.001",     // strike price
  "volume": "100",       // trading volume
  "time": 123456785,   // trading time unit: s
  "ts": 1612685313400, // unit: ms
  "seq": 9935         // unique auto-increment serial number
}
```

### Stream Name
 `<symbol>@trade`
 
 eg: `4@trade`
### param
`param": {"size":2}`





## **K-line Streams**
K-line stream pushes the requested k-line type (the latest k-line) every second.

> Send

```lang=json
 {"op":"SUB","channel":["4@kline@min_1"],"id":1}
```

> Receive

```lang=json
{
   "instrumentId": 88066, 
   "startTime": 1603732500,  // start time of this k-line, unit: s
    "endTime": 1603732559,  // start time of this k-line, unit: s
    "symbol": "USDTBTC",  // symbol
    "interval": "1m",      // K-line interval
    "firstTradeId": 100,       // first trade ID during this k-line period
    "lastTradeId": 200,       // last trade ID during this k-line period
    "open": "0.0010",  // first strike price during this k-line period
    "close": "0.0020",  // last strike price during this K-line period
    "high": "0.0025",  // highest strike price during this K line period
    "low": "0.0015",  // lowest strike price during this K line period
    "volume": "1000",    // trading volume during this K line period
    "amount": "1.0000",  // trading amount during this k-line period
}
```

### Stream Name
`<symbol>@kline@<interval>`

eg: `4@kline@min_1`


### interval optional values
* min_1
* min_5
* min_15
* min_30
* hour_1
* hour_4
* hour_12
* day_1
* week_1
* mon_1



## **K-line Request**
request historical k-line

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

### param

* `channel`: `<symbol>@kline@<interval>` ,eg:88066@kline@min_1
* `limit`: No more than 200
* `endTime`: Optional, exclusive




## **Ticker Information by Symbol**
Streamlined ticker information in the last 24 hours refreshed by Symbol

> Send

```lang=json
 {"op":"SUB","channel":["4@ticker"],"id":1}
```

> Receive

```lang=json
{
  "instrumentId": 88066,
  "symbol": "USDTBTC",      // symbol
  "open": "0.0010",      // first strike price counted backwards 24 hours ago
  "high": "0.0025",      // highest strike price within 24 hours
  "low": "0.0010",      // lowest strike price within 24 hours
  "close": "0.0025",      // latest strike price
  "volume": "10000",       // trading volume within 24 hours
  "amount": "18"          // trading amount within 24 hours
}
```

### Stream Name
`<symbol>@ticker`, eg:  `4@ticker`


## **Ticker information of all symbols in the whole market**
Streamlined ticker information in the last 24 hours refreshed by Symbol

### Stream Name
`!@ticker`

> Send

```lang=json
 {"op":"SUB","channel":["!@ticker"],"id":1}
```

> Receive

```lang=json
{
    "data": [
        {
            "channel": "80002@ticker",
            "open": "9996.24",
            "high": "10048.6601",
            "low": "9768.90178",
            "close": "9833.12538",
            "volume": "3362.405",
            "amount": "33212664.57170620732",
            "symbol": "BTCUSD", 
            "instrumentId": 80002
        },
        {
            "channel": "80001@ticker",
            "open": "10001.57",
            "high": "10053.9441",
            "low": "9816.23078",
            "close": "9832.91038",
            "volume": "3138.93361771",
            "amount": "31040843.0757471861414",
            "symbol": "ETHUSD",
            "instrumentId": 80001
        },
        ...
    ]
}
```

## **Depth information**
Push limited level depth information every second or every 100ms. 

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

### levels optional values
Levels indicates the levels of the buy or sell orders, and 5/10/20/50/100 level can be selected.

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
| part-trade-limit 3012                 | You are currently restricted from trading this spot currency pair. If you have any questions, please contact customer support. We apologize for any inconvenience this may cause.                      |
| trade-limit 3013                      | You do not currently have the qualification for spot trading. If you have any questions, please contact customer support. We apologize for any inconvenience this may cause.                      |
| user-trade-limit 3014                 | The current sub-account temporarily lacks spot trading qualifications                      |
| user-part-trade-limit 3015            | The current sub-account is restricted from trading the current spot currency pair                      |
| account-insufficient 3113             | Insufficient account balance            |
| duplicate-order 3111                  | Duplicate order                            |
| order-not-found 3103                  | Order not found or does not belong to the current account |
| sub-user-operator-business-error 6001 | Sub-account is not allowed to participate in this business   |


## **Dictionary**

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

