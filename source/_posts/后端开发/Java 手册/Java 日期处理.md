---
title: Java 日期处理

categories:
- 后端开发
- Java 手册

date: 2021-05-07
---
#### 计算两个时间差
```java
Duration.between(now, end);

// 计算今天到凌晨的剩余时间
Duration duration = Duration.between(LocalDateTime.now(), LocalDate.now().plusDays(1).atTime(0,0,0));
```


#### 当天、本周、本月 开始及结束时间
**1. Calendar 实现**

```java
public static void main(String[] args) {
   System.out.println("当前时间：" + new Date().toLocaleString());
   System.out.println("当天0点时间：" + getTimesmorning().toLocaleString());
   System.out.println("当天24点时间：" + getTimesnight().toLocaleString());
   System.out.println("本周周一0点时间：" + getTimesWeekmorning().toLocaleString());
   System.out.println("本周周日24点时间：" + getTimesWeeknight().toLocaleString());
   System.out.println("本月初0点时间：" + getTimesMonthmorning().toLocaleString());
   System.out.println("本月未24点时间：" + getTimesMonthnight().toLocaleString());
}

// 获得当天0点时间
public static Date getTimesmorning() {
   Calendar cal = Calendar.getInstance();
   cal.set(Calendar.HOUR_OF_DAY, 0);
   cal.set(Calendar.SECOND, 0);
   cal.set(Calendar.MINUTE, 0);
   cal.set(Calendar.MILLISECOND, 0);
   return cal.getTime();
}

// 获得当天24点时间
public static Date getTimesnight() {
   Calendar cal = Calendar.getInstance();
   cal.set(Calendar.HOUR_OF_DAY, 24);
   cal.set(Calendar.SECOND, 0);
   cal.set(Calendar.MINUTE, 0);
   cal.set(Calendar.MILLISECOND, 0);
   return cal.getTime();
}

// 获得本周一0点时间
public static Date getTimesWeekmorning() {
   Calendar cal = Calendar.getInstance();
   cal.set(cal.get(Calendar.YEAR), cal.get(Calendar.MONDAY), cal.get(Calendar.DAY_OF_MONTH), 0, 0, 0);
   cal.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
   return cal.getTime();
}

// 获得本周日24点时间
public static Date getTimesWeeknight() {
   Calendar cal = Calendar.getInstance();
   cal.setTime(getTimesWeekmorning());
   cal.add(Calendar.DAY_OF_WEEK, 7);
   return cal.getTime();
}

// 获得本月第一天0点时间
public static Date getTimesMonthmorning() {
   Calendar cal = Calendar.getInstance();
   cal.set(cal.get(Calendar.YEAR), cal.get(Calendar.MONDAY), cal.get(Calendar.DAY_OF_MONTH), 0, 0, 0);
   cal.set(Calendar.DAY_OF_MONTH, cal.getActualMinimum(Calendar.DAY_OF_MONTH));
   return cal.getTime();
}

// 获得本月最后一天24点时间
public static Date getTimesMonthnight() {
   Calendar cal = Calendar.getInstance();
   cal.set(cal.get(Calendar.YEAR), cal.get(Calendar.MONDAY), cal.get(Calendar.DAY_OF_MONTH), 0, 0, 0);
   cal.set(Calendar.DAY_OF_MONTH, cal.getActualMaximum(Calendar.DAY_OF_MONTH));
   cal.set(Calendar.HOUR_OF_DAY, 24);
   return cal.getTime();
}

===========================================================分割线===========================================================
当前时间：2021-3-1 10:31:40
当天0点时间：2021-3-1 0:00:00
当天24点时间：2021-3-2 0:00:00
本周周一0点时间：2021-3-1 0:00:00
本周周日24点时间：2021-3-8 0:00:00
本月初0点时间：2021-3-1 0:00:00
本月未24点时间：2021-4-1 0:00:00
```

**2. Java8 实现**

```java
// 当天零点
LocalDateTime.of(LocalDate.now(), LocalTime.MIN);

// 获取当天结束时间
LocalDateTime.of(LocalDate.now(), LocalTime.MAX);

/**
 * @Description TODO 获取本周的第一天或最后一天
 * @Param: [today, isFirst: true 表示开始时间，false表示结束时间]
 */
public static String getStartOrEndDayOfWeek(LocalDate today, Boolean isFirst){
    DayOfWeek week = today.getDayOfWeek();
    int value = week.getValue();
    resDate = isFirst ? today.minusDays(value - 1) : resDate = today.plusDays(7 - value);
    return resDate.toString();
}

/**
 * @Description TODO 获取本月的第一天或最后一天
 * @Param: [today, isFirst: true 表示开始时间，false表示结束时间]
 */
public static String getStartOrEndDayOfMonth(LocalDate today, Boolean isFirst){
    Month month = today.getMonth();
    int length = month.length(today.isLeapYear());
    resDate = isFirst ? LocalDate.of(today.getYear(), month, 1) : LocalDate.of(today.getYear(), month, length);
    return resDate.toString();
}

/**
 * @Description TODO 获取本季度的第一天或最后一天
 * @Param: [today, isFirst: true 表示开始时间，false表示结束时间]
 */
public static String getStartOrEndDayOfQuarter(LocalDate today, Boolean isFirst){
    Month month = today.getMonth();
    Month firstMonthOfQuarter = month.firstMonthOfQuarter();
    Month endMonthOfQuarter = Month.of(firstMonthOfQuarter.getValue() + 2);
    resDate = isFirst ? LocalDate.of(today.getYear(), firstMonthOfQuarter, 1) : 
                        LocalDate.of(today.getYear(), endMonthOfQuarter, endMonthOfQuarter.length(today.isLeapYear()));
    return resDate.toString();
}

/**
 * @Description TODO 获取本年的第一天或最后一天
 * @Param: [today, isFirst: true 表示开始时间，false表示结束时间]
 */
public static String getStartOrEndDayOfYear(LocalDate today, Boolean isFirst){
    resDate = isFirst ? LocalDate.of(today.getYear(), Month.JANUARY, 1) :
                        LocalDate.of(today.getYear(), Month.DECEMBER, Month.DECEMBER.length(today.isLeapYear()));
    return resDate.toString();
}
```