> 本篇介绍使用requests，lxml开发简单的爬虫

1. 例：获取简书网页正文的内容，并保存到文件

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
'''
# =============================================================================
#      FileName: html_body.py
#          Desc: 获取简书网页正文的内容，并保存到文件
#        Author: Jase Chen
#         Email: xxmm@live.cn
#      HomePage: http://jase.im/
#       Version: 0.0.1
#    LastChange: 2018-05-06 21:06:37
#       History:
# =============================================================================
'''
import requests
from lxml import html
import os

# 获取网页文章正文的内容保存文件

# 设定保存结果的文件名
file_name = os.path.join(os.getcwd(), 'crawl.txt')
headers = {
    'user-agent':
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.139 Safari/537.36'
}
# 获取网页
response = requests.get(
    'https://www.jianshu.com/p/3ea8262b0927', headers=headers)
# 网页转换为lxml可用
selector = html.fromstring(response.text)
with open(file_name, 'w') as f:
    # 获取相应正文处的内容
    for sel in selector.xpath('//div[@class="show-content"]/*/p/text()'):
        # 保存内容
        f.write(sel.strip() + '\n')
print('Processing done...')
```