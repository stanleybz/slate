---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

# includes:
  # - errors

search: true
---

# Introduction

WeCeive Pay official document

# Endpoint
All API requests must be made over HTTPS

Stable version: ```v1```

Production: ```https://api.pay.wemine.net/{version}/```

# Authentication

```javascript
var request = require("request");

request({
  headers: {
    'api-key': '{API Key}',
    'api-secret': '{API Secret}'
  }
})
```

To identify you account when using the API you should including your API key in the request header, please aware some of the action need to pass the api-secret for processing with confidential information.

Please follow the badge under the endpoint to submit the correct credentials.

<span class="badge">BASIC</span> Just api-key
Just `api-key: {API Key}`

<span class="badge orange">SECRET</span> api-key with api-secret

<aside class="warning">
Remember not to show any of you credentials on publicly accessible areas such client-side code(.js), GitHub, etc.
</aside>

# Wechat OpenID

The OpenID is a unique encrypted WeChat ID for each user of an official account, and users can have separate OpenIDs corresponding to different official accounts.

You must pass the correct OpenID when creating a new payment.

Here's the official step for reterive Wechat OpenID

[Wechat webpage authorization](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

## Wemine Quick Auth Service (oAuth)

If you've registered **Wemine Quick Auth Service**, you may use these simple step to get the user OpenID:

1) Redirect user to this domain with your ``quick_auth_token`` and ``redirect_url``:

``http://wemine.cn/wx-bridge/oauth_bridge.php?quick_auth={quick_auth_token}&redirect={redirect_url}``

2) Then you will retrieve and query string ``user_open_id`` from you redirect_url:

``http://example.com/?wx_openid={user_open_id}``

# Payment

## Create New Payment

<span class="badge">BASIC</span>

```javascript
var request = require("request");

var options = {
  method: 'POST',
  url: 'https://api.pay.wemine.net/v1/order',
  headers:
   {
     'Cache-Control': 'no-cache',
     'api-key': 'YOUR_API_KEY',
     'Content-Type': 'application/json'
   },
  body:
   {
     "notify_url":"http://api.pay.wemine.net/notify_url",
     "out_trade_no":"9900000000",
     "body":"Demo Order",
     "total_fee":"150",
     "open_id":"o3OAhO5yVZ7xAakJ3c5e13OAhO-M",
     "trade_type":"JSAPI"
   },
  json: true
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);
  console.log(body);
});

```

> The above command returns JSON structured like this:

```json
{
    "status": "success",
    "data": {
        "code_url": "weixin://wxpay/bizpayurl?pr=F2EDdjR",
        "out_trade_no": "9900000000"
    }
}
{
    "status": "success",
    "data": {
        "prepay_id": "wx241454271591128cfc753dab0139979466",
        "out_trade_no": "9900000000",
        "jsapi": {
          "appId": "wxa123b4567c8def90",
          "timeStamp": 1527135717,
          "nonceStr": "Yyj389AvflUTLi8vcSb04uUzVbFjINnw",
          "package": "prepay_id=wx241221577529528cfc753dab1804175993",
          "signType": "MD5",
          "paySign": "8C2B3EF54C5BF3E920DCCC2937571D00"
        }
    }
}
```

### HTTP Request

You can create a new payment order by this endpoint.
Then you will retrieve a `wexin://xxx` deeplink for your customer to pay their order by redirection.

`[POST] /order`

### <span class="remark black">RB</span> Request Body

| Key | Description | Type | Options | Required |
|----|----|----|----|----|
| out_trade_no | Order number, must be unique | integer | - | Required |
| body | Your order description, will show on the payment page | string | - | Required |
| total_fee | A fee in *cents* *(USD)* | int | - | Required |
| open_id | Payment user openid, need to linked with the official account | string | - | Required when trade_type is JSAPI |
| trade_type | Order intergration method | string | NATIVE, JSAPI | Required |
| notify_url | Notify URL for forwarding the XML response from wechat | string | - | Optional |

## Check Payment Status

<span class="badge orange">SECRET</span>

```javascript
var request = require("request");

var options = {
  method: 'GET',
  url: 'https://api.pay.wemine.net/v1/order',
  qs: { out_trade_no: '9900000000' },
  headers:
   {
     'Cache-Control': 'no-cache',
     'api-key': 'YOUR_API_KEY',
     'api-secret': 'YOU_API_SECRET'
     'Content-Type': 'application/json'
   }
  json: true
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);
  console.log(body);
});

```

> The above command returns JSON structured like this:

