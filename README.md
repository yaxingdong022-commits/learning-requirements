# Scrapy 30 天实战训练

## 导师信息

```
┌─────────────────────────────────────────────────────────────┐
│                      你的爬虫导师                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  身份：资深网络爬虫工程师，10 年实战经验                      │
│                                                             │
│  擅长：                                                     │
│  ├─ 框架：Scrapy、Playwright、Selenium、PySpider            │
│  ├─ 请求：Requests、HTTPX、aiohttp、curl_cffi               │
│  ├─ 解析：CSS、XPath、正则、BeautifulSoup、Parsel           │
│  ├─ 反爬：JS 逆向、验证码、指纹、Cookie、代理池              │
│  ├─ 存储：MySQL、MongoDB、Redis、Elasticsearch              │
│  ├─ 分布式：Scrapy-Redis、Celery、消息队列                  │
│  └─ 部署：Docker、Scrapyd、K8s、定时调度                    │
│                                                             │
│  教学风格：                                                  │
│  ├─ 实战派，代码先行                                        │
│  ├─ 不讲废话，直击重点                                      │
│  ├─ 有问必答，但你得先动手试                                │
│  └─ 严格要求，验收不过就重做                                │
│                                                             │
│                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 学习进度

| 天数 | 主题 | 状态 |
|-----|------|------|
| Day 1 | 基础爬虫：选择器、提取数据 | ✅ 已完成 |
| Day 2 | 自动翻页 | ✅ 已完成 |
| Day 3 | 跟踪详情页 | ✅ 已完成 |
| Day 4 | Items + Pipelines | ✅ 已完成 |
| Day 5 | Settings + 反爬 | ✅ 已完成 |
| Day 6 | Ajax 接口 | ✅ 已完成 |
| Day 7 | Middleware 中间件 + 随机 UA | ✅ 已完成 |
| Day 8 | 代理 IP | ⬜ |
| ...   | ...  | ... |

---

## CSS 选择器速查表

### 基础选择器

| 选择器 | 含义 | 示例 |
|-------|------|------|
| `div` | 标签名 | 所有 div |
| `.quote` | class 名 | class="quote" |
| `#main` | id 名 | id="main" |
| `div.quote` | 标签 + class | class="quote" 的 div |

### 层级选择器

| 选择器 | 含义 | 示例 |
|-------|------|------|
| `div a` | 后代（任意层级） | div 下面所有 a |
| `div > a` | 直接子元素（一层） | div 的直接子 a |
| `span.text a` | 多层嵌套 | span.text 里的 a |

### 提取内容

| 选择器 | 含义 | 示例 |
|-------|------|------|
| `::text` | 提取文本 | `span::text` |
| `::attr(href)` | 提取属性 | `a::attr(href)` |
| `::attr(src)` | 提取 src | `img::attr(src)` |

### 属性选择器

| 选择器 | 含义 | 示例 |
|-------|------|------|
| `a[href]` | 有 href 属性 | 有链接的 a |
| `a[href="/page"]` | href 等于 | 精确匹配 |
| `a[href*="/author/"]` | href 包含 | 模糊匹配 |
| `a[href^="/page"]` | href 开头 | 以 /page 开头 |
| `a[href$=".jpg"]` | href 结尾 | 以 .jpg 结尾 |

### 获取方法

| 方法 | 返回 | 示例 |
|-----|------|------|
| `.get()` | 第一个（字符串） | "hello" |
| `.getall()` | 所有（列表） | ["a", "b", "c"] |

---

## 常用命令

```bash
# 创建项目
scrapy startproject 项目名

# 创建爬虫
scrapy genspider 爬虫名 域名

# 运行爬虫
scrapy crawl 爬虫名 -o 文件名.json

# 调试选择器
scrapy shell "网址"
```

---

## 核心代码模板

### Day 1：基础爬虫

```python
import scrapy

class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = ["https://quotes.toscrape.com/"]

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                "text": quote.css("span.text::text").get(),
                "author": quote.css("small.author::text").get(),
            }
```

### Day 2：自动翻页

```python
def parse(self, response):
    # 提取数据
    for quote in response.css("div.quote"):
        yield {"text": quote.css("span.text::text").get()}  
    
    # 翻页
    next_page = response.css("li.next a::attr(href)").get()
    if next_page:
        yield response.follow(next_page, callback=self.parse)
```

### Day 3：跟踪详情页

```python
def parse(self, response):
    for quote in response.css("div.quote"):
        author_url = quote.css('a[href*="/author/"]::attr(href)').get()
        if author_url:
            yield response.follow(
                author_url,
                callback=self.parse_author,
                cb_kwargs={"text": quote.css("span.text::text").get()}
            )


def parse_author(self, response, text):
    yield {
        "text": text,
        "author_born": response.css(".author-born-date::text").get(),
    }
```

### Day 7：Middleware 随机 UA

```python
# middlewares.py
from fake_useragent import UserAgent


class RandomUserAgentMiddleware:
    def __init__(self):
        self.ua = UserAgent()

    def process_request(self, request, spider):
        request.headers['User-Agent'] = self.ua.random
```

```python
# settings.py
DOWNLOADER_MIDDLEWARES = {
    "项目名.middlewares.RandomUserAgentMiddleware": 400,
}
```

---

## Middleware 速查表

| 方法 | 调用时机 | 参数 |
|-----|---------|------|
| `process_request` | 请求发出前 | `(self, request, spider)` |
| `process_response` | 响应返回后 | `(self, request, response, spider)` |
| `process_exception` | 发生异常时 | `(self, request, exception, spider)` |

---

## 常见错误

| 错误 | 原因 | 解决 |
|-----|------|------|
| `url can't be None` | 选择器没找到链接 | 用 shell 调试选择器 |
| `NoneType has no attribute 'strip'` | `.get()` 返回 None | 加判断或用 `or ""` |
| JSON 为空 | 代码没写完整 / 选择器错 | 检查代码和选择器 |
| 只爬了一页 | 翻页逻辑没写 | 加 `yield response.follow` |

---

## 学习规矩

1. 每天一个实战任务
2. 完成后截图验收
3. 遇到问题先看报错，搞不定再问
4. 代码亲手敲，别光看