---
title: 聚合支付 Icbcmo

categories:
- 架构设计
- 支付开发

date: 2021-04-27 00:00:56
---
公司官网：https://www.icbc.com.mo
开发文档：https://app.icbcmo.site/gitbook Login name: ebyte Password: ebyte

工銀澳門聚合支付 Plus。

## 发起支付
#### 微信小程序支付
```java
public void order(Long orderId, String createIp) {
   try {
      // 1. 校验支付配置
      icbcmoProperties.valid();

      // 2. 查询订单数据
      Order order = orderFeign.one(attr(Order::getMerchantId,
                                          Order::getMemberId,
                                          Order::getCode,
                                          Order::getPayment),
                                    id(orderId));

      // 3. 查询用户OpenId
      String openId = memberFeign.openId(order.getMemberId());
      if (StringUtils.isEmpty(openId)) {
            throw new ServiceException(I18n2.warn_pay_1000001.locale());
      }

      // 4. 查询商户支付数据
      IcbcmoPay icbcmoPay = merchantFeign.icbcmoPay(order.getMerchantId());

      // 5. 支付金额
      String amount = String.valueOf(order.getPayment().multiply(BigDecimal.valueOf(100)).intValue());

      TreeMap<String, String> params = new TreeMap<>();
      params.put("merchantId", icbcmoPay.getMerchantId());            // （必填）商戶號
      params.put("merchantTid", icbcmoPay.getMerchantTid());          // （选填）商戶設備ID只適用於微信/支付寶
      params.put("channel", "WechatMiniApp");                         // （必填）取值範圍請參考支付請求 channel 參數信息表
      params.put("merchantOrderId", order.getCode());                 // （必填）商戶訂單ID
      params.put("merchantUserId", "");                               // （选填）商户的客户ID（仅线上POS使用）
      params.put("currency", "MOP");                                  // （必填）貨幣（固定MOP）
      params.put("amount", amount);                                   // （必填）額（分）
      params.put("timeout", "300");                                   // （必填）超時時間（秒）（需要設置白名單才有效，該字段用來設置支付超時時間，沒有設置該值的情況，該值默認900秒，如果設置該值，選擇渠道為線上POS，超時支付之後系統會自動沖正退款）
      params.put("notifyUrl", icbcmoProperties.getNotifyurl());       // （选填）通知支付接口
      params.put("returnUrl", icbcmoProperties.getReturnUrl());       // （必填）前端回調接口
      params.put("channelExt", weixinChannelExt(icbcmoPay, openId));  // （选填）渠道擴展字段, 必須為JSON格式
      params.put("merchantExt", "");                                  // （选填）商戶擴展字段該為保留字段, 無任何限制, 內容不會被處理
      params.put("sign", sign(icbcmoPay.getMd5Key(), params));        // （必填）簽名用於校驗參數是否被篡改

      // 6. 转换成为请求参数 a=1&b=2&c=3&sign=4
      String queryStr = queryStr(params);

      // 7. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
      String request = icbcmoProperties.getUrl() + "/api/v1/order?" + queryStr;

      // 8. 请求接口
      log.info("IcbcmoService.order Request: " + request);
      String response = Request.Post(request)
                                 .addHeader("Content-Type", "application/x-www-form-urlencoded")
                                 .execute()
                                 .returnContent().toString();
      log.info("IcbcmoService.order Response: " + response);

      // 9. 结果处理  {"code":"9998","retCode":"500","msg":"channel 請求參數不合法"}
      JSONObject responseJSON = JSONObject.parseObject(response);
      if (!responseJSON.getString("retCode").equals("200")) {
            throw new ServiceException(responseJSON.getString("msg"));
      }
   } catch (ServiceException e) {
      throw e;
   } catch (Exception e) {
      throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
   }
}
```

## 支付通知

## 支付查询

## 发起退款

## 退款通知

## 退款查询

## 关闭订单


## 通用方法
#### 将 Map 转换成为请求参数
```java
/**
* 将Map转换成为请求参数
* {"a":1, "b": 2} -> a=2&b=2
*/
private static String queryStr(Map<String, String> signMap) {
   List<String> keys = new ArrayList<>(signMap.keySet());
   Collections.sort(keys);

   StringBuilder sb = new StringBuilder();
   for (String key : keys) {
      String value = signMap.get(key);
      if (StringUtils.isEmpty(value)) continue;
      try {
            value = URLEncoder.encode(value, "utf-8");
      } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
      }
      sb.append(key).append("=").append(value).append("&");
   }
   if (sb.length() > 0) {
      sb.deleteCharAt(sb.length() - 1);
   }
   System.out.println("dataStr: " + sb);
   return sb.toString();
}
```

