---
title: Mybatis

categories:
- Java

date: 2018-03-26 17:00:00
---

Mybatis比较

# 分页

直接`limit`

# 循环

```xml
<update id="pubS" parameterType="Map">
  UPDATE BMC_SUBPLATE
  SET PLSTATUS = '02'
  WHERE
  <foreach collection="ids" item="plid" open="" close="" separator="OR">
   PLID = #{plid}
  </foreach>
</update>
```

# 嵌套

```xml
<resultMap id="salesResultMap" type="com.emerson.learning.pojo.Sales">
    <id property="salesId" column="sales_id" />
    <result property="salesName" column="sales_name" />
    <result property="phone" column="sales_phone" />
    <result property="fax" column="sales_fax" />
    <result property="email" column="sales_email" />
    <result property="isValid" column="is_valid" />
    <result property="createdTime" column="created_time" />
    <result property="updateTime" column="update_time" />

    <!-- 定义多对一关联信息（嵌套结果方式） -->
    <association property="userInfo" resultMap="userResult" />
    <!-- 定义一对多集合信息（每个销售人员对应多个客户） -->
    <!-- <collection property="customers" column="sales_id" select="getCustomerForSales" /> -->

    <collection property="customers" ofType="com.emerson.learning.pojo.Customer">
        <id property="customerId" column="customer_id" />
        <result property="customerName" column="customer_name" />
        <result property="isValid" column="is_valid" />
        <result property="createdTime" column="created_time" />
        <result property="updateTime" column="update_time" />
        <!-- 映射客户与登录用户的关联关系，请注意columnPrefix属性 -->
        <association property="userInfo" resultMap="userResult" columnPrefix="cu_" />
    </collection>
</resultMap>
```

## 返回类型

- resultMap：结果集[对象等]
- resultType：`Integer, String, Long, class`

## 常见查询

- 查询某个日期的记录
    `String starttime = "2018-04-19"`
    ```sql
    and game.starttime >= CONCAT(#{starttime},' 00:00:00')
    and CONCAT(#{starttime},' 23:59:59') >= game.starttime
    ```

## 常见问题

- 返回Map数据提示Long无法转化为String
    当Mybatis返回的数据全部是数字类型时，用Map<String, String>接收会失败，需要将返回的数据中的一个设为字符串
    ```sql
    select CONCAT(answer.question_category, 'a'), count(*), sum( answer.score > 0) from an_game_player_answer answer
    where answer.player_id = 24
    group by answer.question_category
    ```