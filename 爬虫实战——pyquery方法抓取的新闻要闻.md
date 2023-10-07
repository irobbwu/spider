# 爬虫实战——pyquery方法抓取的新闻要闻
使用lxml模块进行快速的 XML 和 HTML 操作。

安装方法：
```shell
pip install pyquery
```
导入方式：
```python
from pyquery import PyQuery
```

使用方法 .text()、.html()
```python
html = (
'<a href="./202309/t20230920_1360717.html" target="_blank" title="2023&#x5E74;9&#x6708;20&#x65E5;24&#x65F6;&#x8D77;&#x56FD;&#x5185;&#x6210;&#x54C1;&#x6CB9;&#x4EF7;&#x683C;&#x6309;&#x673A;&#x5236;&#x8C03;&#x6574;">2023年9月20日24时起国内成品油价格按机制调整</a><span>2023/09/20</span>'
 '<a href="./202309/t20230907_1360442.html" target="_blank" title="&#x56FD;&#x5BB6;&#x53D1;&#x5C55;&#x6539;&#x9769;&#x59D4;&#x6709;&#x5173;&#x8D1F;&#x8D23;&#x540C;&#x5FD7;&#x5C31;&#x300A;&#x4E1C;&#x839E;&#x6DF1;&#x5316;&#x4E24;&#x5CB8;&#x521B;&#x65B0;&#x53D1;&#x5C55;&#x5408;&#x4F5C;&#x603B;&#x4F53;&#x65B9;&#x6848;&#x300B;&#x7B54;&#x8BB0;&#x8005;&#x95EE;">国家发展改革委有关负责同志就《东莞深化两岸创新发展合作总体方案》答记者问</a><span>2023/09/07</span>'
 '<a href="./202309/t20230906_1360399.html" target="_blank" title="&#x56FD;&#x5BB6;&#x53D1;&#x5C55;&#x6539;&#x9769;&#x59D4;&#x53EC;&#x5F00;&#x5B66;&#x4E60;&#x8D2F;&#x5F7B;&#x4E60;&#x8FD1;&#x5E73;&#x65B0;&#x65F6;&#x4EE3;&#x4E2D;&#x56FD;&#x7279;&#x8272;&#x793E;&#x4F1A;&#x4E3B;&#x4E49;&#x601D;&#x60F3;&#x4E3B;&#x9898;&#x6559;&#x80B2;&#x603B;&#x7ED3;&#x4F1A;&#x8BAE;">国家发展改革委召开学习贯彻习近平新时代中国特色社会主义思想主题教育总结会议</a><span>2023/09/06</span>'
 '<a href="./202309/t20230906_1360397.html" target="_blank" title="2023&#x5E74;9&#x6708;6&#x65E5;&#x56FD;&#x5185;&#x6210;&#x54C1;&#x6CB9;&#x4EF7;&#x683C;&#x4E0D;&#x4F5C;&#x8C03;&#x6574;">2023年9月6日国内成品油价格不作调整</a><span>2023/09/06</span>'
)
pq = PyQuery(html)
print(pq.html())
```
输出：![Screenshot from 2023-09-28 16-49-30](/home/ubuntu/Pictures/Screenshot from 2023-09-28 16-49-30.png)
```python
print(pq.text())
```
输出：
![Screenshot from 2023-09-28 16-50-27](/home/ubuntu/Pictures/Screenshot from 2023-09-28 16-50-27.png)
看一下类型：

```python
pq[0]

# <Element span at 0x7f65aa0dd5e0>
```
是一个
lxml.etree._Element 类型，可以用xpath语法解析

 **检索方式**
对pyquery对象（'标签'），即可获得所有标签，支持css选择器
.items 可以将每个元素转化成pyquery迭代器
- id选择器 #
- 标签选择器 标签
- 类选择器  .
- 选择器分类 ，
- 后代选择器 空格
```python
for i in pq('a').items():
    print(type(i))
'''
<class 'pyquery.pyquery.PyQuery'>
<class 'pyquery.pyquery.PyQuery'>
<class 'pyquery.pyquery.PyQuery'>
<class 'pyquery.pyquery.PyQuery'>
'''
```

