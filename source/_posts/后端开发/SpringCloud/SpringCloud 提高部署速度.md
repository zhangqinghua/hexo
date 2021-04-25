---
title: SpringCloud 提高部署速度

categories:
- 后端开发
- SpringCloud

date: 2021-03-16
---

1. 只编译必须的模块。
1. 体积瘦身。
1. 提高启动速度。

## 打包指定的模块
在多模块的 Maven 项目中，如果每次打包整个工程显得有些冗余和笨重。

```bash
mvn clean package

# [INFO] icebartech-parent .................................. SUCCESS [  1.406 s]
# [INFO] icebartech-beetlsql ................................ SUCCESS [ 15.974 s]
# [INFO] easybyte ........................................... SUCCESS [  0.060 s]
# [INFO] aframework ......................................... SUCCESS [  0.065 s]
# [INFO] icebartech-core .................................... SUCCESS [ 15.247 s]
# [INFO] icebartech-oss ..................................... SUCCESS [  3.345 s]
# [INFO] icebartech-sms ..................................... SUCCESS [  3.655 s]
# [INFO] easybyte-api ....................................... SUCCESS [  9.841 s]
# [INFO] easybyte-gateway ................................... SUCCESS [  4.043 s]
# [INFO] easybyte-swagger ................................... SUCCESS [  3.582 s]
# [INFO] easybyte-log ....................................... SUCCESS [  4.723 s]
# [INFO] easybyte-auth ...................................... SUCCESS [  5.448 s]
# [INFO] easybyte-product ................................... SUCCESS [  4.815 s]
# [INFO] easybyte-store ..................................... SUCCESS [  4.883 s]
# [INFO] easybyte-merchant .................................. SUCCESS [  5.926 s]
# [INFO] easybyte-base ...................................... SUCCESS [  4.157 s]
# [INFO] easybyte-flow ...................................... SUCCESS [  3.539 s]
# [INFO] easybyte-consumer .................................. SUCCESS [  3.471 s]
# [INFO] easybyte-media ..................................... SUCCESS [  3.906 s]
# [INFO] easybyte-stats ..................................... SUCCESS [  3.871 s]
# [INFO] easybyte-member .................................... SUCCESS [  4.311 s]
# [INFO] easybyte-market .................................... SUCCESS [  4.254 s]
# [INFO] easybyte-template .................................. SUCCESS [  3.564 s]
# [INFO] ------------------------------------------------------------------------
# [INFO] BUILD SUCCESS
# [INFO] ------------------------------------------------------------------------
# [INFO] Total time:  01:56 min
# [INFO] Finished at: 2021-03-16T12:22:29+08:00
# [INFO] ------------------------------------------------------------------------
```

我们可以使用命令 `mvn clean package install -pl 模块的名称 -am` 打包指定模块。
1. `-pl` 
   打包指定模块。
1. `-am`
   打包所指定模块的依赖模块。

```bash
mvn clean package install -pl easybyte-template -am

# [INFO] icebartech-parent .................................. SUCCESS [  0.732 s]
# [INFO] icebartech-beetlsql ................................ SUCCESS [  6.958 s]
# [INFO] easybyte ........................................... SUCCESS [  0.051 s]
# [INFO] aframework ......................................... SUCCESS [  0.056 s]
# [INFO] icebartech-core .................................... SUCCESS [ 10.841 s]
# [INFO] easybyte-api ....................................... SUCCESS [ 11.444 s]
# [INFO] easybyte-template .................................. SUCCESS [  5.048 s]
# [INFO] ------------------------------------------------------------------------
# [INFO] BUILD SUCCESS
# [INFO] ------------------------------------------------------------------------
# [INFO] Total time:  36.369 s
# [INFO] Finished at: 2021-03-16T12:09:57+08:00
# [INFO] ------------------------------------------------------------------------
```

