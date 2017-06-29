## Reuqests 源码阅读 v0.3.0


### 0x00
2011.2.14 是礼拜一。

这个系列叫源码阅读好了.... 源码剖析这名字太专业，而且万一我说错了呢，本来我还想叫源码扯淡呢...

我喵了一眼tag，觉得每个大版本更新一次好了，这个大版本到下一个版本中迭代的东西，尽量都扒出来说下。即使这样，我也得写个20+篇...

话说我第一篇的反响很好啊！（并不是）， 所以我当然会接着写下去了。

嗯，这次是 requests v0.3.0 版本。

### 0x01

照例先从 HISTORY.rst 开始咯。

```
0.3.0 (2011-02-25)
++++++++++++++++++

* Automatic Authentication API Change
* Smarter Query URL Parameterization
* Allow file uploads and POST data together
* New Authentication Manager System
    - Simpler Basic HTTP System
    - Supports all build-in urllib2 Auths
    - Allows for custom Auth Handlers
```

才过11天，就跳大版本了...

而且...

```
0.2.2 (2011-02-14)
++++++++++++++++++

* Still handles request in the event of an HTTPError. (Issue #2)
* Eventlet and Gevent Monkeypatch support.
* Cookie Support (Issue #1)


0.2.1 (2011-02-14)
++++++++++++++++++

* Added file attribute to POST and PUT requests for multipart-encode file uploads.
* Added Request.url attribute for context and redirects

```

情人节那天更是连更了两个版本... 默默搜了下2011.2.14那天是礼拜一。

颤抖吧屌丝..

扯句题外话，最近能看到的混的风生水起的年轻人都是高产似母猪，bilibili 上面的王老菊更游戏实况，更是变态到日更...一集还两个小时。

果然大力出奇迹。

好的，回归正题，看下跳大版本之间都做了哪些改动。

```
0.2.1 (2011-02-14)
++++++++++++++++++

* Added file attribute to POST and PUT requests for multipart-encode file uploads.
* Added Request.url attribute for context and redirects

```

1. 加了对post 方法文件的支持。嗯，这个要研究下。
2. 加了request.url 的暴露.. 这个应该很简单吧。

```
0.2.2 (2011-02-14)
++++++++++++++++++

* Still handles request in the event of an HTTPError. (Issue #2)
* Eventlet and Gevent Monkeypatch support.
* Cookie Support (Issue #1)
```

1. 当不是200时，依然有返回内容（原来错误状态码时，返回的内容都是None）。
2. 异步的monkeypatch支持，又是我不熟的东西。
3. cookie 支持。

题外话，push上去一天就有两个issue? 这么凶？我去扒一下...

好的，我回来了，的确不是自己提的，一个法国人，一个意大利人。果然当时大家都很需要这玩意。
所以说，要顺势而动 :)


```
0.2.3 (2011-02-15)
++++++++++++++++++

* New HTTPHandling Methods
    - Reponse.__nonzero__ (false if bad HTTP Status)
    - Response.ok (True if expected HTTP Status)
    - Response.error (Logged HTTPError if bad HTTP Status)
    - Reponse.raise_for_status() (Raises stored HTTPError)

```

1. 新的HTTPHandling Methods，什么意思.. 猜测是对返回的Response类做了友好的处理。

```
0.2.4 (2011-02-19)
++++++++++++++++++

* Python 2.5 Support
* PyPy-c v1.4 Support
* Auto-Authentication tests
* Improved Request object constructor

```

1. python2.5 支持，i don't care..
2. pypy1.4 支持， i don't care..
3. 自动认证测试。
4. 改善了 Request 的类结构。

```
0.3.0 (2011-02-25)
++++++++++++++++++

* Automatic Authentication API Change
* Smarter Query URL Parameterization
* Allow file uploads and POST data together
* New Authentication Manager System
    - Simpler Basic HTTP System
    - Supports all build-in urllib2 Auths
    - Allows for custom Auth Handlers
```

