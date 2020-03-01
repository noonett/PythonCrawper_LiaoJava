---
title: python爬取偷廖老师的Java教程
date: 2020-03-01 23:40:56
tags: 
- 爬虫
- python
---
    今天想学点Java，访问了一下廖老师的网站，结果网很卡，可能廖老师在更新网站，然后突然想到可以写个爬虫一劳永逸。做这个爬虫的目的也是方便学习廖雪峰老师的Java教程，毕竟把网页存在本地方便浏览，并无他意。
<!--more-->
# Code
用的是requerst和xpath，推荐用Chrome上的xpath helper，真香。
```python
##导入库
import os
import time
import requests
import lxml
from lxml import etree
```
## 发送请求
```python
#send request
def send_request(url, headers):
    r = requests.get(url,headers=headers)
    print('状态码:',r.status_code)
    return r
```
## 提取数据
```python
#fetch data
def fetch(domain,resp, headers):
    selector = lxml.etree.HTML(resp.text)    #generate object
    titles = selector.xpath('//a[@class="x-wiki-index-item"]/text()')#拿到各个页面的小标题，比如《面向对象》这种      
    uri = selector.xpath('//a[@class="x-wiki-index-item"]/@href') #拿到各个页面的url的参数
    #打包到一起返回      
    urls = []
    for i in uri:
        urls.append(domain + i)    
    return titles, urls
```

## 下载页面
```python
def download(titles,urls):
    #设置下载路径
    cwd=os.getcwd()
    if not os.path.exists(cwd + '/廖雪峰Java教程HTML'):
        os.mkdir(cwd + '/廖雪峰Java教程HTML')
    filepath = cwd + '/廖雪峰Java教程HTML/'

    for key, value in zip(titles, urls):
        resp = requests.get(value,headers=headers)
        selector = lxml.etree.HTML(resp.text)    
        #拿到各个页面的正文
        html = selector.xpath('//div[@class="uk-flex-item-1"]') 
        #转换类型
        html = etree.tostring(html[0], pretty_print = True, method = "html")
        #设置文件名
        filename = filepath + "%d" % titles.index(key) + key +'.html'
        with open(filename,'wb')as f:
            f.write(html)
        #加一个延时，别访问太快
        time.sleep(0.3)
    print("下载完成")
```
## 主函数
```python
#main
headers = {
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chro'
                      'me/53.0.2785.104 Safari/537.36 Core/1.53.2372.400 QQBrowser/9.5.10548.400'
}
domain = 'http://www.liaoxuefeng.com'           #廖雪峰网站的域名
url =  "https://www.liaoxuefeng.com/wiki/1252599548343744" #Java教程的url

#获取Java教程的页面
resp = send_request(url, headers)
#提取所有的数据回来，包括标题和urls
titles, urls = fetch(domain, resp, headers)
#开始下载
download(titles,urls)
```

