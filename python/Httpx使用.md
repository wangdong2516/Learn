<center><h1>Httpx使用

> 安装

1. pip安装

```shell
pip install httpx
```

> 使用

1. 同步语法

```python
import httpx

response = httpx.get('http://www.baidu.com')

# 状态码
response.status_code

# 响应头
response.headers

# 响应体(解码过的)
response.text
```

2. 异步语法

```python

```

3. url传参

```python
params = {'name': 'wangdong', 'age': 12}
res = httpx.get('url', params=params)

# 传递相同的参数名
params = {"name": 'wangdong' , 'name': 'sss'}
res = httpx.get('url', params)
```

4. 请求体传参

```python
data = {key: value}

httpx.post('url', data=data)
```

5. 获取响应体中的json数据

```python
>>> r = httpx.get('https://api.github.com/events')
>>> r.json()
[{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...' ...  }}]
```

1. 查看生成的url

```python
res.url
```

![image-20201030174618835](/Users/wangdong/Library/Application Support/typora-user-images/image-20201030174618835.png)

6. 其他形式的请求

```python
>>> r = httpx.put('https://httpbin.org/put', data={'key': 'value'})
>>> r = httpx.delete('https://httpbin.org/delete')
>>> r = httpx.head('https://httpbin.org/get')
>>> r = httpx.options('https://httpbin.org/get')
```

> 自定义请求头

```python
>>> url = 'http://httpbin.org/headers'
>>> headers = {'user-agent': 'my-app/0.0.1'}
>>> r = httpx.get(url, headers=headers)
```

> 发送表单数据

```python
>>> data = {'key1': 'value1', 'key2': 'value2'}
>>> r = httpx.post("https://httpbin.org/post", data=data)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}


>>> data = {'key1': ['value1', 'value2']}
>>> r = httpx.post("https://httpbin.org/post", data=data)
>>> print(r.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
}
```

> 文件上传

```python
>>> files = {'upload-file': open('report.xls', 'rb')}
>>> r = httpx.post("https://httpbin.org/post", files=files)
>>> print(r.text)
{
  ...
  "files": {
    "upload-file": "<... binary content ...>"
  },
  ...
}

# 指定上传的文件名
>>> files = {'upload-file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel')}
>>> r = httpx.post("https://httpbin.org/post", files=files)
>>> print(r.text)
{
  ...
  "files": {
    "upload-file": "<... binary content ...>"
  },
  ...
}
```

