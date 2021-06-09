---
title: Redis 常用操作

categories:
- 后端开发
- Redis 手册

date: 2021-06-09
---
####  Redis 限制短信频率
```java
/**
* 发送验证码频率验证
* 1. 一小时内，同个手机号，发送验证码条数达到5条，提示内容：一小时内只能发送5条短信，请稍后再尝试
* 2. 一天内，同个手机号，发送验证码条数达到10条， 提示内容：一天内只能发送10条短信，请明天再尝试
* @param merchantId    商户Id
* @param mobile        手机号码 131354564565
* @param mobileCode    手机区号 86
*/
private void limitValid(Long merchantId, String mobile, String mobileCode) {
   String key = mobileCode + mobile;
   final String hourGroup = "sms_code_count_limit_hour";
   Long count = redisComponent.incr(hourGroup, key);
   if (count > 5) throw new ServiceException(I18n2.warn_sms_code_count_limit_hour.locale());

   final String dayGroup = "sms_code_count_limit_day";
   count = redisComponent.incr(dayGroup, key);
   if (count > 10) throw new ServiceException(I18n2.warn_sms_code_count_limit_day.locale());

   // 设置离下一小时过期时间
   LocalDateTime nextHour = LocalDateTime.now().plusHours(1).withMinute(0).withSecond(0);
   redisComponent.expire(hourGroup, key, Duration.between(LocalDateTime.now(), nextHour).toMinutes() * 60);

   // 设置离下一天过期时间
   LocalDateTime nextDay = LocalDateTime.of(LocalDate.now(), LocalTime.MAX);
   redisComponent.expire(dayGroup, key, Duration.between(LocalDateTime.now(), nextHour).toMinutes() * 60);
}
```