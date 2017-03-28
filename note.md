# git

+ .git文件过大！删除大文件
在我们日常使用Git的时候，一般比较小的项目，我们可能不会注意到.git 这个文件。
其实， .git文件主要用来记录每次提交的变动，当我们的项目越来越大的时候，我们发现 .git文件越来越大。
很大的可能是因为提交了大文件，如果你提交了大文件，那么即使你在之后的版本中将其删除，但是，实际上，记录中的大文件仍然存在。
为什么呢？仔细想一想，虽然你在后面的版本中删除了大文件，但是Git是有版本倒退功能的吧，那么如果大文件不记录下来，
git拿什么来给你回退呢？但是，.git文件越来越大导致的问题是： 每次拉项目都要耗费大量的时间，并且每个人都要花费那么多的时间。。
git给出了解决方案，使用git branch-filter来遍历git history tree, 可以永久删除history中的大文件，达到让.git文件瘦身的目的。

首先找出git中前五大的文件：

```
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -g | tail -5
```
第一行的字母其实相当于文件的id,用以下命令可以找出id 对应的文件名：

```
git rev-list --objects --all | grep 8f10eff91bb6aa2de1f5d096ee2e1687b0eab007
```
好了，最大的文件找到了。怎么删除呢？

```
git filter-branch --index-filter 'git rm --cached --ignore-unmatch <your-file-name>'
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git fsck --full --unreachable
git repack -A -d
git gc --aggressive --prune=now
git push --force [remote] master
```
首先，里面最重要的两条命令是 git filter-branch 和 gc, filter-branch 真正在清理，但是只运行它也是没用的，需要再删除备份的文件，重新打包之类的，最后的gc命令，
用来收集产生的垃圾，最终清除大文件。
一步到位，再看看你的.git文件，有没有大吃一惊呢！





# scrapy

+ 如何防止蜘蛛被网站Ban掉

    1. 动态设置user agent
    2. 禁用cookies
    3. 设置延迟下载
    4. 使用[google cache](http://www.googleguide.com/cached_pages.html)
    5. 使用IP地址池([Tor project](https://www.torproject.org), VPN和代理IP)
    6. 使用[Crawlera](https://scrapinghub.com/crawlera)

    由于Google cache受国内网络的影响，你懂得；Crawlera的分布式下载，我们可以在下次用一篇专门的文章进行讲解。所以本文主要从动态随机设置user agent、禁用cookies、设置延迟下载和使用代理IP这几个方式。好了，入正题：

    1. 创建middlewares.py
        scrapy代理IP、user agent的切换都是通过DOWNLOADER_MIDDLEWARES进行控制，下面我们创建middlewares.py文件。
```python
[root@bogon cnblogs]# vi cnblogs/middlewares.py
import random
import base64
from settings import PROXIES
class RandomUserAgent(object):
    """Randomly rotate user agents based on a list of predefined ones"""
    def __init__(self, agents):
        self.agents = agents
    @classmethod
    def from_crawler(cls, crawler):
        return cls(crawler.settings.getlist('USER_AGENTS'))
    def process_request(self, request, spider):
        #print "**************************" + random.choice(self.agents)
        request.headers.setdefault('User-Agent', random.choice(self.agents))
class ProxyMiddleware(object):
    def process_request(self, request, spider):
        proxy = random.choice(PROXIES)
        if proxy['user_pass'] is not None:
            request.meta['proxy'] = "http://%s" % proxy['ip_port']
            encoded_user_pass = base64.encodestring(proxy['user_pass'])
            request.headers['Proxy-Authorization'] = 'Basic ' + encoded_user_pass
            print "**************ProxyMiddleware have pass************" + proxy['ip_port']
        else:
            print "**************ProxyMiddleware no pass************" + proxy['ip_port']
            request.meta['proxy'] = "http://%s" % proxy['ip_port']
```
        类RandomUserAgent主要用来动态获取user agent，user agent列表USER_AGENTS在settings.py中进行配置。
        类ProxyMiddleware用来切换代理，proxy列表PROXIES也是在settings.py中进行配置 。

    2. 修改settings.py配置USER_AGENTS和PROXIES

        a). 添加USER_AGENTS
```python
USER_AGENTS = [
    "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; AcooBrowser; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.0; Acoo Browser; SLCC1; .NET CLR 2.0.50727; Media Center PC 5.0; .NET CLR 3.0.04506)",
    "Mozilla/4.0 (compatible; MSIE 7.0; AOL 9.5; AOLBuild 4337.35; Windows NT 5.1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)",
    "Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 2.0.50727; Media Center PC 6.0)",
    "Mozilla/5.0 (compatible; MSIE 8.0; Windows NT 6.0; Trident/4.0; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; .NET CLR 1.0.3705; .NET CLR 1.1.4322)",
    "Mozilla/4.0 (compatible; MSIE 7.0b; Windows NT 5.2; .NET CLR 1.1.4322; .NET CLR 2.0.50727; InfoPath.2; .NET CLR 3.0.04506.30)",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN) AppleWebKit/523.15 (KHTML, like Gecko, Safari/419.3) Arora/0.3 (Change: 287 c9dfb30)",
    "Mozilla/5.0 (X11; U; Linux; en-US) AppleWebKit/527+ (KHTML, like Gecko, Safari/419.3) Arora/0.6",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.2pre) Gecko/20070215 K-Ninja/2.1.1",
    "Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.9) Gecko/20080705 Firefox/3.0 Kapiko/3.0",
    "Mozilla/5.0 (X11; Linux i686; U;) Gecko/20070322 Kazehakase/0.4.5",
    "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko Fedora/1.9.0.8-1.fc10 Kazehakase/0.5.6",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/535.20 (KHTML, like Gecko) Chrome/19.0.1036.7 Safari/535.20",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; fr) Presto/2.9.168 Version/11.52",
]
```

        b): 添加代理IP设置PROXIES
```python
PROXIES = [
    {'ip_port': '111.11.228.75:80', 'user_pass': ''},
    {'ip_port': '120.198.243.22:80', 'user_pass': ''},
    {'ip_port': '111.8.60.9:8123', 'user_pass': ''},
    {'ip_port': '101.71.27.120:80', 'user_pass': ''},
    {'ip_port': '122.96.59.104:80', 'user_pass': ''},
    {'ip_port': '122.224.249.122:8088', 'user_pass': ''},
]
```
            代理IP可以网上搜索一下，上面的代理IP获取自：http://www.xici.net.co/ .

        c): 禁用cookies
```python
            COOKIES_ENABLED=False
```

        d): 设置下载延迟