#### 计算签名
```java
private static String sign(String privateKey, Map<String, String> signMap) {
   String dataStr = queryStr(signMap);
   return Md5Utils.getMD5((dataStr + privateKey).getBytes());
}
```

#### 渠道擴展字段
参考：https://app.icbcmo.site/gitbook/cn/chap4/chap4.4.2.html

```java
/**
* 微信参数表
*/
private static String weixinChannelExt(IcbcmoPay icbcmoPay, String openId) {
   if (icbcmoPay == null || icbcmoPay.getWeixinChannelExt() == null || StringUtils.isEmpty(openId)) {
      throw new ServiceException(BaseResultEnum.ERROR, "数据异常");
   }
   JSONObject params = new JSONObject();
   params.put("body", "");                                             // 訂單購物明細，非必填。如不填入則系統自動填入商家簡稱。
   params.put("custom_display_code", "");                              // 是否商家自建支付頁面或使用系統支付頁面。商家自建支付頁面: Y系統支付頁面: N (默認)
   params.put("wx_appid", icbcmoPay.getWeixinChannelExt().getAppId()); // 商戶在微信開放平臺註冊生成的appid。渠道為微信APP和微信小程序時必填。
   params.put("openid", openId);                                       // 商戶在微信開放平臺註冊生成的appid。渠道為微信APP和微信小程序時必填。
   return params.toJSONString();
}
```


## 响应数据
#### 正常响应
```json
// 微信扫码支付返回
{
   "retCode":"200",
   "returnObj":
   {
      "merchantId":"011901200001001",
      "merchantTid":"11",
      "channel":"WechatPayPC",
      "merchantOrderId":"ICBC_AIO_20190711_091944",
      "icbcOrderId":"011WE20190711091957823988639",
      "currency":"MOP",
      "amount":"100",
      "redirectUrl":"https://mobilepaytest.icbc.com.mo/api/payment/NativePayPage.aspx?orderno=90120000100100000011190711538371&sign=3661b8d0975563e85dcc9df3ce94b065&time=120"
   },
   "sign":"c5a7ec6aa5c57eb6b4fa0aa41a2bb26a"
}
// 微信小程序支付返回
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011900000000002", 
        "merchantTid": "00000011", 
        "channel": "WechatMiniApp", 
        "merchantOrderId": "10000058160843", 
        "icbcOrderId": "011WE20210603183143021474642", 
        "currency": "MOP", 
        "amount": "5", 
        "respCode": "00", 
        "respMsgDetail": "SUCCESS", 
        "sign": "D1E5B79FF8569EFD5CF1BFF42481817C", 
        "appId": "wx47d7859ff4ec85c4", 
        "nonceStr": "BD686FD640BE98EFAAE0091FA301E613", 
        "signType": "MD5", 
        "timeStamp": "1622716302", 
        "package": "prepay_id=wx031831441262294383aa020140228c0000"
    }, 
    "sign": "2f553aa3ab9e44c516d8172cd5c720a8"
}
```

#### 异常响应
```json
{
    "code": "9998", 
    "retCode": "500", 
    "msg": "channel 請求參數不合法"
}
{
    "code": "1091", 
    "retCode": "500", 
    "msg": "簽名信息有誤, Signature information is incorrect"
}
{
    "code": "1094", 
    "retCode": "500", 
    "msg": "訂單號重複, Duplicate order number"
}
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011999010002500", 
        "merchantTid": "00000001", 
        "channel": "WechatMiniApp", 
        "merchantOrderId": "100000581", 
        "icbcOrderId": "001WE20210602105446369294200", 
        "currency": "MOP", 
        "amount": "5", 
        "respCode": "42", 
        "respMsgDetail": "參數錯誤，請根據文檔上送參數[小程式尺幅需要填入參數openid]"
    }, 
    "sign": "970360aa375bdebc7e7ab2dccc68b719"
}

{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011999010002500", 
        "merchantTid": "00000001", 
        "channel": "WechatMiniApp", 
        "merchantOrderId": "10000058182137", 
        "icbcOrderId": "001WE20210602111424424911969", 
        "currency": "MOP", 
        "amount": "5", 
        "respCode": "32", 
        "respMsgDetail": "生成預付單失敗，請聯繫機構檢查", 
        "wxErrCode": "FAIL", 
        "wxErrCodeDesc": "sub_mch_id与sub_appid不匹配"
    }, 
    "sign": "0477f49c3ab4645085c3455afce9c91e"
}

```