1. 自动认证的API做了调整；
2. 更智能的查询参数设置；
3. 允许post数据的同时进行文件上传；
4. 新的认证管理系统 balabala...

果然大版本改动的东西多一点呢... 
┬─┬﻿ ノ( ゜-゜ノ)

喵了一眼README.rst, 没有什么特别的改动。
好的我要享用源码了 ψ(｀∇´)ψ

### 0X02

### v0.2.1 

** 对post文件的支持。**


测试终于加了关于post的测试。

```python
	def test_POSTBIN_GET_POST_FILES(self):

		bin = requests.post('http://www.postbin.org/')
		self.assertEqual(bin.status_code, 200)

		post = requests.post(bin.url, data={'some': 'data'})
		self.assertEqual(post.status_code, 201)

		post2 = requests.post(bin.url, files={'some': StringIO('data')})
		self.assertEqual(post2.status_code, 201)

```

postbin是什么鬼..我去扒一下..

好了，我回来了，真是什么鬼都有，postbin就是给你随便post的一个网站。我访问会直接跳转到 http://requestb.in/

果然之前没有post的测试是没有地方post啊哈哈哈哈...

用ipython 测试了下

```
In [22]: import requests, time

In [23]: r = requests.post('http://requestb.in/xiwxabxi', data={"ts":time.time()})

In [24]: print r.status_code
200

In [25]: print r.content
ok
```

可以看到，现在post过去，返回的状态码是200，而不是201(Created)了。

StringIO 是一个可以把 String 强制转换成 类似于 FILE 类的东西。标准库里面这么描述。

```
	StringIO — Read and write strings as files
```

也就是说，在post和put方法里面加入files 参数来支持，可以是StirngIO类型，也可以是File类型（maybe），上源码细节。

```python
		elif self.method == 'POST':
			if (not self.sent) or anyway:

				if self.files:
					register_openers()
					datagen, headers = multipart_encode(self.files)
					req = _Request(self.url, data=datagen, headers=headers, method='POST')

					if self.headers:
						req.headers.update(self.headers)
				
				else:
					req = _Request(self.url, method='POST')
					req.headers = self.headers

```
加了一层 if self.files 的判断，进入后做了两件事。 1. register_opener(), 2. multipart_encode(self.files)

这两个函数，又是调用了另一个轮子...这两个代码文件直接被copy到requests/packages中。这个轮子的作者是 Chris AtLee，我去他 github 扒了一下，并不能找到这个仓库。感谢他让我几个小时欲仙欲死 ：）

我猜可能2011年，urllib2并不支持文件上传，（现在的版本有看到fileHandler）。他觉得已经有了，就把仓库给删了？或者标准库用的就是他的代码哈哈哈～

他用Mixin的方法重写 httplib.HTTPConnection 类里面的send方法，类的名字叫 StreamHTTPConnection。然后用自己继承的类改了全局的urllib.opener，扒到这里不想看细节了...

好，转战 multipart_encode。当post 一个文件的时候，数据要以MIME 格式分块，客户端生成随机的 boundry来标记一块的开头和结尾，并且需要在 header中加入 Content-Length 来表明文件大小。所以这个函数大致是干这些事情。我把注释写在后面。

```python
def multipart_encode(params, boundary=None, cb=None):
    if boundary is None:
        boundary = gen_boundary()	# 生成随机字符串作为 boundary
    else:
        boundary = urllib.quote_plus(boundary)

    headers = get_headers(params, boundary) 	# 生成Content-Length等头信息
    params = MultipartParam.from_params(params) 

    return multipart_yielder(params, boundary, cb), headers # 返回迭代器和头信息

```

** 加了request.url 的暴露 **
	
在Response 类中加入了 url。
	
```
	self.response.url = resp.url
```

### v0.2.2 

1. 当不是200时，依然有返回内容（原来错误状态码时，返回的内容都是None）。#isssue 2
2. 异步的monkeypatch支持。
3. cookie 支持。#issue 1

