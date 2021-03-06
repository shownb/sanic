# 路由

路由允许用户为不同的URL路径指定处理的函数。

一个基本的路由应该像下面这样, `app`是`Sanic`类的一个实例:

```python
from sanic.response import json

@app.route("/")
async def test(request):
    return json({ "hello": "world" })
```

当这个url `http://server.url/` 被访问的时候 (the base url of the server), 最终`/`会被路由器匹配到这个处理函数, `test`, 然后这个函数会返回 一个 JSON 对象.

Sanic的处理函数一定要用这样的语法规则被定义`async def`, 因为他们是异步函数来的.

## Request参数

Sanic原生带有一个基本的路由器来支持request的参数.

为了指定一个参数, 用<>号包围它: `<PARAM>`.Request的参数会作为关键字参数被传到路由处理函数里面.

```python
from sanic.response import text

@app.route('/tag/<tag>')
async def tag_handler(request, tag):
	return text('Tag - {}'.format(tag))
```

为了指定参数类型, 在引号里面紧跟参数名字后面加入这样的`:type`. 如果参数匹配不到指定的类型, Sanic会抛出`NotFound`异常, 在url的页面会出现 `404: Page not found` 的错误

```python
from sanic.response import text

@app.route('/number/<integer_arg:int>')
async def integer_handler(request, integer_arg):
	return text('Integer - {}'.format(integer_arg))

@app.route('/number/<number_arg:number>')
async def number_handler(request, number_arg):
	return text('Number - {}'.format(number_arg))

@app.route('/person/<name:[A-z]+>')
async def person_handler(request, name):
	return text('Person - {}'.format(name))

@app.route('/folder/<folder_id:[A-z0-9]{0,4}>')
async def folder_handler(request, folder_id):
	return text('Folder - {}'.format(folder_id))

```

## HTTP请求类型

按默认来说, 一条定义了URL的路由只会获得GET请求.但是呢, 装饰器`@app.route`接受可选参数,`methods`,这就允许处理函数可以根据任何HTTP的methods来工作.

```python
from sanic.response import text

@app.route('/post', methods=['POST'])
async def post_handler(request):
	return text('POST request - {}'.format(request.json))

@app.route('/get', methods=['GET'])
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

```

这里也提供一个`host`参数(可以是一个列表或者字符串). This restricts a route to the host or hosts provided. If there is a also a route with no host, it will be the default.

```python
@app.route('/get', methods=['GET'], host='example.com')
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

# if the host header doesn't match example.com, this route will be used
@app.route('/get', methods=['GET'])
async def get_handler(request):
	return text('GET request in default - {}'.format(request.args))
```

There are also shorthand method decorators:

```python
from sanic.response import text

@app.post('/post')
async def post_handler(request):
	return text('POST request - {}'.format(request.json))

@app.get('/get')
async def get_handler(request):
	return text('GET request - {}'.format(request.args))

```
## `add_route`方法

正如我们所见, 路由常常用`@app.route`装饰器来指定.
However, this decorator is really just a wrapper for the `app.add_route`
method, which is used as follows:

```python
from sanic.response import text

# Define the handler functions
async def handler1(request):
	return text('OK')

async def handler2(request, name):
	return text('Folder - {}'.format(name))

async def person_handler2(request, name):
	return text('Person - {}'.format(name))

# Add each handler function as a route
app.add_route(handler1, '/test')
app.add_route(handler2, '/folder/<name>')
app.add_route(person_handler2, '/person/<name:[A-z]>', methods=['GET'])
```

## URL building with `url_for`

Sanic provides a `url_for` method, to generate URLs based on the handler method name. This is useful if you want to avoid hardcoding url paths into your app; instead, you can just reference the handler name. For example:

```python
@app.route('/')
async def index(request):
    # generate a URL for the endpoint `post_handler`
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return redirect(url)


@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return text('Post - {}'.format(post_id))
```

Other things to keep in mind when using `url_for`:

