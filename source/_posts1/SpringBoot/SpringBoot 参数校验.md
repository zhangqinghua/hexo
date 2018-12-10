---
title: SpringBoot 参数校验.md
categories:
- SpringBoot

date: 2018-06-13
---

常用参数校验正则

## 常用参数校验正则

```java
@ApiModelProperty(value = "手机号",example = "13169920185")
@NotBlank(message = "手机号码必填")
@Pattern(regexp = "^1(3|4|5|7|8)\\d{9}$",message = "手机号码格式错误")
private String mobile;

@ApiModelProperty(value = "密码", example = "123456")
@NotBlank(message = "密码必填")
@Pattern(regexp = "[a-zA-Z0-9]{6,12}", message = "密码只能6-12位字母与数据组合")
private String password;

@ApiModelProperty("身份证号")
@NotBlank(message = "身份证号不能为空")
@Pattern(regexp = "^\\d{17}[\\d|x|X]|\\d{15}$", message = "身份证号码格式错误")
private String cardId;

@Min(value = 0, message = "库存数量不能小于0")
@Max(value = 999999, message = "库存数量不能大于999999")
```

