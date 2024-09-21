

# diff算法

在 Git 中，有四种diff算法，分别是Myers、Minimal、Patience和Histogram，它们用于获取位于两个不同提交中的两个相同文件的差异。

Myers算法由Eugene W.Myers在1986年发表的一篇论文中提出，是一个能在大部分情况产生”最短的直观的“diff的一个算法。

文本差异对比涉及到的算法介绍 ：
[How different are different diff algorithms in Git](https://link.springer.com/article/10.1007/s10664-019-09772-z)

Myers Diff 差分算法  [git生成diff原理：Myers差分算法](https://chenshinan.github.io/2019/05/02/git%E7%94%9F%E6%88%90diff%E5%8E%9F%E7%90%86%EF%BC%9AMyers%E5%B7%AE%E5%88%86%E7%AE%97%E6%B3%95/)



diff-match-patch

https://github.com/google/diff-match-patch

java-diff-utils

https://github.com/java-diff-utils/java-diff-utils

```xml
<dependency>
  <groupId>io.github.java-diff-utils</groupId>
  <artifactId>java-diff-utils</artifactId>
  <version>4.11</version>
</dependency>
```





博客网站：

https://blog.csdn.net/qq_33697094/article/details/121681707