元素属性可以使用attr()方法检索。
```python
for i in pq('a').items():
    print(i.attr('href'))
'''
./202309/t20230920_1360717.html
./202309/t20230907_1360442.html
./202309/t20230906_1360399.html
./202309/t20230906_1360397.html
'''
```

.remove 删除标签
```python
url = 'https://www.ndrc.gov.cn/xwdt/xwfb/'
reponse = requests.get(url, headers=head)
reponse.encoding= 'utf-8'
pq = PyQuery(reponse.text)
pq('ul.u-list')('li')

# 输出
# [<li>, <li>, <li>, <li>, <li>, <li.empty>, <li>, <li>, <li>, <li>, <li>, <li.empty>, <li>, <li>, <li>, <li>, <li>, <li.empty>, <li>, <li>, <li>, <li>, <li>, <li.empty>, <li>, <li>, <li>, <li>, <li>, <li.empty>]
```
删除 li.empty 标签
```python
pq('ul.u-list')('li.empty').remove()
pq('ul.u-list')('li')

# 输出
# [<li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>, <li>]
```
# 要闻抓取
观察[](https://www.ndrc.gov.cn/xwdt/xwfb/)新闻信息都在li标签中![Screenshot from 2023-09-28 15-06-40](/home/ubuntu/Pictures/Screenshot from 2023-09-28 15-06-40.png)
```html
<ul class="u-list"> 
	<li>
```
其中跳转链接在a中，标题在正文中，时间在span中，利用PyQuery对其进行解析
```python
url = 'https://www.ndrc.gov.cn/xwdt/xwfb/'
reponse = requests.get(url, headers=head)
reponse.encoding= 'utf-8'
pq = PyQuery(reponse.text)
# li.empty 为空行标签，删除
pq('ul.u-list')('li.empty').remove()
li_info_list = [[url + i('a').attr('href')[1:], i('a').text(), i('span').text()] for i in pq('ul.u-list')('li').items()]

print(li_info_list)
```
![Screenshot from 2023-09-28 17-36-39](/home/ubuntu/Pictures/Screenshot from 2023-09-28 17-36-39.png)
获取制定时间的li_info_list

```python
def get_l_list(url, end_time, page=0):
    # 控制翻页变化的url
    if page != 0:
        crawl_url = url + "_%s" % page + ".html"
    else:
        crawl_url = url + ".html"

    # 获取li_info_list
    reponse = requests.get(crawl_url, headers=head)
    reponse.encoding = "utf-8"
    pq = PyQuery(reponse.text)
    pq("ul.u-list")("li.empty").remove()
    li_info_list = [
        ['https://www.ndrc.gov.cn/xwdt/xwfb' + i("a").attr("href")[1:], i("a").text(), i("span").text()]
        for i in pq("ul.u-list")("li").items()
    ]

    # 获取当页最早文章发布时间
    publish_time = datetime.strptime(li_info_list[-1][-1], "%Y/%m/%d")

    # 如果时间时间小于截至时间，停止
    if publish_time < end_time:
        return li_info_list

    # 否则翻页，递归执行
    page += 1
    li_info_list += get_l_list(url, end_time, page)
    return li_info_list


begin_time = datetime.now()
end_time = datetime(2023, 3, 1)
li_list = get_l_list(url, end_time)
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a64ef2dd979047d5b093da68cf40c690.png#pic_center)
对li标签中每个url进行爬取即可
```python
def crawl_li_list(li_list, db_insert_data):
    for li in li_list:
        publish_time = datetime.strptime(li[-1], "%Y/%m/%d")
        if end_time <= publish_time < begin_time:
            a_url = li[0]
            reponse = requests.get(a_url, headers=head)
            reponse.encoding = "utf-8"
            pq = PyQuery(reponse.text)
            db_insert_data.append(
                [
                    # crawl_time
                    datetime.now(),
                    # publish_time
                    publish_time,
                    # 原始网址
                    a_url,
                    # 网站模块
                    pq('div.path')('a:last').text(),
                    # 标题
                    li[1],
                    # 作者或来源
                    pq('div.ly.laiyuantext')('span').text(),
                    # 文章内容
                    pq('div.article_con.article_con_title').text(),
                    # 附件 存储附件地址 media/app03/data/
                    [],
                ]
            )
db_insert_data = []
crawl_li_list(li_list, db_insert_data) 
```
看一下结果
![Screenshot from 2023-09-28 17-57-23](/home/ubuntu/Pictures/Screenshot from 2023-09-28 17-57-23.png)

# 完整代码
```pyhon
import json
import os
import re
from datetime import datetime, timedelta

import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup
from pyquery import PyQuery

head = {
    "User-Agent": np.random.choice(
        [
            "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.1 Safari/537.36",
            "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36",
            "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2227.0 Safari/537.36",
            "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2226.0 Safari/537.36",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; chromeframe/13.0.782.215)",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; chromeframe/11.0.696.57)",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0) chromeframe/10.0.648.205",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/4.0; GTB7.4; InfoPath.1; SV1; .NET CLR 2.8.52393; WOW64; en-US)",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/5.0; chromeframe/11.0.696.57)",
            "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.0; Trident/4.0; GTB7.4; InfoPath.3; SV1; .NET CLR 3.1.76908; WOW64; en-US)",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.75.14 (KHTML, like Gecko) Version/7.0.3 Safari/7046A194A",
            "Mozilla/5.0 (iPad; CPU OS 6_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/6.0 Mobile/10A5355d Safari/8536.25",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/537.13+ (KHTML, like Gecko) Version/5.1.7 Safari/534.57.2",
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/534.55.3 (KHTML, like Gecko) Version/5.1.3 Safari/534.53.10",
            "Mozilla/5.0 (iPad; CPU OS 5_1 like Mac OS X) AppleWebKit/534.46 (KHTML, like Gecko ) Version/5.1 Mobile/9B176 Safari/7534.48.3",
        ]
    )
}


