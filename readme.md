# һ�ζ�֪�����˹���վ������

## ��ƷЧ��ͼ��

![d2b5ca33bd023851](https://static.iculture.cc/wp-content/uploads/2023/06/20230603183851777-694x1024.png)

## ���룺

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

    img = cv2.imread('image.jpg', 0)  # ����Ҷ�ͼ

    def img_color_ascii(img, r=3):
        # img: input img
        # r:  raito params #���ڲ�ͬ����̨���ַ�����Ȳ�ͬ�����Ա�����Ҫ�ʵ�������
        # window cmd��r=3/linux console r=

        grays = "@%#*+=-:. "  # ���ڿ���̨�ǰ�ɫ�������������ܺ���/��ɫ����Ҫת��һ��
        gs = 10  # 10���Ҷ�
        # grays2 = "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~i!lI;:,\"^.` "
        # gs2 = 67              #67���Ҷ�

        # ���У��͸ߣ�������
        w = img.shape[1]
        h = img.shape[0]
        ratio = r * float(w) / h  # ���������-�����ն˸ı�r

        scale = w // 100  # ���ų߶�/ȡֵ����������ȡ����ÿ100/50������ȡһ�� ֵԽСͼԽС(scale Խ��)

        for y in range(0, h, int(scale * ratio)):  # �������ų��� �����߶� y����h��x��Ӧw
            strline = ''
            for x in range(0, w, scale):  # �������ų��� �������
                idx = img[y][x] * gs // 255  # ��ȡÿ����ĻҶ�  ���ݲ�ͬ�ĻҶ���д��Ӧ�� �滻�ַ�
                if idx == gs:
                    idx = gs - 1  # ��ֹ���
                # �����ֵ���������и�ʽ���������
                color_id = "\033[38;5;%dm%s" % (
                img[y][x], grays[2])  # �����
                strline += color_id  # ����д�����̨
            print(strline)

    img_color_ascii(img)


if __name__ == '__main__':
    get_img("https://picnew12.photophoto.cn/20171117/kuloutouguowangmiankoupsdtoumingsucai-29174408_1.jpg")
    print("��ӭ����֪���������棡���س���������һ������")
    input()
    print("��ʼ�������񣬰�Q��ȡ������")
    if not os.path.exists("zhihu"):
        os.mkdir("zhihu")
    web_url = 'https://www.zhihuban.ml/2021/12/navigation.html'
    article_list = get_article_list(web_url)
    for title in article_list.keys():
        title, content = get_web_content(article_list[title])
        content = html.unescape(content)  # �� html ʵ��ת����Ӧ�ַ�
        content = BeautifulSoup(content, 'html.parser').get_text()
        title = ''.join(c if c.isalnum() else '-' for c in title)  # ���Ƿ��ַ��滻Ϊ -
        with open(f'zhihu/{title}.txt', 'w', encoding='utf-8') as f:
            f.write(content)
            color = random.choice([Fore.RED, Fore.GREEN, Fore.YELLOW, Fore.BLUE, Fore.MAGENTA, Fore.CYAN])
            print(color + f'����ȡ��{title}')

        # ��������ʾ
        progress = list(Back.GREEN + ' ' * 10 + Style.RESET_ALL)
        start_time = time.time()  # ��¼��ʼʱ��
        for i in range(10):
            if keyboard.is_pressed('q'):
                print(Fore.RED + "������Q������ͣ��ȡ")
                break
            progress[i] = Back.GREEN + ' ' * i + Style.RESET_ALL
            print('\r' + ''.join(progress) + f' {i * 10}% ', end='')
            time.sleep(0.2)
            downloaded_size = (i + 1) * len(content) // 10
            speed = downloaded_size / (time.time() - start_time)
            print(f'{speed / 1024:.2f} KB/s', end='')

        print("\t" + Fore.BLUE + "�����")
        print('\n')

        if keyboard.is_pressed('q'):
            break

    # ��ӡ����
    colors = [Fore.RED, Fore.GREEN, Fore.YELLOW, Fore.BLUE, Fore.MAGENTA, Fore.CYAN]
    print(random.choice(colors) + '*' * 60)
    for title in os.listdir('zhihu'):  # �����Ѿ���ɵ������б�
        color = random.choice(colors)
        print(color + '|' + f'����ɣ�{title:<50}' + color + '|')
    print(random.choice(colors) + '*' * 60)

    for txt_file in glob.glob(r"zhihu/*.txt"):
        color = random.choice(colors)
        title = os.path.split(txt_file)[1][:-4]
        print(color + f'������TXT��{"".join(c if c.isalnum() else "-" for c in title)}')
input("��������ɣ����س����˳�...")
```

![c6cc2ba5a3024202](https://static.iculture.cc/wp-content/uploads/2023/06/20230603184202757.jpg)

 
