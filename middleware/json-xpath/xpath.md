## python

### pip install htmlparsing

```python
from htmlparsing import Element

url = 'https://python.org'
r = requests.get(url)
e = Element(text=r.text, base_url=url)

#对象和属性
e.xpath('//a')[0].attrs
e.xpath('//a')[0].attrs.title

e.xpath('//a')[5].text
e.xpath('//a')[5].html
e.xpath('//a')[5].markdown

```

### pip3 install lxml

```python
from lxml import etree
 
wb_data = """
        <div>
            <ul>
                 <li class="item-0"><a href="link1.html">first item</a></li>
                 <li class="item-1"><a href="link2.html">second item</a></li>
                 <li class="item-inactive"><a href="link3.html">third item</a></li>
                 <li class="item-1"><a href="link4.html">fourth item</a></li>
                 <li class="item-0"><a href="link5.html">fifth item</a>
             </ul>
         </div>
        """
html = etree.HTML(wb_data)
result = etree.tostring(html)  # html_data = etree.tostring(html,pretty_print=True)
print(result.decode("utf-8"))

# 方式一
html = etree.HTML(wb_data)
html_data = html.xpath('/html/body/div/ul/li/a')

# 方式二
html = etree.HTML(wb_data)
html_data = html.xpath('/html/body/div/ul/li/a/text()')

```

### pip install bs4

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html,'html.parser',from_encoding='utf-8')
#查找所有的h4标签 
links = soup.find_all("h4")


```





## Java 解析xpath

### Java标准库中

#### xpath解析对象：javax.xml.xpath

```java
import javax.xml.xpath.*;

XPathFactory xpathFactory = XPathFactory.newInstance();
XPath xpath = xpathFactory.newXPath();
```

#### 文本对象：javax.xml.parsers

```java
import org.w3c.dom.*;

DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse("example.xml");
```

### 使用解析

```java
XPathExpression expr = xpath.compile("//book[price<10]");
NodeList nodes = (NodeList) expr.evaluate(doc, XPathConstants.NODESET);


XPathExpression expr = xpath.compile("//book[@id='1']/@title");
String title = (String) expr.evaluate(doc, XPathConstants.STRING);

```

