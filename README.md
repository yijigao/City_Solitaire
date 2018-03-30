
## 城市获取
### 来源：百度百科，地级市(不包含直辖市)


```python
# 导入必要的包
import requests
from lxml import etree
from bs4 import BeautifulSoup
```


```python
# 发送请求，获取响应
headers = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "zh,zh-CN;q=0.9,ja;q=0.8,en-GB;q=0.7,en;q=0.6,en-US;q=0.5",
    "DNT": "1",
    "Host": "baike.baidu.com",
    "Referer": "https://baike.baidu.com/item/%E5%8E%BF/34322",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"   
}

def get_html(url):
    try:
        r = requests.get(url, timeout=30, headers=headers)
        r.raise_for_status()
        r.encoding = 'utf-8'
        return r.text
    except:
        print("错误! requests code", requests.status_codes)
```


```python
# 解析页面
html = get_html("https://baike.baidu.com/item/%E5%9C%B0%E7%BA%A7%E5%B8%82")
```


```python
# 使用Xpath 获取地级市字段
selector = etree.HTML(html)
city = selector.xpath("*//tr/td/div/a/text()")
```


```python
# 删除空字段以及字段后面的【市】字
city = [i.replace("市", "").strip() for i in city if i != "市"]
```


```python
# 将city写入txt
with open("city.txt", "w") as f:
    for i in city:
        f.write("{}\n".format(i))
    f.close()
```

## 城市接龙


###  考虑谐音，需要涉及拼音
参考： https://zhuanlan.zhihu.com/p/26726297



```python
def chinese_to_pinyin(x):
    # 汉字转拼音
    y = ''
    dic = {}
    with open("unicode_pinyin.txt") as f:
        for i in f.readlines():
            dic[i.split()[0]] = i.split()[1]
    for i in x:
        i = str(i.encode('unicode_escape'))[-5:-1].upper()
        try:
            y += dic[i] + ' '
        except:
            y += 'XXXX ' #非法字符用XXXX代替
    return y
```


```python
def city_exists(x):
    # 判断输入的城市是否在库中
    with open('city.txt','r') as f:
        for i in set(f.readlines()):
            if x == i.strip():
                return True
        return False
```


```python
# 匹配接龙，返回城市
def city_select(x, mode):
    with open('city.txt','r') as f:
        pinyin = chinese_to_pinyin(x[-1])
        base = f.readlines()
        for i in base:
            # 完全匹配模式
            if mode == 0 and i[0] == x[-1]:
                return i
            # 拼音匹配模式
            if mode == 1 and chinese_to_pinyin(i[0])[:-2] == pinyin[:-2]:
                return i
        return None
```


```python
def city(mode = 0):
    mem = []
    while True:
        count = 0
        p = input("请输入城市:")
        if p.strip() == '':
            print("无效城市名，重新输入")
        if city_exists(p) == False:
            print("该城市不存在")
            return 0
        while True:
            t = city_select(p, mode)
            if t != None and (t not in mem):
                count += 1
                mem.append(t)
                print(t)
            else:
                print("匹配完成，找到{}个!\n 输入空格结束程序".format(count))
                break
```


```python
city(mode=1)
```

    请输入城市:深圳
    镇江
    
    匹配完成，找到1个!
     输入空格结束程序
    请输入城市:1
    该城市不存在
    




    0


