---
title: 聚合支付 SwiftPass

categories:
- 架构设计
- 支付开发

date: 2021-04-27 00:00:56
---
公司官网：https://www.swiftpass.cn
开发文档：https://open.swiftpass.cn

## 微信公众号/小程序接口
参考：https://open.swiftpass.cn/openapi/doc?index_1=67&index_2=1&chapter_1=1202&chapter_2=1203

#### 场景介绍
商户已有H5商城网站，用户通过消息或扫描二维码在微信内打开网页时，可以调用微信支付完成下单购买的流程。

步骤：
1. 商户下发图文消息或者通过自定义菜单吸引用户点击进入商户网页。
1. 进入商户网页，用户选择购买，完成选购流程。
1. 调起微信支付控件，用户开始输入支付密码。
1. 密码验证通过，支付成功。商户后台得到支付成功的通知。
1. 返回商户页面，显示购买成功。该页面由商户自定义。
1. 公众号下发消息，提示发货成功。该步骤可选。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210429145011.png)

#### 后端：初始化请求
```java

```

> 注意一：ContentType 必须为 APPLICATION_JSON，否则会报签名错误。
> 注意二：mch_id 错误也提示签名错误，很是误导人。

#### 前端：小程序调起支付

#### 前端：公众号调起支付

#### 前端：支付通知

#### 后端：支付通知
通知 URL 是初始化请求 API 中提交的参数 `notify_url`。支付完成后，平台会把相关支付和用户信息发送到该 URL，商户需要接收处理并按文档规范返回应答。

#### 后端：订单查询
由于网络异常或者系统的波动，可能会出现用户支付成功，但是商户侧未能成功接收到支付结果通知的情况，此时建议商户主动调用订单查询API确认订单状态。

**方案一**
以订单下单成功时间为基准，每隔 5 秒/30 秒/1 分钟/3 分钟/5 分钟/10 分钟/30 分钟调用订单查询 API 查询一次，最后一次查询还是未返回支付成功状态，则停止后续查询。（轮询时间间隔和次数，商户可以根据自身业务场景自定义设置）。

**方案二**
定时任务每隔 30 秒启动一次，找出最近 10 分钟内创建并且未支付的订单，调用订单查询 API 核实订单状态。系统记录订单查询的次数，在 10 次查询之后状态还是未支付成功，则停止后续查询。（轮询时间间隔和次数，商户可以根据自身业务场景自定义设置）。

#### 后端：申请退款

#### 后端：查询退款

#### 后端：关闭订单
商户订单支付失败需要生成新单号重新发起支付，要对原订单号调用关单，避免重复支付；系统下单后，用户支付超时，系统退出不再受理，避免用户继续，请调用关单接口。

#### 后端：通用方法
```java
/**
* 计算签名
*/
private static String sign(TreeMap<String, String> params) {
   // 1. 除 sign 字段外，所有參數按照字段名的 ascii 碼從小到大排序後使用 QueryString 的格式(即 key1=value1&key2=value2...)拼接而成，空值不傳遞，不參與簽名組串。
   String queryString = new ArrayList<>(params.keySet())
            .stream()
            .filter(key -> !StringUtils.isEmpty(params.get(key)))
            .map(key -> key + "=" + params.get(key))
            .collect(Collectors.joining("&"));

   System.out.println(queryString);
   // 2. MD5 加签
   if (!StringUtils.isEmpty(params.get("sign_type")) && !params.get("sign_type").equals("MD5")) {
      throw new ServiceException("目前只支持MD5加密");
   }
   queryString += "&key=0dc0b16bc452a9a562c1430ff2a7e720";


   return md5(queryString);
}

/**
* MD5加密，生成签名。
*/
private static String md5(String queryString) {
   try {
      MessageDigest m = MessageDigest.getInstance("MD5");
      m.update(queryString.getBytes(StandardCharsets.UTF_8));
      byte[] s = m.digest();
      StringBuilder result = new StringBuilder();
      for (byte b : s) {
            result.append(Integer.toHexString((0x000000ff & b) | 0xffffff00).substring(6));
      }
      return result.toString().toUpperCase();
   } catch (Exception e) {
      e.printStackTrace();
      throw new ServiceException("MD5加密失败：" + e.getMessage());
   }
}
```

## 补单机制
参考：https://open.swiftpass.cn/openapi/doc?index_1=67&index_2=1&chapter_1=1200&chapter_2=1201

对后台通知交互模式，如果平台收到商户的应答不是纯字符串 success 或超过 5 秒后返回时，平台认为通知失败，平台会通过一定的策略（通知频率为 0/15/15/30/180/1800/1800/1800/1800/3600，单位：秒）间接性重新发起通知，尽可能提高通知的成功率，但平台不保证通知最终能成功。

由于存在重新发送后台通知的情况，因此同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。平台推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回纯字符串 success。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重复插入数据造成的数据混乱。