** issue2 算是一个小bug, 下面是v0.2.1 和 v0.2.2版本的对比，以get举例。**

v0.2.1

```python
				try:
					opener = self._get_opener()
					resp =  opener(req)

					self.response.status_code = resp.code
					self.response.headers = resp.info().dict
					self.response.content = resp.read()
					self.response.url = resp.url

					success = True

				except urllib2.HTTPError as why:
					self.response.status_code = why.code
```

当 resp = opener(req) 出现问题时（即进入 except 时），只返回了状态码。


v0.2.2

```python
				try:
					resp = opener(req)
					self._build_response(resp)
					success = True

				except urllib2.HTTPError as why:
					self._build_response(why)
					success = False
```
修复问题，封装一个返回函数～

** monkeypatch支持。**

在 core 代码最顶部加了如下部分。

```python
try:
	import eventlet
	eventlet.monkey_patch()
except ImportError:
	pass

if not 'eventlet' in locals():
	try:
		from gevent import monkey
		monkey.patch_all()
	except ImportError:
		pass

```

此外找不到跟这个有关系的东西。难道只要import就好了？这么黑科技？

在知乎上找到了赖勇浩大神回答某个问题时提到的

> 当一个库的代码是纯 Python 的时候，加上 monkey patch 技法，那么这个库使用的就是 gevent.socket 了，从而不需要任何更改就能够获得 gevent 的同步代码异步执行的“超级牛力”。 -- 赖勇浩

问题地址 https://www.zhihu.com/question/29746887 

666，这个内容还需要仔细研究下，先挖个坑


** cookie 支持 **

同样以get 举例

```python
r.cookiejar = cookies
```
加入了cookies 参数

```python
if self.cookiejar:

	cookie_handler = urllib2.HTTPCookieProcessor(cookiejar)
	_handlers.append(cookie_handler)
	opener = urllib2.build_opener(*_handlers)
			return opener.open

```

调用urllib2函数,加入cookie_handler
标准库我就不扒了..

### V0.2.3

```
0.2.3 (2011-02-15)
++++++++++++++++++

* New HTTPHandling Methods
    - Reponse.__nonzero__ (false if bad HTTP Status)
    - Response.ok (True if expected HTTP Status)
    - Response.error (Logged HTTPError if bad HTTP Status)
    - Reponse.raise_for_status() (Raises stored HTTPError)

```

改进了类  Response 的表现形式。来看一下对新的 Response 类的新加的测试内容,第一个测试。

```python
	def test_nonzero_evaluation(self):
		r = requests.get('http://google.com/some-404-url')
		self.assertEqual(bool(r), False)
	
		r = requests.get('http://google.com/')
		self.assertEqual(bool(r), True)
```

 对返回的 Response 类进行 bool 运算，相关实现代码如下。

```python
	def __nonzero__(self):
		"""Returns true if status_code is 'OK'."""
		return not self.error
		
```

`__nonzero__(self):` 为python的内置 magic 方法，当对该类调用bool(object) 时调用该方法。

当调用opener(resq)的时候，出现异常，会更改self.error 变量。

```python
try:
	resp = opener(req)
	self._build_response(resp)
	self.response.ok = True

except urllib2.HTTPError as why:
	self._build_response(why)
	self.response.error = why

```

第二个测试。

```python
def test_request_ok_set(self):
	r = requests.get('http://google.com/some-404-url')
	self.assertEqual(r.ok, False)

```
跟上面的程序类似，当正确发送的时候，会对response.ok 参数进行改变。

第三个测试

```python
def test_status_raising(self):
	r = requests.get('http://google.com/some-404-url')
	self.assertRaises(requests.HTTPError, r.raise_for_status)
	
	r = requests.get('http://google.com/')
	self.assertFalse(r.error)
	r.raise_for_status()

```
相应代码：

```python
	def raise_for_status(self):
		"""Raises stored HTTPError if one exists."""
		if self.error:
			raise self.error
```

