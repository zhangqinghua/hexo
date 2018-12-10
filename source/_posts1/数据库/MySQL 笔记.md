---
title: MySQL 笔记

categories:
- 数据库

date: 2018-04-16
---

MySQL 笔记

## 常用查询

- 随机查询记录
    ```sql
    SELECT *  
    FROM `an_question` AS t1 
    JOIN (SELECT ROUND(RAND() * ((SELECT MAX(id) FROM `an_question`)-(SELECT MIN(id) FROM `an_question`))+(SELECT MIN(id) FROM `an_question`)) AS id) AS t2  
    WHERE t1.id >= t2.id  
    ORDER BY t1.id LIMIT 1;
    ```

- 查询玩家的游戏记录，胜利记录
    ```sql
    select game.category, count(game.id) totalcount, count(player2.id) wincount from an_game game
    left join an_game_player player on player.game_id = game.id 
    left join an_game_player player1 on player1.game_id = game.id and player1.min_user_id != 24
    left join an_game_player player2 on player2.game_id = game.id and player2.min_user_id != player1.min_user_id and player2.score > player1.score
    where player.min_user_id = 24
    ```

- 按照酒店产品数量和销量查询前3个城市
    ```sql
    SELECT
        sysCode.code_Name,
        sysCode.code_id 
    FROM
        rv_hotel_info hotel
        LEFT JOIN (
        SELECT
            room.*,
            count( orderd.id ) AS orderCount 
        FROM
            rv_room_info room
            LEFT JOIN rv_pay_order_detail orderd ON orderd.product_id = room.id 
            AND orderd.is_deleted = 'n' 
        GROUP BY
            room.id 
        ) room ON hotel.id = room.hotel_id 
        AND room.is_deleted = 'n' 
        AND room.STATUS = 'on'
        RIGHT JOIN sys_code sysCode ON sysCode.code_id = hotel.city_code 
    WHERE
        hotel.is_deleted = 'n' 
    GROUP BY
        sysCode.code_id 
    ORDER BY
        COUNT( room.id ) DESC,
        COUNT( room.orderCount ) DESC 
        LIMIT 3
    ```

- 酒店按照销售数量大小排序
    ```sql
    SELECT
        hotel.id,
        hotel.NAME 
    FROM
        rv_hotel_info hotel
        LEFT JOIN (
        SELECT
            room.hotel_id,
            room.id,
            orderd.nums 
        FROM
            rv_room_info room,
            ( SELECT product_id, sum( nums ) nums FROM rv_pay_order_detail GROUP BY product_id ) orderd 
        WHERE
            orderd.product_id = room.id 
        ) room ON room.hotel_id = hotel.id 
    GROUP BY
        hotel.id 
    ORDER BY
        sum( room.nums ) DESC
    ``` 

- mysql 将指定列的浮点数转化为整数
    ```sql
    update A set B =  cast(B as decimal(10,0)) 
    -- 或者
    update A set B = round(B,0)
    ```

## 常见问题

- 使用Navicat打开远程连接mysql很慢
    `选中数据库右键 > 连接属性 > 高级 > 保持连接间隔` 设置为100秒。

