# Xpath语法基础

参考博客地址：https://blog.csdn.net/Once_day/article/details/129869027

## 第一种（dom基本处理）

```java
import org.w3c.dom.Document;
import javax.xml.parsers.DocumentBuilderFactory;
import org.xml.sax.InputSource;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathFactory;

// 创建XPath对象
XPath xpath = XPathFactory.newInstance().newXPath();


// 将HTML内容转换为Document对象
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
// 输入html 文本
InputSource inputSource = new InputSource(new StringReader("htmlContent"));
Document document = builder.parse(inputSource);
// 执行XPath查询
NodeList nodes = (NodeList) xpath.evaluate(xpathExpression, document, XPathConstants.NODESET);
```

### 第二种（JsoupXpath）

```xml
<dependency>
    <groupId>cn.wanghaomiao</groupId>
    <artifactId>JsoupXpath</artifactId>
    <version>2.5.3</version>
</dependency>
```

代码实现

```java
// 核心方法
// JXDocument doc = new JXDocument("html/xml 的文本字符串");
// List<JXNode> list = doc.selN("xpath");	
// //JXNode可以.sel()继续搜索下去，JXNode其toString方法为输出其内容
// List<Object> list = doc.sel("xpath");


JXDocument doc = new JXDocument("htmlContent");
List<JXNode> list = doc.selN("//div[@class='app']");
System.out.println("------------");
for (int i = 0; i < list.size(); i++) {
  JXNode app = list.get(i);
  System.out.println("style"+app.sel("@style").get(0));
  System.out.println("id:"+app.sel("p2[@class='id']/text()").get(0));
  System.out.println("name:"+app.sel("p2[@class='name']/text()").get(0));
  System.out.println("version:"+app.sel("p2[@class='version']/text()").get(0));
  System.out.println("------------");
}
```

