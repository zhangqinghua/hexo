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

## 发起支付请求
参考：https://app.icbcmo.site/gitbook/cn/chap4/chap4.html

#### 微信小程序支付
```java
public PayInfo orderWechatMiniApp(Long orderId) {
    // 1. 查询订单
    Order order = orderFeign.one(attr(Order::getMerchantId, Order::getMemberId, Order::getCode, Order::getPayment), id(orderId));

    // 2. 查询支付配置
    ReceiptConfig.IcbcmoWeixinJsapi icbcmoWeixinJsapi = receiptConfigFeign.icbcmoWeixinJsapi(order.getMerchantId());
    IcbcmoConfig icbcmoConfig = BeanMapper.map(icbcmoWeixinJsapi, IcbcmoConfig.class);

    // 3. 查询支付的微信openId
    String openId = memberFeign.openId(order.getMemberId());
    if (StringUtils.isEmpty(openId)) {
        throw new ServiceException(I18n2.warn_pay_1000001.locale());
    }

    // 4. 发起支付请求
    return order(order, icbcmoConfig, "WechatMiniApp", weixinChannelExt(icbcmoWeixinJsapi.getWx_appid(), openId));
}
```

#### 信用卡支付
```java
@Override
public PayInfo orderIcbcOnlinePosOrder(Long orderId) {
    // 1. 查询订单
    Order order = orderFeign.one(attr(Order::getMerchantId, Order::getCode, Order::getPayment), id(orderId));

    // 2. 查询支付配置
    ReceiptConfig.IcbcmoPos icbcmoPos = receiptConfigFeign.icbcmoPos(order.getMerchantId());
    IcbcmoConfig icbcmoConfig = BeanMapper.map(icbcmoPos, IcbcmoConfig.class);

    // 3. 发起支付请求
    return order(order, icbcmoConfig, "ICBCOnlinePosOrder", postChannelExt());
}
```

