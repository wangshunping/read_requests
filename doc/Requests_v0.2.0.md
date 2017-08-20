

### 0X00

是的，我开始拆轮子了，我一拆轮子，连我自己都害怕。

从 requests 开始拆，一是因为这轮子被各种人贴上pythonic 的标签；二是这轮子足够好拆；三是这轮子的作者跟我一样是个靠颜值吃饭的。不信？

上图。


<img src="img/author.png" width = "300" height = "300" alt="图片名称" align=center />









### 0X01

这篇文章拆的是v 0.2.0版本。

根据 HISTORY.rst ，requests 是从 0.0.1 版本开始的，但是 github 的版本库里面找不到 v0.0.1版本的了。所以从 v0.2.0 开始。

下面是版本信息。

```
History
-------

0.2.0 (2011-02-14)
++++++++++++++++++

* Birth!


0.0.1 (2011-02-13)
++++++++++++++++++

* Frustration
* Conception

```

2011-02-14， requests v0.2.0正式发布，看日期，情人节呐，这个轮子肯定是送给自己情人节礼物：）

话说我今年送给自己什么情人节礼物来着.... （脑补思索）

可以看到0.01 版本中用到了 frustration ，结果第二天就发了新版本：）

### 0X03

下面是 README.rst 信息

```

Requests: The Simple (e.g. usable) HTTP Module
==============================================

Most existing Python modules for dealing HTTP requests are insane. I have to look up *everything* that I want to do. Most of my worst Python experiences are a result of the various built-in HTTP libraries (yes, even worse than Logging). 

But this one's different. This one's going to be awesome. And simple.

Really simple. 

```

这是readme的第一部分信息。我用我专业的英文六级水平翻译一下。

Requests: 简单好用的HTTP 模块

大部分存在的处理HTTP请求的模块太傻逼了。要搞懂如何使用这些库，我还必须得看一大堆的文档和代码。这些乱七八糟的内置HTTP库对给造成了很不好的体验(是的，比用logging时的体验还要差劲)。

但是！这个库跟其他那些傻逼是不一样的！这个库会变的超级超级棒！嗯，还有简单。

真的超级简单 ：）


哈哈哈，看了这个README 我就笑了，我喜欢狂妄的年轻人。

```
Features
--------

- Extremely simple GET, HEAD, POST, PUT, DELETE Requests
    + Simple HTTP Header Request Attachment
    + Simple Data/Params Request Attachment
- Simple Basic HTTP Authentication
    + Simple URL + HTTP Auth Registry

```

能GET, HEAD, POST, PUT, DELETE， 能添加请求头，能添加参数和数据，能进行简单的HTTP 验证。

嗯，都是基本功能。

```

Usage
-----

It couldn't be simpler. ::

    >>> import requests
    >>> r = requests.get('http://google.com')


HTTPS? Basic Authentication? ::
    
    >>> r = requests.get('https://convore.com/api/account/verify.json')
    >>> r.status_code
    401

    
Uh oh, we're not authorized! Let's add authentication. ::
    
    >>> conv_auth = requests.AuthObject('requeststest', 'requeststest')
    >>> r = requests.get('https://convore.com/api/account/verify.json', auth=conv_auth)
    
    >>> r.status_code
    200 
    
    >>> r.headers['content-type']
    'application/json'
    
    >>> r.content
    '{"username": "requeststest", "url": "/users/requeststest/", "id": "9408", "img": "censored-long-url"}'

```

嗯。跟现在的用法都是一样简洁 ：）

API 部分，安装部分，就直接跳过吧。


### 0X04

requests 代码之前，先从测试代码开始吧。

