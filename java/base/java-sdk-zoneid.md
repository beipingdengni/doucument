## ZoneId

Java新版日期时间API中使用ZoneId代表时区。ZoneId通常作为Instant和LocalDateTime转换的桥梁。Instant表示时间线上的一点（时间戳），LocalDateTime也表示时间点。他们两者不同之处就是Instant的时间点是带上了时区的。我们可以理解为Instant = LocalDateTime + ZoneId。下面我们就来详细介绍ZoneId。

### ZoneOffset和ZoneRegion

ZoneId支持两种类型格式初始化，一种是时区偏移的格式（基于UTC/Greenwich时），一种是地域时区的格式（eg：Europe/Paris）。 ZoneId是抽象类，具体的逻辑实现由来子类完成，ZoneOffset处理时区偏移类型的格式，ZoneRegion处理基于地域时区的格式。但是我们在实际使用中还是使用ZoneId。他的子类我们一般不直接使用。

### ZoneId的使用

1.获取系统默认时区：

```
ZoneId zoneId = ZoneId.systemDefault();
```

2.获取可用的时区Id：

```
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
```

部分结果如下：

> Asia/Aden, America/Cuiaba, Etc/GMT+9, Etc/GMT+8, Africa/Nairobi, America/Marigot, Asia/Aqtau, Pacific/Kwajalein, America/El_Salvador, Asia/Pontianak, Africa/Cairo, Pacific/Pago_Pago...
>
> 国内一般用Asia/Shanghai

3.初始化方法：

- 使用ZoneRegion格式，获取中国地区时区，打印当地日期时间

```
ZoneId zoneId = ZoneId.of("Asia/Shanghai");  
Clock clock = Clock.system(zoneId);            
Instant instant = clock.instant();             
LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zoneId); 
System.out.println(localDateTime.toString());  
```

- 使用ZoneOffset格式，获取中国地区时区，打印当地日期时间

```
ZoneId zoneId = ZoneId.of("+8");
Clock clock1 = Clock.system(zoneId);
Instant instant = clock1.instant();
LocalDateTime localDateTime1 = LocalDateTime.ofInstant(instant, zoneId);
System.out.println(localDateTime1.toString());
```

### tips：ZoneId支持的格式

| 格式                                                         | 描述                                                         | 示例                        |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------- |
| Z, GMT, UTC, UT                                              | 格林尼治标准时间，和中国相差8个小时                          | ZoneId.of("Z");             |
| +h +hh +hh:mm -hh:mm +hhmm -hhmm +hh:mm:ss -hh:mm:ss +hhmmss -hhmmss | 表示从格林尼治标准时间偏移时间，中国用+8表示                 | ZoneId.of("+8");            |
| 前缀：UTC+, UTC-, GMT+, GMT-, UT+ UT-。后缀:+h +hh +hh:mm -hh:mm... | 表示从格林尼治标准时间偏移时间                               | ZoneId.of("UTC+8");         |
| Asia/Aden, America/Cuiaba, Etc/GMT+9, Etc/GMT+8, Africa/Nairobi, America/Marigot... | 地区表示法，这些ID必须包涵在getAvailableZoneIds集合中，否则会抛出异常 | ZoneId.of("Asia/Shanghai"); |