#### 支付请求
```java
private PayInfo order(Order order, IcbcmoConfig icbcmoConfig, String channel, String channelExt) {
    try {
        // 1. 校验配置
        icbcmoProperties.valid();
        // 2. 一些字段
        String amount = String.valueOf(order.getPayment().multiply(BigDecimal.valueOf(100)).intValue());
        String md5Key = icbcmoConfig.getMd5Key();
        String merchantOrderId = order.getCode();
        String returnUrl = icbcmoProperties.getReturnUrl();
        String notifyUrl = icbcmoProperties.getOrderNotifySite() + "/api/icbcmo/pay/notify";

        // 4. 请求参数
        TreeMap<String, String> params = new TreeMap<>();
        params.put("merchantId", icbcmoConfig.getMerchantId());         // （必填）商戶號
        params.put("merchantTid", icbcmoConfig.getMerchantTid());       // （选填）商戶設備ID只適用於微信/支付寶
        params.put("channel", channel);                                 // （必填）取值範圍請參考支付請求 channel 參數信息表
        params.put("merchantOrderId", merchantOrderId);                 // （必填）商戶訂單ID
        params.put("merchantUserId", "");                               // （选填）商户的客户ID（仅线上POS使用）
        params.put("currency", "MOP");                                  // （必填）貨幣（固定MOP）
        params.put("amount", amount);                                   // （必填）額（分）
        params.put("timeout", "900");                                   // （必填）超時時間（秒）（需要設置白名單才有效，該字段用來設置支付超時時間，沒有設置該值的情況，該值默認900秒，如果設置該值，選擇渠道為線上POS，超時支付之後系統會自動沖正退款）
        params.put("notifyUrl", notifyUrl);                             // （选填）通知支付接口
        params.put("returnUrl", returnUrl);                             // （必填）前端回調接口
        params.put("channelExt", channelExt);                           // （选填）渠道擴展字段, 必須為JSON格式
        params.put("merchantExt", "");                                  // （选填）商戶擴展字段該為保留字段, 無任何限制, 內容不會被處理
        params.put("sign", sign(md5Key, params));                       // （必填）簽名用於校驗參數是否被篡改

        // 5. 转换成为请求参数 a=1&b=2&c=3&sign=4
        String queryStr = queryStr(params);

        // 6. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
        String request = icbcmoProperties.getUrl() + "/api/v1/order?" + queryStr;

        // 7. 请求接口
        log.info("IcbcmoService.order Request: " + request);
        String response = Request.Post(request)
                                    .addHeader("Content-Type", "application/x-www-form-urlencoded")
                                    .execute()
                                    .returnContent().toString();
        log.info("IcbcmoService.order Response: " + response);

        // 8. 结果处理  {"code":"9998","retCode":"500","msg":"channel 請求參數不合法"}
        JSONObject responseJSON = JSONObject.parseObject(response);
        if (!responseJSON.getString("retCode").equals("200")) {
            throw new ServiceException(responseJSON.getString("msg"));
        }
        // 8.1 微信小程序支付
        JSONObject returnObj = responseJSON.getJSONObject("returnObj");
        if (returnObj.getString("channel").equals("WechatMiniApp")) {
            if (!returnObj.getString("respCode").equals("00")) {
                throw new ServiceException(returnObj.getString("respMsgDetail"));
            }
            PayInfo payInfo = new PayInfo();
            PayInfo.WeixinJSAPI weixinJSAPI = new PayInfo.WeixinJSAPI();
            weixinJSAPI.setAppId(returnObj.getString("appId"));
            weixinJSAPI.setPaySign(returnObj.getString("sign"));
            weixinJSAPI.setSignType(returnObj.getString("signType"));
            weixinJSAPI.setNonceStr(returnObj.getString("nonceStr"));
            weixinJSAPI.setTimestamp(returnObj.getString("timeStamp"));
            weixinJSAPI.setPackageDesc(returnObj.getString("package"));
            payInfo.setWeixinJSAPI(weixinJSAPI);
            return payInfo;
        }
        // 8.2 POS支付
        if (returnObj.getString("channel").equals("ICBCOnlinePosOrder")) {
            PayInfo payInfo = new PayInfo();
            PayInfo.Pos pos = new PayInfo.Pos();
            pos.setRedirectUrl(returnObj.getString("redirectUrl"));
            pos.setAgentRedirectUrl(replaceDomainAndPort(icbcmoProperties.getAgentPosPaySite(), pos.getRedirectUrl()));
            payInfo.setPos(pos);
            return payInfo;
        }
        // 8.3 其它异常
        throw new ServiceException(BaseResultEnum.ERROR, "Unknow Error");
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 支付结果异步通知
参考：https://app.icbcmo.site/gitbook/cn/chap6/chap6.html

#### 异步通知参数
**1. 异常参数**
原始回调数据：

```
http://dev.easybyte-hk.com/api/icbcmo/pay/notify&amount=5&channel=ICBCOnlinePosOrder&channelTransId=HFG000000000000012264583&currency=MOP&icbcOrderId=001IC20210610174515370150881&merchantExt=%7B%22remark1%22%3A%22625018******1479%7C6%7C2%7C0%22%2C%22remark2%22%3A%22%3F%3F%3F%3F%22%7D&merchantId=011901200001005&merchantOrderId=20210610170740444269&merchantTid=00000001&result=Error&resultMsg=93000128%7C%E8%B6%85%E4%B8%83%E6%97%A5%E6%B6%88%E8%B4%B9%E7%AC%94%E6%95%B0%E7%B4%AF%E8%AE%A1+%E8%AF%B7%E8%AE%B0%E5%BD%95%E9%94%99%E8%AF%AF%E4%BB%A3%E7%A0%81%EF%BC%8C%E5%8F%8A%E6%97%B6%E4%B8%8E%E6%88%91%E8%A1%8C%E5%AE%A2%E6%88%B7%E6%9C%8D%E5%8A%A1%E9%83%A8%E9%97%A8%E8%81%94%E7%B3%BB%E3%80%82&sign=aab1cab2f80e48fa4af680a5d788038c
```

解析后的参数：

```
merchantExt={"remark1":"625018******1479|6|2|0","remark2":"????"}, 
result=Error, 
amount=5, 
merchantId=011901200001005, 
channel=ICBCOnlinePosOrder, 
sign=aab1cab2f80e48fa4af680a5d788038c, 
currency=MOP, 
merchantOrderId=20210610170740444269, 
icbcOrderId=001IC20210610174515370150881, 
merchantTid=00000001, 
channelTransId=HFG000000000000012264583, 
resultMsg=93000128|超七日消费笔数累计 请记录错误代码，及时与我行客户服务部门联系。
```

```
merchantExt={"remark1":"625018******1479|6|2|0","remark2":"????"}, 
result=Error, 
amount=7, 
merchantId=011901200001005, 
channel=ICBCOnlinePosOrder, 
sign=4110e3ffb1975ce1c99135383df643ee, 
currency=MOP, 
merchantOrderId=20210621114823813766, 
icbcOrderId=001IC20210621114826045985355, 
merchantTid=00000001, 
channelTransId=HFG000000000000012265945, 
resultMsg=8268|系統錯誤，請聯繫工行 請記錄錯誤代碼，及時與我行客戶服務部門聯繫
```

**1. 微信支付回调**

原始回调数据：

```
http://dev.easybyte-hk.com/api/icbcmo/pay/notify&amount=5&channel=WechatMiniApp&channelTransId=4200001159202106101341217912&currency=MOP&icbcOrderId=011WE20210610103250045862338&merchantId=011900000000002&merchantOrderId=20210610103246168335&merchantTid=00000011&result=Success&resultMsg=success&sign=005e94348fd2deffa5836b26461f21e8
```

解析后的参数：

```
result=Success, 
amount=5, 
merchantId=011900000000002, 
channel=WechatMiniApp, 
sign=005e94348fd2deffa5836b26461f21e8, 
currency=MOP, 
merchantOrderId=20210610103246168335, 
icbcOrderId=011WE20210610103250045862338, 
merchantTid=00000011, 
channelTransId=4200001159202106101341217912, 
resultMsg=success
```

**2. POS 支付回调**

原始回调数据：

```
http://dev.easybyte-hk.com/api/icbcmo/pay/notify&amount=1205&channel=ICBCOnlinePosOrder&channelTransId=HFG000000000000012264401&currency=MOP&icbcOrderId=001IC20210610144458507737894&merchantExt=%7B%22remark1%22%3A%22625018******6308%7C6%7C2%7C0%22%2C%22remark2%22%3A%22%3F%3F%3F%3F%22%7D&merchantId=011901200001005&merchantOrderId=20210610144456805106&merchantTid=00000001&result=Success&resultMsg=0%7C&sign=bf00d2e0d9f03da7d205b4a2a40b7343
```

解析后的参数：

```
merchantExt={"remark1":"625018******6308|6|2|0","remark2":"????"}, 
result=Success, 
amount=1205, 
merchantId=011901200001005, 
channel=ICBCOnlinePosOrder, 
sign=bf00d2e0d9f03da7d205b4a2a40b7343, 
currency=MOP, 
merchantOrderId=20210610144456805106, 
icbcOrderId=001IC20210610144458507737894, 
merchantTid=00000001, 
channelTransId=HFG000000000000012264401, 
resultMsg=0|
```

#### 异步通知接收方法
```java
@GetMapping("/pay/notify")
public void payNotify(HttpServletRequest request, HttpServletResponse response) throws IOException {
    // 1. 解析回调参数
    log.info("ICBCMO 原始支付回调参数：{}{}{}", request.getRequestURL(), "&", request.getQueryString());
    Enumeration<String> names = request.getParameterNames();
    Map<String, String> params = new HashMap<>();
    while (names.hasMoreElements()) {
        String name = names.nextElement();
        String value = request.getParameter(name);
        params.put(name, value);
    }

    // 2. 处理支付回调 todo 应该异步处理
    try {
        log.info("ICBCMO 解析支付回调参数：{}", params);
        icbcmoService.syncOrderNotiry(params);
    } catch (Exception e) {
        log.error("ICBCMO 处理支付回调异常：{}", e.getMessage(), e);
    }

    // 3. 返回success以確認完成通知成功（不管有没有处理成功）
    @Cleanup
    PrintWriter printWriter = response.getWriter();
    printWriter.write("success");
    printWriter.flush();
    printWriter.close();
}
```

#### 异步通知处理方法
```java
@Override
public void syncOrderNotiry(Map<String, String> params) {
    try {
        // 1. 校验参数
        if (!"Success".equals(params.get("result"))) {
            throw new ServiceException(BaseResultEnum.ERROR, "result异常：" + params.get("result"));
        }
        // 2. 渠道类型、支付类型（WechatMiniApp）
        String channel = params.get("channel");
        // 3. 商户订单号（20210610103246168335）
        String orderCode = params.get("merchantOrderId");

        // 4. 计算支付类型
        PayEnum payEnum = "WechatMiniApp".equals(params.get("channel")) ? PayEnum.weixin :
                            "ICBCOnlinePosOrder".equals(params.get("channel")) ? PayEnum.pos :
                            null;
        if (payEnum == null) throw new ServiceException(BaseResultEnum.ERROR, "非法的支付类型：" + params.get("channel"));

        // 5. 设置订单支付成功
        orderFeign.paid(orderCode, payEnum, PayAgent.icbcmo, LocalDateTime.now());
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 发起主动查询
参考：https://app.icbcmo.site/gitbook/cn/chap6/chap6.html

#### 请求参数
```bash
https://app.icbcmo.site/api/v1/query?merchantId=011900000000002&merchantOrderId=20210604155233849229&sign=c5696f38c847aa0b53b9e105f04bf985
```

#### 响应数据
**1. 异常响应**
```json
{
    "code": "1095", 
    "retCode": "500", 
    "msg": "查無此訂單，請核實, Check this order, please verify"
}

{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "merchantTid": "00000001", 
        "merchantOrderId": "20210610170740444269", 
        "icbcOrderId": "001IC20210610174515370150881", 
        "channelTransId": "HFG000000000000012264583", 
        "customer_user_id": "", 
        "orderStatus": "Error", 
        "orderStatusDesc": "", 
        "channel": "ICBCOnlinePosOrder", 
        "orderCurrency": "MOP", 
        "orderAmount": "5", 
        "discountAmount": "0", 
        "cashCurrency": "81", 
        "cashAmount": "5", 
        "bankType": "", 
        "receipt_amount": "", 
        "tradeTime": "2021-06-10 17:46:40", 
        "errorCode": "0", 
        "errorMessage": "", 
        "cardNo": "625018******1479", 
        "cardType": "", 
        "merchantExt": "{\"remark1\":\"625018******6308|6|2|0\",\"remark2\":\"????\"}"
    }, 
    "sign": "399c4fc7950259040790dd422abc97f0"
}
```


**2. 支付宝响应**

```json
{
  "retCode": "200",
  "returnObj": {
    "merchantId": "011901200001001",
    "merchantTid": "00000011",
    "merchantOrderId": "ICBC_AIO_20190710_233100",
    "icbcOrderId": "011AL20190710233158379876488",
    "channelTransId": "2019071022001330371041428517",
    "orderStatus": "Success",
    "orderStatusDesc": "支付成功",
    "channel": "AlipayWEB",
    "orderCurrency": "MOP",
    "orderAmount": "5",
    "cashCurrency": "CNY",
    "cashAmount": "4",
    "bankType": "",
    "tradeTime": "2019-07-10 23:32:21",
    "errorCode": "",
    "errorMessage": ""
  },
  "sign": "4e349f7fa5ec47f5321ab967edc6bc8c"
}
```

**3. 微信支付响应**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011900000000002", 
        "merchantTid": "00000011", 
        "merchantOrderId": "20210607103609472187", 
        "icbcOrderId": "011WE20210607103617064565574", 
        "channelTransId": "4200001030202106070520800740", 
        "customer_user_id": "", 
        "orderStatus": "Success", 
        "orderStatusDesc": "支付成功", 
        "channel": "WechatMiniApp", 
        "orderCurrency": "MOP", 
        "orderAmount": "6", 
        "discountAmount": "0", 
        "cashCurrency": "CNY", 
        "cashAmount": "4", 
        "bankType": "PAB_CREDIT", 
        "receipt_amount": "", 
        "tradeTime": "2021-06-07 10:37:18", 
        "errorCode": "", 
        "errorMessage": "", 
        "cardNo": "", 
        "cardType": ""
    }, 
    "sign": "e8080adfff9d1aeaec416948abf299fa"
}
```

**4. POS 支付响应**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "merchantTid": "00000001", 
        "merchantOrderId": "20210610144456805106", 
        "icbcOrderId": "001IC20210610144458507737894", 
        "channelTransId": "HFG000000000000012264401", 
        "customer_user_id": "", 
        "orderStatus": "Success", 
        "orderStatusDesc": "", 
        "channel": "ICBCOnlinePosOrder", 
        "orderCurrency": "MOP", 
        "orderAmount": "1205", 
        "discountAmount": "0", 
        "cashCurrency": "81", 
        "cashAmount": "1205", 
        "bankType": "", 
        "receipt_amount": "", 
        "tradeTime": "2021-06-10 14:45:37", 
        "errorCode": "0", 
        "errorMessage": "", 
        "cardNo": "625018******6308", 
        "cardType": "", 
        "merchantExt": "{\"remark1\":\"625018******6308|6|2|0\",\"remark2\":\"????\"}"
    }, 
    "sign": "f9caf885ca2c5d5b985a88bcdf4f2a13"
}
```