```python
DOWNLOAD_DELAY=3
```
        保存settings.py

    3. 测试是否成功




+ 给自己的spider设置ip代理

    + I am going to assume that you have already installed scrapy on your system.
    + Install Tor as per the instruction from official documentation. On my mac I used macport to install Tor.
    + Start tor
    + Install polipo using macport.
    + uncomment following lines in /etc/polipo/config or /opt/local/etc/polipo/config file.

```
socksParentProxy = localhost:9050
diskCacheRoot=""
disableLocalInterface=""
```
    start polipo. By default polipo listens on 8123 port and Tor on 9050 port. If you want you may change this port and accordingly adjust settings in config files.

    + Now after basic setup is complete let’s add middleware code in Scrapy to make use of this proxy. + Add a new file called middlewares.py in your project and add following code

```python
import os
import random
from scrapy.conf import settings
class RandomUserAgentMiddleware(object):
    def process_request(self, request, spider):
        ua  = random.choice(settings.get('USER_AGENT_LIST'))
        if ua:
            request.headers.setdefault('User-Agent', ua)

class ProxyMiddleware(object):
    def process_request(self, request, spider):
        request.meta['proxy'] = settings.get('HTTP_PROXY')
```

    + In settings.py add the code shown below

```python
### More comprehensive list can be found at
### http://techpatterns.com/forums/about304.html
USER_AGENT_LIST = [
    'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.7(KHTML, like Gecko) Chrome/16.0.912.36 Safari/535.7',
    'Mozilla/5.0 (Windows NT 6.2; Win64; x64; rv:16.0)Gecko/16.0 Firefox/16.0',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_3) AppleWebKit/534.55.3(KHTML, like Gecko) Version/5.1.3 Safari/534.53.10'
]
HTTP_PROXY = 'http://127.0.0.1:8123'
DOWNLOADER_MIDDLEWARES = {
     'myproject.middlewares.RandomUserAgentMiddleware': 400,
     'myproject.middlewares.ProxyMiddleware': 410,
     'scrapy.contrib.downloadermiddleware.useragent.UserAgentMiddleware': None
    # Disable compression middleware, so the actual HTML pages are cached
}
```

    You are all set to crawl now. Make sure you use the crawler responsibly with sufficient delay and follow the website terms and conditions along with robots.txt rules.


# 设置phantomjs的USER-AGENT
    有些网站的WebServer对User-Agent有限制，可能会拒绝不熟悉的User-Agent的访问，所以，写Web自动化代码可能需要将User-Agent稍微伪装一下，否则可能会被拒绝访问。这里简单记录一下Selenium中使用PhantomJS，设置User-Agent的方法。
    默认情况下，是没有自动设置User-Agent的；设置PhantomJS的user-agent，是要设置“phantomjs.page.settings.userAgent”这个desired_capability.
    Python代码如下：

```python
    '''
    Created on Dec 6, 2013
    @author: Jay
    @summary: Set user-agent before using PhantomJS to get a web page.
    '''
    from selenium import webdriver
    from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

    dcap = dict(DesiredCapabilities.PHANTOMJS)
    dcap["phantomjs.page.settings.userAgent"] = (
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0 "
    )
    driver = webdriver.PhantomJS(executable_path='./phantomjs', desired_capabilities=dcap)
    driver.get("http://dianping.com/")
    cap_dict = driver.desired_capabilities
    for key in cap_dict:
        print '%s: %s' % (key, cap_dict[key])
    print driver.current_url
    driver.quit
```
    执行该脚本打印的输出结果如下：

```python
    jay@Jay-Air:~/workspace/python_study/dp/qa/2013/12 $python user_agent_phantomjs.py 
    rotatable: False
    takesScreenshot: True
    acceptSslCerts: False
    browserConnectionEnabled: False
    javascriptEnabled: True
    driverVersion: 1.0.3
    databaseEnabled: False
    locationContextEnabled: False
    platform: mac-unknown-32bit
    browserName: phantomjs
    version: 1.9.1
    driverName: ghostdriver
    nativeEvents: True
    phantomjs.page.settings.userAgent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:25.0) Gecko/20100101 Firefox/25.0 
    applicationCacheEnabled: False
    webStorageEnabled: False
    proxy: {u'proxyType': u'direct'}
    handlesAlerts: False
    cssSelectorsEnabled: True
    http://www.dianping.com/citylist
```

# scrapy post 提交

```python
yield scrapy.FormRequest(
    url="http://www.medsci.cn/sci/index.do?action=search",
    formdata={
        'fullname': str(ls),
        'bigclass': 'null',
        'smallclass': 'null',
        'impact_factor_b': '',
        'impact_factor_s': '',
        'rank': 'number_rank_b'},
        callback=self.parse_project,
        dont_filter = True
        )
```

# use markdown table 





