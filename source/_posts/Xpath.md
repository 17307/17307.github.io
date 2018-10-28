---
title: Xpath
date: 2018-10-28 10:41:59
categories: python
tags: 爬虫
---


# 0x00 例子
```python
import requests
from lxml import html

page = requests.get('http://econpy.pythonanywhere.com/ex/001.html')
tree = html.fromstring(page.text)

buyers = tree.xpath('/html/body/div/div/text()')
# 这将创建prices的列表：
prices = tree.xpath('//span[@class="item-price"]/text()')
# /html/body/div[5]/div /html/body/div[5] /html/body/script/text()
print(buyers)
print(prices)
```

# 0x01 Xpath 语法

## XML 实例文档

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<bookstore>

<book>
  <title lang="eng">Harry Potter</title>
  <price>29.99</price>
</book>

<book>
  <title lang="eng">Learning XML</title>
  <price>39.95</price>
</book>

</bookstore>
```
![image](E7799025DC564312A87677C77B3CD29E)
![image](7CF6FAE8B47042779EA8C81EC24AF9F0)
![image](9F93C074B04A413EBC6F5FD6A37E36B4)
![image](6ADB6FFE727C4AD8BF9F2B42C8DF15DF)
