---
title: Selenium爬虫_1
date: 2017-10-01 10:12:47
tags: 
- Selenium 
- 爬虫
categories: 
- 爬虫
---

# Selenium爬虫_1

上个月进了老师的实验室帮忙干活，要准备一个图像识别相关项目的图片素材。给了我一个台湾的“[国立故宫博物院Open Data资料开放平台](http://theme.npm.edu.tw/opendata/)”的网址，让我爬这个网站上面的图片。

这个网站嘞很是神奇，从藏品的详情页跳转到下一件藏品的详情页，网页的URL是不会变的（由于知识浅薄，并不知道它是如何做到这一点的）。然后嘞，如果不是从资料检索系统进入详情页，而是直接用浏览器访问某一个详情页的URL的话，跳转到下一件藏品的详情页的那个按钮就点不动啦。

因此啊，我就得现学一个能够帮助我爬下这个网站数据的框架——Selenium。

## Selenium简介

Selenium是什么，其实就是一个Web的自动化测试工具。但是呢，我们同时也可以用它来写爬虫，而且Selenium可以让你的爬虫更拟人，就像是真正的用户在操作一样。

## 思路

既然没法拿到详情页的URL，那就只能通过点击下一页这个按钮一页一页的翻了。因此呢，我们就需要从资料检索系统页面一步步跳转到详情页，之后通过`xpath`解析，得到一些格式规范的数据，以及藏品图片的URL，然后把这个数据存到数据库里面，最后再通过图片的URL把图片下载到本地。

后来发现爬着爬着中间老是出错，网页提示已经爬到最后一页了，然后并没有。就又要从数据库里读一遍数据，然后通过计算有多少条数据，来算出是在哪一个详情页报错的，之后再通过页面跳转一步一步访问到那个出错时正在爬取的详情页，之后继续向后爬取数据。

## 代码

```python
# coding: utf-8

from selenium import webdriver
from lxml import etree
import time
import pymysql
import os
import threading
from urllib import request

def get_page(num, data_set):
    # num表示将要爬取第几件文物的信息，而get_page则会调用浏览器到达文物所在的页面
    # data_set是从数据库里读出来的已经爬取完毕的图片URL的集合，这是为了避免有重复
    # 数据进入数据库
    if num >10:
        ten_page_num = (num - 10) // 100 # 如果藏品件数超过110 则需要采用 后十页 按钮
        page_num = (num - ten_page_num * 100) // 10 # 只能是个位数
        rank_num = num - ten_page_num * 100 - page_num * 10
    else:
        ten_page_num = 0
        page_num = 0
        rank_num = num
    print('第{}件藏品  要翻{}个10页  然后再翻{}页  最后是第{}个 '.format(num, ten_page_num, page_num, rank_num + 1))
    driver = webdriver.Chrome()
    driver.get('http://antiquities.npm.gov.tw/') # 器物典藏资料检索系统
    driver.find_element_by_xpath('//*[@id="content"]/div[2]/a').click() # 藏品检索
    driver.find_element_by_xpath('//*[@id="Image4"]').click() # 玻璃器物
    time.sleep(5)
    # 后十页
    for i in range(ten_page_num):
        if i == 0:
            driver.find_element_by_xpath('//*[@id="rptPager_lnkPage_13"]').click()
        else:
            driver.find_element_by_xpath('//*[@id="rptPager_lnkPage_15"]').click()
    # 后一页
    for j in range(page_num):
        if ten_page_num == 0:
            driver.find_element_by_xpath('//*[@id="rptPager_lnkPage_12"]/img').click()
        else:
            driver.find_element_by_xpath('//*[@id="rptPager_lnkPage_14"]/img').click()
    # 第几个藏品
    if rank_num <= 4:
        driver.find_element_by_xpath('//*[@id="ListView1_ctrl0_imgShowImage_{}"]'.format(rank_num)).click() # 进入详情页
    else:
        driver.find_element_by_xpath('//*[@id="ListView1_ctrl1_imgShowImage_{}"]'.format(rank_num)).click()
    # 调用get_data参数 爬取详情页信息
    get_data(driver, data_set)


def get_data(driver, data_set):
    html = etree.HTML(driver.page_source)
    # 爬取详情页信息
    lblCollectionUnicode = html.xpath("//span[@id='lblCollectionUnicode']/text()")[0]
    lblTitleName = html.xpath("//span[@id='lblTitleName']/text()")[0]
    lblRegisterType = html.xpath("//span[@id='lblRegisterType']/text()")[0]
    lblType = html.xpath("//span[@id='lblType']/text()")[0]
    lblFunctions = html.xpath("//span[@id='lblFunctions']/text()")[0]
    lblDimension = html.xpath("//span[@id='lblDimension']/text()")[0]
    lblChineseDisplay = html.xpath("//span[@id='lblChineseDisplay']/text()")[0]
    lblWestDisplay = html.xpath("//span[@id='lblWestDisplay']/text()")[0]
    lblDescription = html.xpath("//span[@id='lblDescription']/text()")[0]
    imgShowImage = "http://antiquities.npm.gov.tw/" + html.xpath("//input[@id='imgShowImage']/@src")[0].replace(
        "&maxW=422&maxH=492", "")
    time.sleep(1)
    if imgShowImage not in data_set:
        # 如果不是重复的数据
        data_set.add(imgShowImage)
        # 调用爬取图片的函数
        get_img_url(imgShowImage, lblTitleName, lblCollectionUnicode)
        # 把数据插入数据库
        insert_data(lblCollectionUnicode, lblTitleName, lblRegisterType, lblType, lblFunctions, lblDimension,
              lblChineseDisplay, lblWestDisplay, lblDescription, imgShowImage)
        print(lblCollectionUnicode, lblTitleName, lblRegisterType, lblType, lblFunctions, lblDimension,
              lblChineseDisplay, lblWestDisplay, lblDescription, imgShowImage)
        # 下一页
    driver.find_element_by_id('btnSearchListNex').click()
    # 调用自身，把driver和更新后的data_set传进去
    get_data(driver, data_set)


def create_table():
    # 创建数据库
    db = pymysql.connect("localhost", "root", "password", "npm", use_unicode=True, charset="utf8")
    cursor = db.cursor()
    cursor.execute("DROP TABLE IF EXISTS jade")
    sql = """CREATE TABLE jade (
             lblCollectionUnicode  CHAR(100) NOT NULL,
             lblTitleName  CHAR(100),
             lblRegisterType CHAR(100),
             lblType CHAR(100),
             lblFunctions CHAR(100),
             lblDimension CHAR(100),
             lblChineseDisplay CHAR(100),
             lblWestDisplay CHAR(100),
             lblDescription VARCHAR(1000),
             imgShowImage CHAR(100)
             )"""
    cursor.execute(sql)



def insert_data(lblCollectionUnicode, lblTitleName, lblRegisterType, lblType, lblFunctions, lblDimension, lblChineseDisplay,
        lblWestDisplay, lblDescription, imgShowImage):
        db = pymysql.connect("localhost", "root", "password", "npm", use_unicode=True, charset="utf8")
        cursor = db.cursor()
        sql = """INSERT INTO jade VALUES (%r, %r, %r, %r, %r, %r, %r, %r, %r, %r);""" % (
            lblCollectionUnicode, lblTitleName, lblRegisterType, lblType, lblFunctions, lblDimension, lblChineseDisplay,
            lblWestDisplay, lblDescription, imgShowImage)
        cursor.execute(sql)
        db.commit()



def get_num():
    db = pymysql.connect("localhost", "root", "password", "npm", use_unicode=True, charset="utf8")
    cursor = db.cursor()
    sql = """SELECT imgShowImage FROM jade"""
    cursor.execute(sql)
    data_set = set(cursor.fetchall())
    get_page(len(data_set), data_set)
    db.commit()


class customThread(threading.Thread):
    def __init__(self, img_url, img_path, img_catalog):
        threading.Thread.__init__(self)
        self.img_url = img_url
        self.img_path = img_path
        self.img_catalog = img_catalog


    def run(self):
        download_img(self.img_url, self.img_path, self.img_catalog)


def download_img(img_url, img_path, img_catalog):
    try:
        response = request.urlopen(img_url)
        img_contents = response.read()
    except:
        print(img_path + '下载出错')
    else:
        # 如果该目录不存在，则新建该目录
        isExists = os.path.exists(img_catalog)
        if not isExists:
            os.makedirs(img_catalog)
        f = open(img_path, 'wb')
        f.write(img_contents)
        f.close()
        print('保存成功>>>>' + img_path)


def get_img_url(img_url, title, _id):
    img_path = 'H:\欧阳宁\\npm\jade\\' + '{}\\'.format(title) +'{}_{}'.format(_id, title) + '.jpg'
    img_catalog = 'H:\欧阳宁\\npm\jade\\' + '{}\\'.format(title)
    customThread(img_url, img_path, img_catalog).start()


if __name__ == '__main__':
    create_table()
    while True:
        try:
            get_num()
        except:
            pass
```

## 总结

还有这么几件事没有做好

1. Selenium这个框架用来爬东西太慢了，应该要试试多进程的。
2. 关于程序不间断持续进行和提供报错信息之间平衡没有处理好。
3. 没有花太多的时间在程序的优化上面，倒是程序运行的太慢。