```
class RequestsTestSuite(unittest.TestCase):
	"""Requests test cases."""
	
	def setUp(self):
		pass

	def tearDown(self):
		"""Teardown."""
		pass
		
	def test_invalid_url(self):
		self.assertRaises(ValueError, requests.get, 'hiwpefhipowhefopw')

	def test_HTTP_200_OK_GET(self):
		r = requests.get('http://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTPS_200_OK_GET(self):
		r = requests.get('https://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTP_200_OK_HEAD(self):
		r = requests.head('http://google.com')
		self.assertEqual(r.status_code, 200)

	def test_HTTPS_200_OK_HEAD(self):
		r = requests.head('https://google.com')
		self.assertEqual(r.status_code, 200)

	def test_AUTH_HTTPS_200_OK_GET(self):
		auth = requests.AuthObject('requeststest', 'requeststest')
		url = 'https://convore.com/api/account/verify.json'
		r = requests.get(url, auth=auth)

		self.assertEqual(r.status_code, 200)

```

凭我多年(半年)TDD的经验，这份测试肯定是代码写完之后补的。并且写完代码之后很开心，写测试的时候满脑子想着快点丢到github 上面去哈哈哈哈，只测了get 和 head ，当然也可能他找不到 url去测post :)

self.assertRaises 这个用法我没用过，去翻了一个标准库文档，文档里面有这样一个例子，还是很好玩的：

```
	def test_split(self):
      # check that s.split fails when the separator is not a string
      with self.assertRaises(TypeError):
          s.split(2)

```
函数定义：
assertRaises(exc, fun, *args, **kwds)


### 0x05

终于开始撸代码了，requests 包只包含一个__init__.py 和 core.py, core.py 405行。

当使用 requests.get('www.baidu.com')时，直接到requests 包定义的相应函数，以 get 举例。

```
def get(url, params={}, headers={}, auth=None):
	"""Sends a GET request. Returns :class:`Response` object.

	:param url: URL for the new :class:`Request` object.
	:param params: (optional) Dictionary of GET Parameters to send with the :class:`Request`.
	:param headers: (optional) Dictionary of HTTP Headers to sent with the :class:`Request`.
	:param auth: (optional) AuthObject to enable Basic HTTP Auth.
	"""
	
	r = Request()
	
	r.method = 'GET'
	r.url = url
	r.params = params
	r.headers = headers
	r.auth = _detect_auth(url, auth)
	
	r.send()
	
	return r.response

```
先会实例化一个 Request, Request 类定义在47行。

```
class Request(object):
	"""The :class:`Request` object. It carries out all functionality of
	Requests. Recommended interface is with the Requests functions.
	
	"""
	
	_METHODS = ('GET', 'HEAD', 'PUT', 'POST', 'DELETE')
	
	def __init__(self):
		self.url = None
		self.headers = dict()
		self.method = None
		self.params = {}
		self.data = {}
		self.response = Response()
		self.auth = None
		self.sent = False
		
	def __repr__(self):
		try:
			repr = '<Request [%s]>' % (self.method)
		except:
			repr = '<Request object>'
		return repr

```

接下来初始化这个类的各个属性。 r.method = 'GET' , r.url = url ....

在 `__setattr__`  对method做了限制。

```
	def __setattr__(self, name, value):
		if (name == 'method') and (value):
			if not value in self._METHODS:
				raise InvalidMethod()
		
		object.__setattr__(self, name, value)
```

这也是一个好玩的用法，我比较水啦，如果是我，我会直接在类初始化的时候搞定这些...
就像下面这样。

```
class Request(object):
	"""The :class:`Request` object. writtern by Lionel Wang	
	"""
	
	_METHODS = ('GET', 'HEAD', 'PUT', 'POST', 'DELETE')
	
	def __init__(self, url, method, data, params):
		self.url = url
		self.headers = dict()
		self.method = method
		self.params = params
		self.data = data
		self.response = Response()
		self.auth = None
		self.sent = False
		
		if self.method not in _METHODS:
			raise InvalidMethod
```

有没有高人跟我说下这样的好处？

```
r,auth = _detect_auth(url, auth)
```

权限认证，如果需要的话 ：）

所以重要的逻辑都在 r.send() 里面了，继续～

在Requests.send() 里面，对 method 的不同，做了不一样的处理，我们只看get。

