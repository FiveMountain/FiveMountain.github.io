---
title: Python 爬虫+GUI 获取豆瓣电影一周口碑榜信息
date: 2022-05-20 20:37:52
tags:
---

# 说明
Python课设
爬虫和PyQt5都是自己在网上学的
单纯记录一下

# 界面
![aoo-ui](app-ui.png)
# 代码
```python
import sys
from PyQt5.QtGui import *
from PyQt5.QtWidgets import *
import requests
import bs4
import os
import time

# 保存电影封面
# 无法直接从网页获取，因此须先保存在本地
folder_path = './poster/'

url = "https://movie.douban.com/chart"
headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) \
                Chrome/83.0.4103.97 Safari/537.36'}

# 获取网页信息
def get_res():
    res = requests.get(url, headers=headers)
    soup = bs4.BeautifulSoup(res.text, 'lxml')

    return soup

# 用于获取所有电影详情页的链接
def get_info_urls():
    soup = get_res()

    # 用于保存榜上电影详情页的网址
    # 以此进入详情页来获取电影评分及简介
    target_urls = []

    targets = soup.find_all("div", class_="name")
    for each in targets:
        target_urls.append(each.a.get('href'))

    return target_urls

# 此函数用于从网页获取所需数据
def find_movies():
    soup = get_res()

    # 保存电影名
    movies = []

    targets = soup.find_all("div", class_="name")
    for each in targets:
        # 去除空格及换行符 并保存
        fix_name = each.a.text.replace(" ", "")
        final_name = fix_name.replace('\n', '')
        movies.append(final_name)

    # 下面将遍历 target_urls 中每一个链接，获取评分及简介
    ranks = []      # 保存评分
    messages = []   # 保存简介

    index = 1   # 保存图片时用作图片名
    target_urls = get_info_urls()

    for ur in target_urls:
        res = requests.get(ur, headers=headers)
        each_movie = bs4.BeautifulSoup(res.text, 'lxml')

        # 获取评分
        rank = each_movie.find("strong", class_="ll rating_num").text
        ranks.append(' 评分: %s ' % rank)

        # 获取简介
        message = each_movie.find("span", property="v:summary").text
        messages.append(message.strip())

        # 获取封面
        poster = each_movie.find("div", id="mainpic").a.img

        # 用于保存图片的文件夹若不存在则创建
        if not os.path.exists(folder_path):
            os.mkdir(folder_path)

        # 保存封面
        html = requests.get(poster.get('src'))
        img_name = folder_path + str(index) + '.jpg'
        index += 1
        with open(img_name, 'wb') as f:
            f.write(html.content)
            f.flush()
        time.sleep(1)

    # result 用于整合获取的信息
    result = []

    length = len(movies)
    for i in range(length):
        result.append(movies[i] + '\t' + ranks[i] + '\t' + messages[i] + '\t')

    return result


class Example(QWidget):
    def __init__(self):
        super().__init__()

        self.init_ui()

    # 用于将页面居中显示
    def center(self):
        qr = self.frameGeometry()
        cp = QDesktopWidget().availableGeometry().center()
        qr.moveCenter(cp)
        self.move(qr.topLeft())

    # 设置页面布局及内容
    def init_ui(self):
        results = find_movies()

        # 初始化
        # 封面、电影名、评分为 QLabel  简介为 QTextBrowser
        movie_posters = []      # 封面
        movie_names = []        # 电影名
        movie_ranks = []        # 评分
        movie_messages = []     # 简介

        for i in range(10):
            movie_posters.append(QLabel())
        for i in range(10):
            movie_names.append(QLabel())
        for i in range(10):
            movie_ranks.append(QLabel())
        for i in range(10):
            movie_messages.append(QTextBrowser())

        grid = QGridLayout()

        # 空间分配
        for i in range(10):
            grid.addWidget(movie_posters[i], i * 2 + 1, 0, 2, 0)
            grid.addWidget(movie_names[i], i * 2 + 1, 1)
            grid.addWidget(movie_ranks[i], i * 2 + 2, 1)
            grid.addWidget(movie_messages[i], i * 2 + 1, 2, 2, 2)

        # 添加内容

        count = 0     # count用于遍历所有模块

        # 获取所有电影详情页的链接，用于将电影名形成超链接
        urls = get_info_urls()

        # 遍历10个results  即口碑榜10个电影的所有内容
        for re in results:
            # result = name + rank + message

            # 将单个电影的三部分内容分离
            result = re.strip().split('\t')

            # 分别对三项内容赋值     电影名为超链接，可进入电影详情页
            movie_names[count].setText("<a href='" + urls[count] + "'>" + result[0] + "</a>")
            movie_names[count].setOpenExternalLinks(True)   # 设置为允许访问超链接

            movie_ranks[count].setText(result[1])
            movie_messages[count].setText(result[2])

            # 电影封面显示
            img_name = folder_path + str(count + 1) + '.jpg'
            pix = QPixmap(img_name)

            # 设置封面规格
            movie_posters[count].resize(60, 85)
            movie_posters[count].setFixedSize(60, 85)

            movie_posters[count].setPixmap(pix)
            movie_posters[count].setScaledContents(True)   # 图片大小自适应

            # 这一部电影的所有内容布置完毕 进入下一电影模块
            count += 1

        self.setLayout(grid)

        # 窗口设置
        self.setGeometry(0, 0, 530, 970)
        self.center()
        self.setWindowTitle("电影一周口碑榜")
        self.show()

if __name__ == '__main__':
    app = QApplication(sys.argv)
    ex = Example()
    sys.exit(app.exec_())

```