- Keyword arguments passed to `url_for` that are not request parameters will be included in the URL's query string. For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', arg_two='two')
# /posts/5?arg_one=one&arg_two=two
```
- Multivalue argument can be passed to `url_for`. For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'])
# /posts/5?arg_one=one&arg_one=two
```
- Also some special arguments (`_anchor`, `_external`, `_scheme`, `_method`, `_server`) passed to `url_for` will have special url building (`_method` is not support now and will be ignored). For example:
```python
url = app.url_for('post_handler', post_id=5, arg_one='one', _anchor='anchor')
# /posts/5?arg_one=one#anchor

url = app.url_for('post_handler', post_id=5, arg_one='one', _external=True)
# //server/posts/5?arg_one=one
# _external requires passed argument _server or SERVER_NAME in app.config or url will be same as no _external

url = app.url_for('post_handler', post_id=5, arg_one='one', _scheme='http', _external=True)
# http://server/posts/5?arg_one=one
# when specifying _scheme, _external must be True

# you can pass all special arguments one time
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'], arg_two=2, _anchor='anchor', _scheme='http', _external=True, _server='another_server:8888')
# http://another_server:8888/posts/5?arg_one=one&arg_one=two&arg_two=2#anchor
```
- All valid parameters must be passed to `url_for` to build a URL. If a parameter is not supplied, or if a parameter does not match the specified type, a `URLBuildError` will be thrown.

## WebSocket routes

Routes for the WebSocket protocol can be defined with the `@app.websocket`
decorator:

```python
@app.websocket('/feed')
async def feed(request, ws):
    while True:
        data = 'hello!'
        print('Sending: ' + data)
        await ws.send(data)
        data = await ws.recv()
        print('Received: ' + data)
```

Alternatively, the `app.add_websocket_route` method can be used instead of the
decorator:

```python
async def feed(request, ws):
    pass

app.add_websocket_route(my_websocket_handler, '/feed')
```

Handlers for a WebSocket route are passed the request as first argument, and a
WebSocket protocol object as second argument. The protocol object has `send`
and `recv` methods to send and receive data respectively.

WebSocket support requires the [websockets](https://github.com/aaugustin/websockets)
package by Aymeric Augustin.


## About `strict_slashes`

You can make `routes` strict to trailing slash or not, it's configurable.

```python

# provide default strict_slashes value for all routes
app = Sanic('test_route_strict_slash', strict_slashes=True)

# you can also overwrite strict_slashes value for specific route
@app.get('/get', strict_slashes=False)
def handler(request):
    return text('OK')

# It also works for blueprints
bp = Blueprint('test_bp_strict_slash', strict_slashes=True)

@bp.get('/bp/get', strict_slashes=False)
def handler(request):
    return text('OK')

app.blueprint(bp)
```

## User defined route name

You can pass `name` to change the route name to avoid using the default name  (`handler.__name__`).

```python

app = Sanic('test_named_route')

@app.get('/get', name='get_handler')
def handler(request):
    return text('OK')

# then you need use `app.url_for('get_handler')`
# instead of # `app.url_for('handler')`

# It also works for blueprints
bp = Blueprint('test_named_bp')

@bp.get('/bp/get', name='get_handler')
def handler(request):
    return text('OK')

app.blueprint(bp)

# then you need use `app.url_for('test_named_bp.get_handler')`
# instead of `app.url_for('test_named_bp.handler')`

# different names can be used for same url with different methods

@app.get('/test', name='route_test')
def handler(request):
    return text('OK')

@app.post('/test', name='route_post')
def handler2(request):
    return text('OK POST')

@app.put('/test', name='route_put')
def handler3(request):
    return text('OK PUT')

# below url are the same, you can use any of them
# '/test'
app.url_for('route_test')
# app.url_for('route_post')
# app.url_for('route_put')

# for same handler name with different methods
# you need specify the name (it's url_for issue)
@app.get('/get')
def handler(request):
    return text('OK')

@app.post('/post', name='post_handler')
def handler(request):
    return text('OK')

# then
# app.url_for('handler') == '/get'
# app.url_for('post_handler') == '/post'
```

## 为静态文件生成URL

You can use `url_for` for static file url building now.
If it's for file directly, `filename` can be ignored.

```python

app = Sanic('test_static')
app.static('/static', './static')
app.static('/uploads', './uploads', name='uploads')
app.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')

bp = Blueprint('bp', url_prefix='bp')
bp.static('/static', './static')
bp.static('/uploads', './uploads', name='uploads')
bp.static('/the_best.png', '/home/ubuntu/test.png', name='best_png')
app.blueprint(bp)

# then build the url
app.url_for('static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='static', filename='file.txt') == '/static/file.txt'
app.url_for('static', name='uploads', filename='file.txt') == '/uploads/file.txt'
app.url_for('static', name='best_png') == '/the_best.png'

# blueprint url building
app.url_for('static', name='bp.static', filename='file.txt') == '/bp/static/file.txt'
app.url_for('static', name='bp.uploads', filename='file.txt') == '/bp/uploads/file.txt'
app.url_for('static', name='bp.best_png') == '/bp/static/the_best.png'

```
