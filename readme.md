# 一次对知乎搬运工网站的爬虫

## 成品效果图：

![d2b5ca33bd023851](https://static.iculture.cc/wp-content/uploads/2023/06/20230603183851777-694x1024.png)

## 代码：

```python
import glob
import json
import os
import html
import time
import requests
from bs4 import BeautifulSoup
from colorama import init, Fore, Back, Style
import random
import cv2
import keyboard

init(autoreset=True)

get_soup = lambda web_url: BeautifulSoup(requests.get(web_url).text, 'html.parser')


def get_web_content(web_url):
    soup = get_soup(web_url)
    content = soup.find('section', id="output_wrapper_id")
    title = soup.find('h3', attrs={'class': "post-title entry-title"}).get_text()
    return title, '\n' + '\n'.join([p.get_text() for p in content.find_all('p')])


def get_article_list(web_url, json_file_path='zhihu_article_list.json'):
    if os.path.exists(json_file_path):
        with open(json_file_path, 'r', encoding='utf-8') as fp:
            return json.load(fp)

    soup = get_soup(web_url)
    result_set = soup.find('tbody').find_all('a')
    article_list = {result.text: result['href'] for result in result_set}

    with open(json_file_path, 'w', encoding='utf-8') as fp:
        json.dump(article_list, fp)

    return article_list

def get_img(url):
    url = f"{url}"
    response = requests.get(url)
    with open("image.jpg", "wb") as f:
        f.write(response.content)

    img = cv2.imread('image.jpg', 0)  # 读入灰度图

    def img_color_ascii(img, r=3):
        # img: input img
        # r:  raito params #由于不同控制台的字符长宽比不同，所以比例需要适当调整。
        # window cmd：r=3/linux console r=

        grays = "@%#*+=-:. "  # 由于控制台是白色背景，所以先密后疏/黑色背景要转置一下
        gs = 10  # 10级灰度
        # grays2 = "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~i!lI;:,\"^.` "
        # gs2 = 67              #67级灰度

        # 宽（列）和高（行数）
        w = img.shape[1]
        h = img.shape[0]
        ratio = r * float(w) / h  # 调整长宽比-根据终端改变r

        scale = w // 100  # 缩放尺度/取值步长，向下取整，每100/50个像素取一个 值越小图越小(scale 越大)

        for y in range(0, h, int(scale * ratio)):  # 根据缩放长度 遍历高度 y对于h，x对应w
            strline = ''
            for x in range(0, w, scale):  # 根据缩放长度 遍历宽度
                idx = img[y][x] * gs // 255  # 获取每个点的灰度  根据不同的灰度填写相应的 替换字符
                if idx == gs:
                    idx = gs - 1  # 防止溢出
                # 将真彩值利用命令行格式化输出赋予
                color_id = "\033[38;5;%dm%s" % (
                img[y][x], grays[2])  # 输出！
                strline += color_id  # 按行写入控制台
            print(strline)

    img_color_ascii(img)


if __name__ == '__main__':
    get_img("https://picnew12.photophoto.cn/20171117/kuloutouguowangmiankoupsdtoumingsucai-29174408_1.jpg")
    print("欢迎来到知乎文字爬虫！按回车键进行下一步操作")
    input()
    print("开始爬虫任务，按Q键取消爬虫")
    if not os.path.exists("zhihu"):
        os.mkdir("zhihu")
    web_url = 'https://www.zhihuban.ml/2021/12/navigation.html'
    article_list = get_article_list(web_url)
    for title in article_list.keys():
        title, content = get_web_content(article_list[title])
        content = html.unescape(content)  # 将 html 实体转回相应字符
        content = BeautifulSoup(content, 'html.parser').get_text()
        title = ''.join(c if c.isalnum() else '-' for c in title)  # 将非法字符替换为 -
        with open(f'zhihu/{title}.txt', 'w', encoding='utf-8') as f:
            f.write(content)
            color = random.choice([Fore.RED, Fore.GREEN, Fore.YELLOW, Fore.BLUE, Fore.MAGENTA, Fore.CYAN])
            print(color + f'已爬取：{title}')

        # 进度条显示
        progress = list(Back.GREEN + ' ' * 10 + Style.RESET_ALL)
        start_time = time.time()  # 记录开始时间
        for i in range(10):
            if keyboard.is_pressed('q'):
                print(Fore.RED + "按下了Q键，暂停爬取")
                break
            progress[i] = Back.GREEN + ' ' * i + Style.RESET_ALL
            print('\r' + ''.join(progress) + f' {i * 10}% ', end='')
            time.sleep(0.2)
            downloaded_size = (i + 1) * len(content) // 10
            speed = downloaded_size / (time.time() - start_time)
            print(f'{speed / 1024:.2f} KB/s', end='')

        print("\t" + Fore.BLUE + "已完成")
        print('\n')

        if keyboard.is_pressed('q'):
            break

    # 打印方框
    colors = [Fore.RED, Fore.GREEN, Fore.YELLOW, Fore.BLUE, Fore.MAGENTA, Fore.CYAN]
    print(random.choice(colors) + '*' * 60)
    for title in os.listdir('zhihu'):  # 遍历已经完成的文章列表
        color = random.choice(colors)
        print(color + '|' + f'已完成：{title:<50}' + color + '|')
    print(random.choice(colors) + '*' * 60)

    for txt_file in glob.glob(r"zhihu/*.txt"):
        color = random.choice(colors)
        title = os.path.split(txt_file)[1][:-4]
        print(color + f'已生成TXT：{"".join(c if c.isalnum() else "-" for c in title)}')
input("任务已完成，按回车键退出...")
```

![c6cc2ba5a3024202](https://static.iculture.cc/wp-content/uploads/2023/06/20230603184202757.jpg)

 