#### 请求方法
```java
@Override
public TradeQuery query(Long orderId) {
    // 1. 查询订单数据
    Order order = orderFeign.one(id(orderId), attr(Order::getMerchantId, Order::getCode));
    // 2. 查询支付配置（可能进行多次查询，因为不清楚此订单是使用哪种方式进行支付）
    List<IcbcmoConfig> icbcmoConfigs = BeanMapper.map(receiptConfigFeign.icbcmos(order.getMerchantId()), IcbcmoConfig.class);
    for (IcbcmoConfig icbcmoConfig : icbcmoConfigs) {
        TradeQuery tradeQuery = query(order, icbcmoConfig);
        if (tradeQuery.getTradeState() == TradeStatus.SUCCESS) return tradeQuery;
    }
    return TradeQuery.notpay();
}

private TradeQuery query(Order order, IcbcmoConfig icbcmoConfig) {
    try {
        // 3. 组装请求参数
        Map<String, String> params = new HashMap<>();
        params.put("merchantId", icbcmoConfig.getMerchantId());         // （必填）商戶號
        params.put("icbcOrderId", "");                                  // （选填）ICBC訂單ID註: icbcOrderId, merchantOrderId: 2選1。處理次序為icbcOrderIdàmerchantOrderId
        params.put("merchantOrderId", order.getCode());                 // （选填）商戶訂單ID 註: icbcOrderId, merchantOrderId: 2選1。處理次序為icbcOrderIdàmerchantOrderId
        params.put("sign", sign(icbcmoConfig.getMd5Key(), params));     // （必填）簽名用於校驗參數是否被篡改

        // 4. 转换成为请求参数 a=1&b=2&c=3&sign=4
        String queryStr = queryStr(params);

        // 5. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
        String request = icbcmoProperties.getUrl() + "/api/v1/query?" + queryStr;

        // 6 请求接口
        log.info("IcbcmoService.query Request: \n" + request);
        String response = Request.Get(request)
                                    .execute()
                                    .returnContent().toString();
        log.info("IcbcmoService.query Response: \n" + response);

        // 7. 响应数据校验
        JSONObject responseJSON = JSONObject.parseObject(response);

        if ("200".equals(responseJSON.getString("retCode"))) {
            JSONObject returnObj = responseJSON.getJSONObject("returnObj");
            // 7.1 其它状态都算未支付（Create、Wait、Processing、Close、Error、Refund、Reverse）
            if (!"Success".equals(returnObj.getString("orderStatus"))) {
                return TradeQuery.notpay();
            }
            // 7.2 支付成功
            if ("Success".equals(returnObj.getString("orderStatus"))) {
                PayEnum payEnum = "AlipayWEB".equals(returnObj.getString("channel")) ? PayEnum.alipay :
                                    "WechatMiniApp".equals(returnObj.getString("channel")) ? PayEnum.weixin :
                                    PayEnum.pos;
                return TradeQuery.success(payEnum,
                                            PayAgent.icbcmo,
                                            LocalDateTime.parse(returnObj.getString("tradeTime"),
                                                                DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
            }
        }

        // 7.3 订单不存在
        if ("1095".equals(responseJSON.getString("code"))) {
            return TradeQuery.notExists();
        }

        // 7.4 其它异常
        throw new ServiceException(BaseResultEnum.ERROR, "Unknow Error");
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 发起退款申请
参考：https://app.icbcmo.site/gitbook/cn/chap7/chap7.html

发起退款申请，微信及支付宝订单为异步接口，需要通过 notifyUrl 异步通知或查询接口确认退款结果。线上 POS 及 e 支付为实时接口，退款接口报文会直接返回退款结果。

#### 上送参数
```
http://202.175.59.29:10443/gwinternet/app-svc/api/v1/refund/initRefund?currency=MOP&merchantId=011901200001005&merchantOrderId=20210610112716186705&merchantRefundOrderId=20210610113232421507&notifyUrl=https%3A%2F%2Fdev.easybyte-hk.com%2Fapi%2Ftest&originAmount=1205&refundAmount=1205&sign=8d7eb9883518e6b33ea1661947e2c4f1
```

#### 返回参数

**1. 异常参数**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011900000000002", 
        "refundStatus": "Fail", 
        "refundStatusDesc": "申请退款失败，商家无权限"
    }, 
    "sign": "2ec7ca078ca1c3f5bafdf467ce794c7a"
}

{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011900000000002", 
        "refundStatus": "Fail", 
        "refundStatusDesc": "申请退款失败，商家单号无法在系统中找到原交易,请联系机构处理"
    }, 
    "sign": "722f348b8bfa035c5185bcc6b2589ee5"
}
```

