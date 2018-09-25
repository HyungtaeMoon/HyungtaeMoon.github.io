---
layout: post
title:  "크롤링 해보기(웹툰)"
date:   2018-09-25 04:41:43
author: Moon Hyungtae
categories: Python
tags:	python
cover:  "/assets/instacode.png"
---

`파이썬 크롤링`

```python
import os
import sys
from urllib import parse

import requests
from bs4 import BeautifulSoup

from utils_webtoon import Webtoon


def search_webtoon(keyword):
    file_path = 'data/webtoon_list.html'
    # file_path 경로에 os모듈을 이용하여 저장
    webtoon_url = 'http://comic.naver.com/webtoon/weekday.nhn'
    # 네이버웹툰의 메인화면이며, file_path에 저장될 웹사이트 주소

    if not os.path.exists(file_path):
    #만약 file_path에 아래 데이터가 존재하지 않는다면
        response = requests.get(webtoon_url)
        # 네이버웹툰 HTML을 url로 가져와서 response 변수에 할당
        html = response.text
        # 그리고 이 url은 response.text형식으로 html 변수에 저장
        open(file_path, 'wt').write(html)
        # 그 저장한 html변수는 file_path 경로에 지정된
        # data/webtoon_list.html에 써라

    else:
    # 그게 아니라면
        html = open(file_path, 'rt').read()
        # webtoon_list.html파일을 열어서 읽어라
    soup = BeautifulSoup(html, 'lxml')
    # file_path(webtoon_list.html) 파일은 BeautifulSoup모듈을 사용하여
    # url주소를 lxml 파서를 이용하여 url 수집을 한다
    a_list = soup.select('a.title')
    # a태그(앵커태그)의 title만 리스트형태로 저장한다.(회차별로 저장)
    result_list = list()
    # list형태로 result_list에 저장할 준비
    webtoon_id_set = set()
    # set형태로 webtoon_id_set에 저장할 준비
    # set을 사용하는 이유는 들어오는 데이터의 중복값을 없애기 위해서다

    for a in a_list:
    # 리스트 형태로 가져온 a_list는 for문을 돌려 a변수에
    # 순서대로 차곡차곡 넣어준다.
        href = a.get('href')
        # href를 get하는 이유는 titleId=703846&weekday=tue 형식을
        # 뽑아내기 위해서이다
        query_string = parse.urlsplit(href).query
        # urllib모듈을 사용하여 튜플형태로 반환해준다
        # urlsplit은 튜플 형태로 반환해준다고 구글링해서 봄
        query_dict = dict(parse.parse_qsl(query_string))
        # 추출된 데이터는 딕셔너리 형식으로 qury_dict에 저장
        webtoon_id = query_dict.get('titleId')
        # 딕셔너리 형식의 데이터에서 'titleId'만 get 함수를 이용하여 추출
        title = a.get_text(strip=True)
        # a 에 들어간 데이터 중 text만 공백을 없애고 title 변수에 할당한다
        # titleId는 추출되었다

        if keyword in title:
        # 추출된 이 tr의 데이터들이 keyword안에 있다면
            if webtoon_id in webtoon_id_set:

                continue

            webtoon_id_set.add(webtoon_id)
            result_list.append({
                'webtoon_id': webtoon_id,
                'title': title,
            })

    return result_list

def ini():
    search_keyword = input('검색할 웹툰명을 입력해주세요 :')
    # 검색할 웹툰명을 입력한다
    result_list = search_webtoon(search_keyword)
    # 입력받은 키워드를 통해 search_webtoon 함수 실행해서
    # result_list 변수에 할당해놓는다

    for num, webtoon in enumerate(result_list):
    # enumerate함수를 이용하여 제목과 일치시킨다
        print(f'{num} : {webtoon["title"]}')

    select_webtoon = input("선택 :")

    webtoon1 = Webtoon(result_list[int(select_webtoon)]['webtoon_id'])

    select_webtoon_menu(webtoon1)

def select_webtoon_menu(webtoon):
    while True:
        print('--------------------------------')
        print(f'현재 {webtoon.title} 웹툰이 선택되어 있습니다.')
        print('1. 웹툰 정보 보기')
        print('2. 웹툰 저장하기')
        print('3. 다른 웹툰에서 선택하기')
        print('4. 종료하기')
        select_menu =  input ('선택 :')

        if select_menu is '1':
            print(webtoon.show_info())
        elif select_menu is '2':
            pass
        elif select_menu is '3':
            ini()
        elif select_menu is '4':
            sys.exit(1)
        else:
            print('올바른 입력이 아닙니다. 다시 선택해주세요')


if __name__ == '__main__':
    ini()
```
