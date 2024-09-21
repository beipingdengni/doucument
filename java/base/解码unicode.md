http://www.chi2ko.com/tool/CJK.htm



## 编码

```java
String unicode = "\\u" + Integer.toHexString('中' & 0xffff);
System.out.println(unicode); // 输出 \u4e2d
// 或者
String chinese = "中";
int codePoint = chinese.codePointAt(0);
System.out.println(Integer.toHexString(codePoint)); // 转为16进制 --》 输出 4e2d
```

## 解码

```java
int codePoint Integer.parseInt(hexStr, 16); // hexStr=4e2d
System.out.println((char) codePoint); // 输出：中
//或
String unicodeDe = new String(Character.toChars(codePoint));
System.out.println(unicodeDe); // 输出 中
// 或
public static String gbEncoding(final String gbString) {
  char[] utfBytes = gbString.toCharArray();
  StringBuilder unicodeBytes = new StringBuilder();
  for (int byteIndex = 0; byteIndex < utfBytes.length; byteIndex++) {
    String hexB = Integer.toHexString(utfBytes[byteIndex]); // 转16进制
    if (hexB.length() <= 2) {
      hexB = "00" + hexB;
    }
    unicodeBytes.append("\\u").append(hexB);
  }
  System.out.println("unicodeBytes is: " + unicodeBytes);
  return unicodeBytes.toString();
}
```