这个版本没什么好说的，针对返回的类的细节优化。
我觉得自己很少会对返回的东西做细节的优化，好像它可用，我就接受了。
当真的需要把轮子开放给其他开发者时，那种追求卓越就会让你做的更好。

或许这就是一流代码狗和三流的区别。

### v0.2.4

```
0.2.4 (2011-02-19)
++++++++++++++++++

* Python 2.5 Support
* PyPy-c v1.4 Support
* Auto-Authentication tests
* Improved Request object constructor

```
1,2 两点略过。

 ** 3. 自动验证测试 **
这个版本最主要的改动，就是优化了测试文件。把测试文件改成了继承 TestSite的类（话说这不是常识吗...）

```python

class RequestsTestSuite(unittest.TestCase):
    """Requests test cases."""
    
    def setUp(self):
        pass

    def tearDown(self):
        """Teardown."""
        pass
        
    def test_invalid_url(self):
        self.assertRaises(ValueError, requests.get, 'hiwpefhipowhefopw')

```

补充了权限验证的测试内容，现在这个测试文件算是比较完整的了，get, head, post, 权限，返回类的检测，都有比较完整的测试。

 ** 4. 优化 Request 类结构 **

哈哈哈，我上一遍质疑他的东西，果然在五天后就改过来了～

```python
class Request(object):
    """The :class:`Request` object. It carries out all functionality of
    Requests. Recommended interface is with the Requests functions.
    """
    
    _METHODS = ('GET', 'HEAD', 'PUT', 'POST', 'DELETE')
    
    def __init__(self, url=None, headers=dict(), files=None, method=None,
                 params=dict(), data=dict(), auth=None, cookiejar=None):
        self.url = url
        self.headers = headers
        self.files = files
        self.method = method
        self.params = params
        self.data = data
        self.response = Response()
        
        self.auth = auth
        self.cookiejar = cookiejar
        self.sent = False
        
```

果然写成那样，是因为太年轻呐 （≧∇≦）

所以我还是说，大家都是这么成长起来的，Don't pray for something, fight.

### V0.3.0

```
0.3.0 (2011-02-25)
++++++++++++++++++

* Automatic Authentication API Change
* Smarter Query URL Parameterization
* Allow file uploads and POST data together
* New Authentication Manager System
    - Simpler Basic HTTP System
    - Supports all build-in urllib2 Auths
    - Allows for custom Auth Handlers
```

终于到了v0.3.0了， 写这个真是太耗费体力了。可是一想，我又没有天赋造轮子，万一我写的这个给某些小同学看到，那些小同学很激动，一不小心搞个大新闻，我岂不是也很自豪 :D

** 1. 自动验证API的改变 **

照例先看测试

之前

```python
    def test_AUTH_HTTPS_200_OK_GET(self):
        auth = requests.AuthObject('requeststest', 'requeststest')
        url = 'https://convore.com/api/account/verify.json'
        r = requests.get(url, auth=auth)

        self.assertEqual(r.status_code, 200)


        requests.add_autoauth(url, auth)

        r = requests.get(url)
        self.assertEqual(r.status_code, 200)

        # reset auto authentication
        requests.AUTOAUTHS = []
```

现在

```python
	def test_AUTH_HTTPS_200_OK_GET(self):
        auth = ('requeststest', 'requeststest')
        url = 'https://convore.com/api/account/verify.json'
        r = requests.get(url, auth=auth)

        self.assertEqual(r.status_code, 200)

        r = requests.get(url)
        self.assertEqual(r.status_code, 200)

        # reset auto authentication
        requests.auth_manager.empty()
```

可以看到，

1. 原来需要实例化一个AuthObject对象， 现在只需要一个元组作为参数就可以，使用角度来说方便了很多；
2. 原来需要手动执行 add_autoauth(), 现在第一次auth=auth后，会自动保存；

这样使用减少了很多使用的困难，不用import 各种东西，除了 requests.get

关于实现细节，我放到第四点去说。