**2. 正常参数**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011900000000002", 
        "refundStatus": "Applied", 
        "refundStatusDesc": "退款申请成功，请等待系统后台处理"
    }, 
    "sign": "3d86ba5224988993293d4417ca9c697b"
}

{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "refundStatus": "Success", 
        "refundStatusDesc": "退货成功", 
        "refundRealCurrency": "81", 
        "refundRealAmount": "1205"
    }, 
    "sign": "e84585523e82e2680da518720496dc83"
}
```

#### 上送方法
```java
@Override
public void initRefund(Long refundId) {
    try {
        // 1. 查询退款订单
        Refund refund = refundFeign.one(id(refundId), attr(Refund::getCode, Refund::getOrderId, Refund::getAgreeAmount));
        // 1. 查询订单数据
        Order order = orderFeign.one(id(refund.getOrderId()), attr(Order::getMerchantId, Order::getCode, Order::getPayment, Order::getPayEnum));

        // 2. 查询支付配置
        IcbcmoConfig icbcmoConfig = icbcmoConfig(order.getMerchantId(), order.getPayEnum());

        // 3. 支付退款金额
        String notifyUrl = icbcmoProperties.getRefundNotifySite() + "/api/icbcmo/refund/notify";
        String originAmount = String.valueOf(order.getPayment().multiply(BigDecimal.valueOf(100)).intValue());
        String refundAmount = String.valueOf(refund.getAgreeAmount().multiply(BigDecimal.valueOf(100)).intValue());

        // 4. 组装退款参数
        Map<String, String> params = new HashMap<>();
        params.put("merchantId", icbcmoConfig.getMerchantId());         // （必填）商戶號
        params.put("merchantOrderId", order.getCode());                 // （选填）商戶訂單號 註: merchantOrderId, icbcOrderId: 2選1
        params.put("icbcOrderId", "");                                  // （选填）ICBC訂單號 註: merchantOrderId, icbcOrderId: 2選1
        params.put("merchantRefundOrderId", refund.getCode());          // （必填）退款號（發起退款的商戶生成，需唯一，長度不超過30位）
        params.put("currency", "MOP");                                  // （必填）貨幣（固定MOP）
        params.put("originAmount", originAmount);                       // （必填）原始金額（單位：分）
        params.put("refundAmount", refundAmount);                       // （必填）退款金額（單位：分）
        params.put("notifyUrl", notifyUrl);                             // （必填）通知支付接口（get）
        params.put("refundExt", "");                                    // （选填）退款擴展字段
        params.put("sign", sign(icbcmoConfig.getMd5Key(), params));

        // 5. 转换成为请求参数 a=1&b=2&c=3&sign=4
        String queryStr = queryStr(params);

        // 6. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
        String request = icbcmoProperties.getUrl() + "/api/v1/refund/initRefund?" + queryStr;

        // 7. 请求接口
        log.info("IcbcmoService.query Request: \n" + request);
        String responseStr = Request.Post(request)
                                    .execute()
                                    .returnContent().toString();
        log.info("IcbcmoService.query Response: \n" + responseStr);

        // 8. 结果处理
        JSONObject response = JSONObject.parseObject(responseStr);
        // 8.1 退款成功
        if ("200".equals(response.getString("retCode")) &&
            "Success".equals(response.getJSONObject("returnObj").getString("refundStatus"))) {
            try {
                refundFeign.success(refundId);
            } catch (Exception e) {
                log.error("退款成功，退款订单处理异常：" + e.getMessage(), e);
            }
            return;
        }
        // 8.2 退款中，需要异步回调里面处理

        // 8.2 商户没有退款权限
        if ("200".equals(response.getString("retCode")) &&
            "Fail".equals(response.getJSONObject("returnObj").getString("refundStatus"))) {
            throw new ServiceException(response.getJSONObject("returnObj").getString("refundStatusDesc"));
        }
        throw new ServiceException(BaseResultEnum.ERROR, "Unknow Error");
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 退款结果异步通知
参考：https://app.icbcmo.site/gitbook/cn/chap8/chap8.html

#### 异步通知参数

**1. 原始参数**

```
http://dev.easybyte-hk.com/api/icbcmo/refund/notify&channelRefundId=90000000000200000011704109187879&channelTransId=4200001201202106174554738390&icbcOrderId=011WE20210617175718736113949&icbcRefundOrderId=RF20210617175801207146359&merchantId=011900000000002&merchantOrderId=20210617175715888215&merchantRefundOrderId=20210617175801196939&refundAmount=5&refundCurrency=MOP&refundRealAmount=4&refundRealCurrency=CNY&refundStatus=Success&refundStatusDesc=%E6%88%90%E5%8A%9F&sign=6d65b03c2ddd17622ff2c0a7c083d51b
```

**2. 解析参数**

```
channelRefundId=90000000000200000011704109187879, 
refundRealAmount=4, 
sign=6d65b03c2ddd17622ff2c0a7c083d51b, 
refundStatus=Success, 
merchantOrderId=20210617175715888215, 
icbcOrderId=011WE20210617175718736113949, 
icbcRefundOrderId=RF20210617175801207146359, 
refundRealCurrency=CNY, 
merchantId=011900000000002, 
refundStatusDesc=成功, 
merchantRefundOrderId=20210617175801196939, 
refundCurrency=MOP, 
channelTransId=4200001201202106174554738390, 
refundAmount=5
```

#### 异步接收方法

```java
@GetMapping("/refund/notify")
public void refundNotify(HttpServletRequest request, HttpServletResponse response) throws IOException {
    // 1. 解析回调参数
    log.info("ICBCMO 原始退款结果异步通知参数：{}{}{}", request.getRequestURL(), "&", request.getQueryString());
    Enumeration<String> names = request.getParameterNames();
    Map<String, String> params = new HashMap<>();
    while (names.hasMoreElements()) {
        String name = names.nextElement();
        String value = request.getParameter(name);
        params.put(name, value);
    }

    // 2. 处理支付回调 todo 应该异步处理
    try {
        log.info("ICBCMO 解析退款结果异步通知参数：{}", params);
        icbcmoService.syncOrderNotiry(params);
    } catch (Exception e) {
        log.error("ICBCMO 处理退款结果异步通知异常：{}", e.getMessage(), e);
    }

    // 3. 返回success以確認完成通知成功（不管有没有处理成功）
    @Cleanup
    PrintWriter printWriter = response.getWriter();
    printWriter.write("success");
    printWriter.flush();
    printWriter.close();
}
```

#### 异步处理方法


## 发起退款查询
参考：https://app.icbcmo.site/gitbook/cn/chap9/chap9.html

#### 上送参数

```
http://202.175.59.29:10443/gwinternet/app-svc/api/v1/refund/getRefundResult?merchantId=011901200001005&merchantRefundOrderId=20210610122701750629&sign=ff6038170e7521b7cef9dbe5e59083f6
```

#### 返回参数
**1. 异常参数**

```json
{
    "code": "1107", 
    "retCode": "500", 
    "msg": "查無此退款記錄, The refund record is not found."
}
```

**2. 线上 POS 支付退款正常参数**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "merchantOrderId": "20210610122513062065", 
        "icbcOrderId": "001IC20210610122524825533543", 
        "channel": "ICBCOnlinePosOrder", 
        "orderStatus": "Refund", 
        "orderStatusDesc": "", 
        "cashCurrency": "81", 
        "cashAmount": "1205", 
        "totalRefundCount": "1", 
        "refundList": [
            {
                "merchantRefundOrderId": "20210610122701750629", 
                "icbcRefundOrderId": "RF20210610122701149283870", 
                "refundStatus": "Success", 
                "refundCurrency": "MOP", 
                "refundAmount": "1205", 
                "refundRealCurrency": "81", 
                "refundRealAmount": "1205", 
                "refundSuccessTime": "2021-06-10 12:27:04"
            }
        ]
    }, 
    "sign": "1cc9acdf69982c51524c2a1ec877facc"
}
```

#### 上送方法
```java
@Override
public RefundStatus getRefundResult(Long refundId) {
    try {
        Refund refund = refundFeign.one(id(refundId), attr(Refund::getOrderId, Refund::getCode));
        Order order = orderFeign.one(id(refund.getOrderId()), attr(Order::getMerchantId, Order::getPayEnum, Order::getPayAgent));
        IcbcmoConfig icbcmoConfig = icbcmoConfig(order.getMerchantId(), order.getPayEnum());

        // 4. 组装请求参数
        Map<String, String> params = new HashMap<>();
        params.put("merchantId", icbcmoConfig.getMerchantId());         // （必填）商戶號
        params.put("icbcOrderId", "");                                  // （选填）ICBC訂單ID註: icbcOrderId, merchantOrderId: 2選1。處理次序為icbcOrderIdàmerchantOrderId
        params.put("merchantOrderId", order.getCode());                 // （选填）商戶訂單ID 註: icbcOrderId, merchantOrderId: 2選1。處理次序為icbcOrderIdàmerchantOrderId
        params.put("channelTransId", "");                               // （选填）渠道方原交易訂單ID註
        params.put("merchantRefundOrderId", refund.getCode());          // （选填）商戶退款訂單
        params.put("icbcRefundOrderId", "");                            // （选填）商戶退款訂單
        params.put("channelRefundId", "");                              // （选填）渠道方生成的退款交易流水號，僅退款成功返回
        params.put("sign", sign(icbcmoConfig.getMd5Key(), params));     // （必填）簽名用於校驗參數是否被篡改

        // 5. 转换成为请求参数 a=1&b=2&c=3&sign=4
        String queryStr = queryStr(params);

        // 6. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
        String request = icbcmoProperties.getUrl() + "/api/v1/refund/getRefundResult?" + queryStr;

        // 7. 请求接口
        log.info("IcbcmoService.getRefundResult Request: \n" + request);
        String responseStr = Request.Get(request)
                                    .execute()
                                    .returnContent().toString();
        log.info("IcbcmoService.getRefundResult Response: \n" + responseStr);

        // 8. 结果处理
        JSONObject response = JSONObject.parseObject(responseStr);

        if ("200".equals(response.getString("retCode"))) {
            // 8.1 退款中（所以其它状态，都当作退款中）
            if ("!Refund".equals(response.getJSONObject("returnObj").getString("orderStatus"))) {
                return RefundStatus.PROCESSING;
            }
            // 8.2 退款成功
            if ("Refund".equals(response.getJSONObject("returnObj").getString("orderStatus"))) {
                return RefundStatus.SUCCESS;
            }
        }
        // 8.3 其它异常（退款不存在之类的）
        throw new ServiceException(BaseResultEnum.ERROR, response.getString("msg"));
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 关闭订单
参考：https://app.icbcmo.site/gitbook/cn/chap10/chap10.html

#### 上送参数
```
http://202.175.59.29:10443/gwinternet/app-svc/api/v1/order/close?merchantId=011901200001005&merchantOrderId=20210610170740444269&sign=ddd3fd6b9085c36da9dc46e8547be54a
```

#### 返回参数
**1. 错误参数**

```json
{
  "code": "1095",
  "retCode": "500",
  "msg": "查無此訂單，請核實, Check this order, please verify"
}
```

**2. 正确参数**

```json
// ？？？没有发现这个返回
{
  "retCode": "200",
  "returnObj": {
    "merchantId": "011901200001001",
    "merchantOrderId": "ICBC_AIO_20190710_233100",
    "icbcOrderId": "011AL20190710233158379876488",
    "orderStatus": "Success",
    "orderStatusDesc": "",
    "errorCode": "",
    "errorMessage": ""
  },
  "sign": "4e349f7fa5ec47f5321ab967edc6bc8c"
}

// 这个有返回
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "merchantOrderId": "20210610170740444269", 
        "icbcOrderId": "001IC20210610174515370150881", 
        "status": "Success"
    }, 
    "sign": "ea8b253df58e8accbab5dfdbe575d966"
}
```

#### 上送方法
```java
private void close(IcbcmoConfig icbcmoConfig, String merchantOrderId) {
    try {
        // 4. 组装退款参数
        Map<String, String> params = new HashMap<>();
        params.put("merchantId", icbcmoConfig.getMerchantId());         // （必填）商戶號
        params.put("merchantOrderId", merchantOrderId);                 // （选填）商戶訂單號 註: merchantOrderId, icbcOrderId: 2選1
        params.put("icbcOrderId", "");                                  // （选填）ICBC訂單號 註: merchantOrderId, icbcOrderId: 2選1
        params.put("sign", sign(icbcmoConfig.getMd5Key(), params));

        // 5. 转换成为请求参数 a=1&b=2&c=3&sign=4
        String queryStr = queryStr(params);

        // 6. 组装接口地址 https://app.icbcmo.site/api/v1/order?a=1&b=2&c=3&sign=4
        String request = icbcmoProperties.getUrl() + "/api/v1/order/close?" + queryStr;

        // 7. 请求接口
        log.info("IcbcmoService.close Request: \n" + request);
        String responseStr = Request.Post(request)
                                    .execute()
                                    .returnContent().toString();
        log.info("IcbcmoService.close Response: \n" + responseStr);

        // 8. 结果处理
        JSONObject response = JSONObject.parseObject(responseStr);

        if ("200".equals(response.getString("retCode"))) {
            // 8.1 关闭失败
            if (!"Success".equals(response.getJSONObject("returnObj").getString("orderStatus"))) {
                throw new ServiceException(BaseResultEnum.ERROR, "关闭订单失败");
            }
            // 8.2 关闭成功
            if ("Success".equals(response.getJSONObject("returnObj").getString("orderStatus"))) {
                return;
            }
        }
        // 8.3 订单不存在，或者其它异常
        throw new ServiceException(BaseResultEnum.ERROR, "关闭订单失败");
    } catch (ServiceException e) {
        throw e;
    } catch (Exception e) {
        throw new ServiceException(BaseResultEnum.ERROR, e.getMessage(), e);
    }
}
```

## 下载对账文件
参考：https://app.icbcmo.site/gitbook/cn/chap11/chap11.html

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
#### 發起支付请求
**1. 请求参数**

```bash
zhangqinghua$ curl https://app.icbcmo.site/api/v1/order?amount=7&channel=WechatMiniApp&channelExt=%7B%22custom_display_code%22%3A%22%22%2C%22wx_appid%22%3A%22wx47d7859ff4ec85c4%22%2C%22openid%22%3A%22ohGhc5MWsYW_zkB4j0C_zCDx93H0%22%2C%22body%22%3A%22%22%7D&currency=MOP&merchantId=011900000000002&merchantOrderId=20210604155233849229&merchantTid=00000011&notifyUrl=https%3A%2F%2Fwww.baidu.com&returnUrl=pages%2Findex%2Findex&sign=722d3cfeae052f6d02bc1b0fa9af8648&timeout=300
```

**2. 异常响应**

```json
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

**3. 微信扫码支付响应**

```json
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
```

**4. 微信小程序支付响应**

```json
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

**5. POS 支付返回**

```json
{
    "retCode": "200", 
    "returnObj": {
        "merchantId": "011901200001005", 
        "merchantTid": "00000001", 
        "channel": "ICBCOnlinePosOrder", 
        "merchantOrderId": "20210607112944860991", 
        "icbcOrderId": "001IC20210607145051758855978", 
        "currency": "MOP", 
        "amount": "6", 
        "redirectUrl": "https://ebankpfovaopay4.dccnet.com.cn/servletxxxxx"
    }, 
    "sign": "7cc6461d3e6f92155f867790c4b40529"
}
```


#### 发起退款申请


#### 退款结果异步通知


## 商户异常
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
    "code": "1125", 
    "retCode": "500", 
    "msg": "獲取商戶信息出現異常. query merchant is abnormal"
}
// 商户号错误？？？？
{
    "code": "9999", 
    "retCode": "400", 
    "msg": "系統後台異常！, System background exception!"
}
```

## 附：簽名測試接口
参考：https://app.icbcmo.site/gitbook/cn/chap13/chap13.html

該測試接口用於測試下單參數的簽名結果，把所有下單參數（排除 sign 字段）post 到這個接口，即會返回參與簽名的內容（已隱藏signKey）及簽名結果，用於商戶自行校驗簽名方法及排查簽名錯誤原因。

請求URL: /api/v1/test/sign 請求方式: POST

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210608180002.png)

## 附：测试卡号和查询短信验证码

5440163901773005
5305063800002981
5428952600066083
5428952600066711
5440163900896286
5305066800004683
4574108900102574
4918872100062525
4377652000074711
4574108900117879
6250171800008073
6250191800004591
6250181800006308
6250181800016455
6250181800041479

访问 https://gwinternet.icbcmo.site/sms-inq-svc/v1/inqByCardNo?cardNo=4377651200336011 按卡号查询短信验证码 (把路径中 cardNo 后的值替换成测试的卡号)