def get_l_list(url, end_time, page=0):
    # 控制翻页变化的url
    if page != 0:
        crawl_url = url + "_%s" % page + ".html"
    else:
        crawl_url = url + ".html"

    # 获取li_info_list
    reponse = requests.get(crawl_url, headers=head)
    reponse.encoding = "utf-8"
    pq = PyQuery(reponse.text)
    pq("ul.u-list")("li.empty").remove()
    li_info_list = [
        [
            "https://www.ndrc.gov.cn/xwdt/xwfb" + i("a").attr("href")[1:],
            i("a").text(),
            i("span").text(),
        ]
        for i in pq("ul.u-list")("li").items()
    ]

    # 获取当页最早文章发布时间
    publish_time = datetime.strptime(li_info_list[-1][-1], "%Y/%m/%d")

    # 如果时间时间小于截至时间，停止
    if publish_time < end_time:
        return li_info_list

    # 否则翻页，递归执行
    page += 1
    li_info_list += get_l_list(url, end_time, page)
    return li_info_list


def crawl_li_list(li_list, db_insert_data):
    for li in li_list:
        publish_time = datetime.strptime(li[-1], "%Y/%m/%d")
        if end_time <= publish_time < begin_time:
            a_url = li[0]
            reponse = requests.get(a_url, headers=head)
            reponse.encoding = "utf-8"
            pq = PyQuery(reponse.text)
            db_insert_data.append(
                [
                    # crawl_time
                    datetime.now(),
                    # publish_time
                    publish_time,
                    # 原始网址
                    a_url,
                    # 网站模块
                    pq("div.path")("a:last").text(),
                    # 标题
                    li[1],
                    # 作者或来源
                    pq("div.ly.laiyuantext")("span").text(),
                    # 文章内容
                    pq("div.article_con.article_con_title").text(),
                    # 附件 存储附件地址 media/app03/data/
                    [],
                ]
            )


url_list = [
    "https://www.ndrc.gov.cn/xwdt/xwfb/index",
    "https://www.ndrc.gov.cn/xwdt/tzgg/index",
    "https://www.ndrc.gov.cn/xwdt/dt/wlddt/index",
]

begin_time = datetime.now()
end_time = datetime(2023, 9, 1)


def spider():
    db_insert_data = []
    for url in url_list:
        li_list = get_l_list(url, end_time)
        crawl_li_list(li_list, db_insert_data)
    return db_insert_data


spider()
```
![Screenshot from 2023-09-28 18-01-47](/home/ubuntu/Pictures/Screenshot from 2023-09-28 18-01-47.png)