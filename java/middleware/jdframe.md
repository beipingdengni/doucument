

## jdframe

GitHub: https://github.com/burukeYou/JDFrame

博客参考[Java8的Stream流](https://www.toutiao.com/article/7396872815141552681/?app=news_article&timestamp=1722956883&use_new_style=1&req_id=202408062308032C24C32B9EBDF9525C7B&group_id=7396872815141552681&share_token=9E00369F-C35A-4FDD-A54F-6502DD6017F8&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_ios&utm_campaign=client_share&wxshare_count=1&source=m_redirect)

```xml
<dependency>
    <groupId>io.github.burukeyou</groupId>
    <artifactId>jdframe</artifactId>
    <version>0.0.4</version>
</dependency>
```

由于经常记不住stream的一些API每次要复制来复制去并且又长又臭，想要更加语意化的api，于是想到了以前写大数据Spark pandnas 等DataFrame模型时的API， 然后发现其实也存在java的JVM层的DataFrame模型比如 tablesaw，joinery

但是他们得硬编码去指定字段名，这对于有代码洁癖的人实在难以忍受，而且我只是简单统计下数据，我想在一些场景下能不能使用匿名函数去指定的字段处理去处理，于是便有了这个

> 一个jvm层级的仿DataFrame工具，语意化和简化java8的stream流式处理工具

```java
// 获取学生年龄在9到16岁的学学校合计分数最高的前10名的学校
SDFrame<FI2<String, BigDecimal>> sdf2 = SDFrame.read(studentList)
  .whereNotNull(Student::getAge)
  .whereBetween(Student::getAge,9,16)
  .groupBySum(Student::getSchool, Student::getScore)
  .sortDesc(FI2::getC2)
  .cutFirst(10);
```