** 2. 更智能的查询参数 **

关于这一点更新，体现在这个测试中

```python
    def test_HTTP_200_OK_GET_WITH_MIXED_PARAMS(self):
        heads = {'User-agent': 'Mozilla/5.0'}
        r = requests.get('http://google.com/search?test=true', params={'q': 'test'}, headers=heads)
        self.assertEqual(r.status_code, 200)      
```

代码实现是，在Reuqest.send() 函数中，对url 进行拼接和处理

send中 

```python
if self.method in ('GET', 'HEAD', 'DELETE'):
            req = _Request(self._build_url(self.url, self._enc_data), method=self.method)
```
self._build_url() 

```python
    @staticmethod
    def _build_url(url, data):
        """Build URLs."""
        
        if urlparse(url).query:
            return '%s&%s' % (url, data)
        else:
            if data:
                return '%s?%s' % (url, data)
            else:
                return url
```

调用了标准库 urlparse ，逻辑大概为，如果url 里面有参数，把data的key,value 以&加到后面，否则，以？加到后面。
很简单的逻辑。

** 3. 允许文件上传和post数据同时进行 **

这一点更新，跟上一点差不多。代码如下。

```python
if self.files:
	register_openers()
    if self.data:
    	self.files.update(self.data)
```

挺无聊的，就是把data 加到file 里面233

** 4. 新的认证管理系统 **
我能看到的新的认证系统的功能包括，
1. 可以接受形如元组的参数；
2. 第二次对同一个url 发送请求时，无需再次添加验证；

所以深挖下实现细节。

首先是Request 初始化时，关于Auth 的细节。

```python
class Reuqest(object):

	def __init__(self):
	...
	
		if isinstance(auth, (list, tuple)):
    	        auth = AuthObject(*auth)
        	if not auth:
            	auth = auth_manager.get_auth(self.url)
	        self.auth = auth
        
    ...

```

也就是说，如果没有提供auth 参数时，会去 auth_manager 中拿取当前url 的认证信息。

auth_manager 是一个全局的 AuthManager() 对象。

并且这个类采用了单例的写法，确保不同文件import requests 时，只有一个 auth_manager。

代码如下

```python
class AuthManager(object):
    """Authentication Manager."""
    
    def __new__(cls):
        singleton = cls.__dict__.get('__singleton__')
        if singleton is not None:
            return singleton

        cls.__singleton__ = singleton = object.__new__(cls)

        return singleton

```

这种写法好像和在豆瓣的dongweiming 大神的写法很像。

地址在这里 http://dongweiming.github.io/python-singleton.html ， 例子是做mongo连接的单例。


这个类做了一个认证信息的存储和获取，也就是说，当你第二次请求时，如果没有给auth, 会默认从这个类实例中获取认证信息。

基本在这个函数中，可窥一斑。

```python
def reduce_uri(self, uri, default_port=True):
        """Accept authority or URI and extract only the authority and path."""
        # note HTTP URLs do not have a userinfo component
        parts = urllib2.urlparse.urlsplit(uri)
        if parts[1]:
            # URI
            scheme = parts[0]
            authority = parts[1]
            path = parts[2] or '/'
        else:
            # host or host:port
            scheme = None
            authority = uri
            path = '/'
        host, port = urllib2.splitport(authority)
        if default_port and port is None and scheme is not None:
            dport = {"http": 80,
                     "https": 443,
                     }.get(scheme)
            if dport is not None:
                authority = "%s:%d" % (host, dport)
        return authority, path
```

将 （域名＋端口， 路径） 作为key, 保存 auth 信息，如果请求参数没有 auth,默认都会进来查找一次。



0X03

好的。终于写完了。

很早之前，有人就教育我，知乎上答不了的代码不要强答、追不到的妹子不要强追、看不了的代码不要强看。

如果那样，和咸鱼有什么区别了。

嗯，快来订阅我吧。

不介意任何形式的转载，请注明作者就可以了，汪顺平或者汪小小。