```
def send(self, anyway=False):
		"""Sends the request. Returns True of successfull, false if not.
		    If there was an HTTPError during transmission,
		    self.response.status_code will contain the HTTPError code.

		    Once a request is successfully sent, `sent` will equal True.
		
		    :param anyway: If True, request will be sent, even if it has
		    already been sent.
		"""
		self._checks()

		success = False
		
		if self.method in ('GET', 'HEAD', 'DELETE'):
			if (not self.sent) or anyway:

				# url encode GET params if it's a dict
				if isinstance(self.params, dict):
					params = urllib.urlencode(self.params)
				else:

					params = self.params

				req = _Request(("%s?%s" % (self.url, params)), method=self.method)

				if self.headers:
					req.headers = self.headers

				opener = self._get_opener()

				try:
					resp = opener(req)
					self.response.status_code = resp.code
					self.response.headers = resp.info().dict
					if self.method.lower() == 'get':
						self.response.content = resp.read()

					success = True
				except urllib2.HTTPError, why:
					self.response.status_code = why.code
					
		self.sent = True if success else False
		
		return success
```
self._checks() 是封装了url是否非None 的函数。pass

我估摸着 `anyway` 是作者自己用来测试的 ：）， not self.send 总是会通过if的。

正如作者所说，这个版本封装了urllib, urllib2 的方法，像我这种被reqeusts 宠坏了的人，为了拆他，滚去翻urllib2的文档了 （；￣ェ￣）

```
		if isinstance(self.params, dict):
			params = urllib.urlencode(self.params)
		else:

			params = self.params

		req = _Request(("%s?%s" % (self.url, params)), method=self.method)
```

如果传过来parms 形如 

```
parms ={'age':23, 'name':wsp}, url='www.baidu.com'


```

将他转换成 www.baidu.com?age=23&name=wsp

urllib.urlencode(query, [,doseq])

定义为 Convert a mapping object or a sequence of two-element tuples to a “percent-encoded” string,
将字典或者两个元素的元组，转换成有%的字符串。


_Request 是继承了 urllib2.Requsts 的类。
代码如下

```
class _Request(urllib2.Request):
	"""Hidden wrapper around the urllib2.Request object. Allows for manual
	setting of HTTP methods.
	"""
	
	def __init__(self, url,
					data=None, headers={}, origin_req_host=None,
					unverifiable=False, method=None):
		urllib2.Request.__init__( self, url, data, headers, origin_req_host,
								  unverifiable)
	   	self.method = method

	def get_method(self):
		if self.method:
			return self.method

		return urllib2.Request.get_method(self)
```

`self._get_opener()`, 如果需要验证，根据 urllib提供的函数，进行请求的验证，如果不需要，则直接返回 urllib.urlopen 。

接着 `resp = opener(req)` , 等于 `resp = urllib2.urlopen(urllib.Reuqest())`了。即是调用标准库发送请求的真正的方法了。

最后组装返回的类，没什么好说的了

```
	self.response.status_code = resp.code
	self.response.headers = resp.info().dict
	self.response.content = resp.read()
```

不过try except 写的很好玩。

```
except urllib2.HTTPError, why:
	self.response.status_code = why.code
```

诚哥说pep8已经不推荐写逗号了，用as, 至少我的ide 会报一个warning, 我想说的是把 `ex` 写出 `why`, 
这样写好像代码可读性又高了一丢丢哈哈哈，我以后也写成why.

嗯。到了这里 requests v0.2.0 算是看完了～～


### 0X06 后记


1. 我喵了一眼现在的版本，已经不是基于 urllib 了，肯定会有意思的多；
2. 这个时候的作者还是很逗比的嘛～ 很多地方都可以抽象的更好一点，但是他直接 copy himself 了，所以我说啊，年轻人，不用怂，有什么想法，赶紧写，出名要趁早，有什么地方有问题，可以给我提issue ，给我看看嘛哈哈哈；
3. requests 的这个系列我会继续写啊，我对这个小鲜肉变成大叔的历程很感兴趣啊；
4. 当然啦，每个版本我都会checkout 去看看，但是只会写改动比较大的版本了；
5. 都看到这里，肯定是真爱啦，赶紧关注我公众号啦混蛋  (((o(*ﾟ▽ﾟ*)o)))
