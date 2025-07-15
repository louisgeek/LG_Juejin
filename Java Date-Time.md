# Java Date-Time
- java.time 是 Java 8 引入的全新日期时间 API，旨在替代旧的 java.util.Date、Calendar 和 SimpleDateFormat 等类，其设计遵循 ISO-8601 标准，解决了旧 API 设计混乱、线程不安全、时区处理复杂等诸多问题，提供了更直观易用、不可变（所有操作返回新对象，可以确保线程安全）、功能强大且符合现代需求的日期时间处理 API
- java.time 脱胎于 joda-time

## LocalDate
```java
//当前日期
LocalDate today = LocalDate.now();
//指定日期（月份从 1 开始，Calendar 从 0 开始）
LocalDate specificDate = LocalDate.of(2025, 6, 11);
//java.time.Month#MAY
LocalDate birthday = LocalDate.of(2000, Month.MAY, 15);
//解析字符串
LocalDate localDate = LocalDate.parse("2025-02-03");
```

## LocalTime
```java
//当前时间
LocalTime now = LocalTime.now();
//hour, minute
LocalTime specificTime = LocalTime.of(14, 30);
//hour, minute, second
LocalTime specificTime = LocalTime.of(14, 30, 45);
//解析字符串
LocalTime meetingTime = LocalTime.parse("15:30:10");  
```

## LocalDateTime
```java
//当前日期时间
LocalDateTime dateTime = LocalDateTime.now();
//LocalDate, LocalTime
LocalDateTime specificDateTime = LocalDateTime.of(specificDate, specificTime);
//java.time.Month#SEPTEMBER
LocalDateTime specificDateTime = LocalDateTime.of(2025, Month.SEPTEMBER, 11, 4, 23);
//year, month, dayOfMonth, hour, minute, second
LocalDateTime specificDateTime = LocalDateTime.of(2025, 6, 11, 14, 13, 17);
//解析字符串（用 T 隔开）
LocalDateTime parsedDateTime = LocalDateTime.parse("2025-05-15T14:30:45");
```

## ZonedDateTime
- 带时区的日期时间，包含时区偏移（比如 +08:00）
```java
//当前默认时区的时间
ZonedDateTime zonedDateTime = ZonedDateTime.now(); //Clock.systemDefaultZone()
//上海时区时间
ZonedDateTime shanghaiDateTime = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
//year, month, dayOfMonth, hour, minute, second, nanoOfSecond, zoneId
ZonedDateTime londonDateTime = ZonedDateTime.of(2025, 6, 11, 14, 0, 0, 0, ZoneId.of("Europe/London"));
ZonedDateTime tokyoDateTime  = ZonedDateTime.parse("2025-05-01T12:29:07.417938+09:00[Asia/Tokyo]");
```

## ZoneId
```java
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
ZonedDateTime shanghaiDateTime = ZonedDateTime.now(shanghaiZoneId);
//转换时区
ZoneId newYorkZoneId = ZoneId.of("America/New_York");
ZonedDateTime nycTime = shanghaiDateTime.withZoneSameInstant(newYorkZoneId); //纽约时间（自动计算时差）
```

## 日期时间运算
```java
LocalDate today = LocalDate.now();
LocalTime now = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
//
LocalDate tomorrow = today.plusDays(1); //当前日期 +1 天
LocalDate nextWeek = today.plusDays(7);

LocalDate nextMonth = today.plusMonths(1);
LocalDate lastMonth = today.minusMonths(1);

LocalTime minusHours = now.minusHours(3); //三小时前
LocalTime nextHour = now.plusHours(1);

LocalDateTime plusMinutes = dateTime.plusMinutes(3);
LocalDateTime plusSeconds = dateTime.minusSeconds(10);
```

## 日期时间比较
```java
LocalDate start = LocalDate.of(2025, 1, 1);
LocalDate end = LocalDate.of(2025, 12, 31);
//
boolean isBefore = start.isBefore(end);
boolean isAfter = start.isAfter(end);
boolean isEqual = start.isEqual(end);
```

