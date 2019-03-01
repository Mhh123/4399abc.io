## 1 selenium+phantomjs

    what is selenium?
    是浏览器的自动化测试工具，自动化，
    通过selenium去写代码，执行代码，
    操作浏览器自动化的执行一些动作。
    
    selenium如何操作谷歌浏览器？
        selenium去操作一个谷歌浏览器的驱动，
        由这个驱动间接控制浏览器执行一些动作
    谷歌驱动下载地址：
      http://chromedriver.storage.googleapis.com/index.html
    谷歌驱动和浏览器版本关系映射表
      http://blog.csdn.net/huilan_same/article/details/51896672


  ```
 find_element_by_id

   find_elements_by_name

   find_elements_by_xpath

   find_elements_by_tag_name

   find_elements_by_class_name

   find_elements_by_css_selector

   find_elements_by_link_text

  ```



    phantomjs
        what is phantomjs?
        是一款浏览器，无界面浏览器，即是浏览器，就要浏览器的功能。
        响应的内容是html，css, js,浏览器可以将这些代码翻译成图文并茂的形式，
        并且最重要的是可以执行js.
    
        在爬取网页的过程中，经常会有很多标签是通过js动态生成的，
        处理方式2种:
        1)  抓接口
        2)  接口抓取不到，或者非常复杂
            使用selenium+phantomjs
            (phantomjs是浏览器，可以得到js执行之后的代码)


    使用phantomjs
    见代码
    		模拟滚动条滚动到底部


## 2  headlesschrome  

也是一款无界面浏览器，后来慢慢替代了phantomjs

    from selenium.webdriver.chrome.options import Options
    chrome_options = Options()
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-gpu')
    见代码
## 3  requests     (下载器)

    what is requests?
    是一个第三方库，功能和urllib是一样的，是模拟http发送请求的，
    是对urllib的一层封装，提供更加人性化的方式
    
    安装：pip install requests
    
    官网:
        http://docs.python-requests.org/zh_CN/latest/index.html
    
    get请求
        r = requests.get(url=url, headers=headers)
    响应对象的方法和属性
        r.text      获取字符串内容
        r.content   获取字节内容
        r.headers
        r.url
        r.status_code
    带参数get
        data是一个字典
        r = requests.get(url=url, headers=headers, params=data)
    
    post请求
     data是表单数据
     r = requests.post(url=url, headers=headers, data=data)
    代理
        proxy = {'http:' 'ip+port'}
        r = requests.get(url=url, headers=headers, proxies=proxy)
    会话
        s = requests.Session()
        所有的请求都用s进行发送
        s.post(0
        s.get()