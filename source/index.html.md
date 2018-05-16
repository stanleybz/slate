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
    'api-key': 'live_ly83rGP2Bo',
    'api-secret': 'tB2d71cDz4g1MjQlczdkxN2lLKO8BwlnJrsVh3O3'
  }
})
```

To identify you account when using the API you should including your API key in the request header, please aware some of the action need to pass the api-secret for processing with confidential information.

Please follow the badge under the endpoint to submit the correct credentials.

<span class="badge">BASIC</span> Just api-key
Just `api-key: live_ly83rGP2Bo`

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
     "fee_type":"HKD",
     "total_fee":"150",
     "open_id":"o3OAhO5yVZ7xAakJ3c5e13OAhO-M",
     "trade_type":"NATIVE"
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
```

### HTTP Request

You can create a new payment order by this endpoint.
Then you will retrieve a `wexin://xxx` deeplink for your customer to pay their order by redirection.

`[POST] /order`

### <span class="remark black">RB</span> Request Body

| Key | Description | Type | Options |
|----|----|----|----|
| out_trade_no | Order number, must be unique | integer | - |
| body | Your order description, will show on the payment page | string | - |
| fee_type | 3-letter ISO code for currency | string | HKD |
| total_fee | A fee in *cents* | int | - |
| open_id | Payment user openid, need to linked with the official account | string | - |
| trade_type | Order intergration method | string | NATIVE |
| notify_url | Notify URL for forwarding the XML response from wechat | string | - |

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

| Key | Description | Type | Options |
|----|----|----|----|
| out_trade_no | Order number | integer | - |