## Instant 时刻
- 瞬时时间戳，精确到纳秒
```java
//当前时间戳（UTC 时间）
Instant now = Instant.now(); //2025-06-11T06:30:00Z
//
ZonedDateTime zonedDateTime = now.atZone(ZoneId.of("Asia/Shanghai"));
LocalDate localDate = zonedDateTime.toLocalDate();
LocalTime localTime = zonedDateTime.toLocalTime();
LocalDateTime localDateTime = zonedDateTime.toLocalDateTime();
```

## Period 周期
- 基于日期单位的持续时间（年/月/日）
```java
Period period = Period.between(startDate, endDate);

Period oneWeek = Period.ofDays(7);
Period oneWeek = Period.ofWeeks(1);

Period period = Period.ofYears(2).ofMonths(3).ofDays(5);
//获取
int years = period.getYears()
int months = period.getMonths()
int days = period.getDays()
```

## Duration 持续时间
- 基于时间单位的持续时间（秒/纳秒）
```java
Duration twoHours = Duration.ofHours(2);

//计算时间差
Duration duration = Duration.between(startTime, endTime); 
long hoursDiff = duration.toHours(); //获取相差小时数
long minutesDiff = duration.toMinutes();

//枚举 MINUTES("Minutes", Duration.ofSeconds(60))
long minutesBetween = ChronoUnit.MINUTES.between(startTime, endTime);
//枚举 DAYS("Days", Duration.ofSeconds(86400))
long daysBetween = ChronoUnit.DAYS.between(startDate, endDate);
```

## DateTimeFormatter
```java
LocalDate parsedDate = LocalDate.parse("2025-06-11", DateTimeFormatter.ISO_DATE);
LocalDate parsedDate = LocalDate.parse("2025-06-11+08:00", DateTimeFormatter.ISO_DATE);
LocalDate parsedTime = LocalDate.parse("14:30", DateTimeFormatter.ISO_TIME);
LocalDate parsedTime = LocalDate.parse("14:30:33", DateTimeFormatter.ISO_TIME);
LocalDate parsedTime = LocalDate.parse("14:30:33+08:00", DateTimeFormatter.ISO_TIME);
LocalDateTime parseDateTime = LocalDateTime.parse("2025-06-11T14:30", DateTimeFormatter.ISO_DATE_TIME);
LocalDateTime parseDateTime = LocalDateTime.parse("2025-06-11T14:30:33+08:00", DateTimeFormatter.ISO_DATE_TIME);
//
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime dateTime = LocalDateTime.parse("2025-05-05 12:30:33", dateTimeFormatter);
String formattedDateTime = dateTime.format(dateTimeFormatter);
//
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDate date = LocalDate.parse("2025-05-05", dateTimeFormatter);
String formattedDate = date.format(dateTimeFormatter);
```

## Clock
```java
Clock clock = Clock.systemUTC();
Instant now = Instant.now(clock);
```

## LocalDate 和 LocalDateTime 互转
```java
//LocalDateTime -> LocalDate
LocalDateTime localDateTime = LocalDateTime.now();
LocalDate localDate = localDateTime.toLocalDate();
//LocalDate -> LocalDateTime
LocalDate localDate = LocalDate.now();
LocalDateTime localDateTime = localDate.atStartOfDay(); //当天 00:00:00 零点
LocalDateTime localDateTime = localDate.atTime(8,20,33); //指定时间
LocalDateTime localDateTime = localDate.atTime(LocalTime.now()); //当前时间
```

## java.util.Date 和 Instant 互转
```java
//java.util.Date -> Instant
java.util.Date date = new java.util.Date();
Instant instant = date.toInstant();
Instant instant = Instant.ofEpochMilli(date.getTime());

Instant instant = Instant.now();
//Instant -> java.util.Date
java.util.Date date = java.util.Date.from(instant);
```

## TemporalAdjusters 时间调整器
```java
LocalDate today = LocalDate.now();
//本月第一天
LocalDate firstDayOfMonth = today.with(TemporalAdjusters.firstDayOfMonth());
//本月最后一天
LocalDate lastDayOfMonth = today.with(TemporalAdjusters.lastDayOfMonth());
//下周一
LocalDate nextMonday = today.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
//当年第一天
LocalDate firstDayOfYear = today.with(TemporalAdjusters.firstDayOfYear());
```