```json
{
    "status": "success",
    "data": {
        "openid": "oLYVCwxZrj6sWQFd4hyLWbdNgQ0U",
        "is_subscribe": "N",
        "trade_type": "NATIVE",
        "bank_type": "CFT",
        "total_fee": "1000",
        "fee_type": "USD",
        "transaction_id": "4200000049274123531243563212",
        "out_trade_no": "9900000000",
        "attach": [],
        "time_end": "20180107125107",
        "trade_state": "SUCCESS",
        "sub_openid": "oBtAL1f4zTZO4OpzvfAyLbZRCc-M",
        "sub_is_subscribe": "Y",
        "cash_fee": "10",
        "cash_fee_type": "CNY",
        "rate": "10000000"
    }
}
```

### HTTP Request

You can check you order status by passing the our_trade_no.

`[GET] /order`

### <span class="remark">QS</span> Query String

| Key | Description | Type | Options | Required |
|----|----|----|----|----|
| out_trade_no | Order number | integer | - | Required |

## Check Callback Sign

<span class="badge orange">SECRET</span>

```javascript
var request = require("request");

var options = {
  method: 'POST',
  url: 'https://api.pay.wemine.net/v1/checksign',
  body: '<xml>
   <appid><![CDATA[wxa123b4567c8def90]]></appid>
   <attach><![CDATA[支付测试]]></attach>
   <bank_type><![CDATA[CFT]]></bank_type>
   <fee_type><![CDATA[CNY]]></fee_type>
   <is_subscribe><![CDATA[Y]]></is_subscribe>
   <mch_id><![CDATA[1234567890]]></mch_id>
   <nonce_str><![CDATA[5d2b6c2a8db53831f7eda20af46e531c]]></nonce_str>
   <openid><![CDATA[oUpF8uMEb4qRXf22hE3X68TekukE]]></openid>
   <out_trade_no><![CDATA[1409811653]]></out_trade_no>
   <result_code><![CDATA[SUCCESS]]></result_code>
   <return_code><![CDATA[SUCCESS]]></return_code>
   <sign><![CDATA[B552ED6B279343CB493C5DD0D78AB241]]></sign>
   <sub_mch_id><![CDATA[1357924680]]></sub_mch_id>
   <time_end><![CDATA[20140903131540]]></time_end>
   <total_fee>1</total_fee>
   <trade_type><![CDATA[JSAPI]]></trade_type>
   <transaction_id><![CDATA[1004400740201409030005092168]]></transaction_id>
</xml>',
  headers:
   {
     'Cache-Control': 'no-cache',
     'api-key': 'YOUR_API_KEY',
     'api-secret': 'YOU_API_SECRET'
     'Content-Type': 'application/xml'
   }
  xml: true
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);
  console.log(body);
});

```

> The above command returns JSON structured like this:

```json
{
    "status": "success",
    "data": {
        "sign": "A1E38D5C4BF8A43FA8E57F2CB4284C9F"
    }
}
```

### HTTP Request

You can check you callback from notify_url if the sign is correct.

`[POST] /checksign`

### <span class="remark">Body</span> Body String

| Content | Description | Type | Required |
|----|----|----|----|----|
| XML body from the callback URL | To check if the callback body has correct XML | application/xml | Required |

## Request Refund

<span class="badge">BASIC</span>

```javascript
var request = require("request");

var options = {
  method: 'POST',
  url: 'https://api.pay.wemine.net/v1/refund',
  qs: { out_trade_no: '9900000000', 'total_fee':'1', 'refund_fee':'1' },
  headers:
   {
     'Cache-Control': 'no-cache',
     'api-key': 'YOUR_API_KEY',
     'Content-Type': 'application/json'
   }
  xml: true
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);
  console.log(body);
});

```

> The above command returns JSON structured like this:

```json
{
    "Code": "FAIL",
    "message": {
        "return_code": "SUCCESS",
        "return_msg": "OK",
        "appid": "wxa123b4567c8def90",
        "mch_id": "1234567890",
        "sub_mch_id": "1357924680",
        "nonce_str": "AN774wB0KB3IRN5Y",
        "sign": "19BCC434A48088A9A1C538BE827E4732",
        "result_code": "FAIL",
        "err_code": "ERROR",
        "err_code_des": "订单已全额退款"
    },
    "data": null
}
```

### HTTP Request

You can send a refund request to refund

`[POST] /refund`

### <span class="remark">Body</span> Body String

| Key | Description | Type | Options | Required |
|----|----|----|----|----|
| out_trade_no | Order number, must be unique | integer | - | Required |
| total_fee | A fee in *cents* | int | - | Required |
| refund_fee | Refund fee in *cents* | int | - | Required |
