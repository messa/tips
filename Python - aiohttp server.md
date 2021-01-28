aiohttp web server cheatsheet
=============================

Homepage, documentation: https://aiohttp.readthedocs.io/en/stable/

API reference of most important objects:
[aiohttp.web.Request](https://aiohttp.readthedocs.io/en/stable/web_reference.html#aiohttp.web.Request), 
[aiohttp.web.Response](https://aiohttp.readthedocs.io/en/stable/web_reference.html#aiohttp.web.Response)

Github: https://github.com/aio-libs/aiohttp

StackOverflow: https://stackoverflow.com/questions/tagged/aiohttp?sort=frequent


Installation
------------

```shell
$ pip install aiohttp cchardet aiodns
```


Hello world
-----------

From documentation:

```python
from aiohttp import web

async def handle(request):
    name = request.match_info.get('name', "Anonymous")
    text = "Hello, " + name
    return web.Response(text=text)

app = web.Application()
app.add_routes([web.get('/', handle),
                web.get('/{name}', handle)])
web.run_app(app)
```

Function `run_app` is a utility function for running an application, serving it until keyboard interrupt and performing a Graceful shutdown.
- reference: https://docs.aiohttp.org/en/stable/web_reference.html#aiohttp.web.run_app
- source code: [aiohttp/web.py](https://github.com/aio-libs/aiohttp/blob/master/aiohttp/web.py#L375)
- signature:
`run_app(app, *, host=None, port=None, path=None, sock=None, shutdown_timeout=60.0, ssl_context=None, print=print, backlog=128, access_log_class=aiohttp.helpers.AccessLogger, access_log_format=aiohttp.helpers.AccessLogger.LOG_FORMAT, access_log=aiohttp.log.access_logger, handle_signals=True, reuse_address=None, reuse_port=None)`


### How to start server from async code

You may have notices that the function `run_app(app)` is synchronous.
It manages the asyncio event loop and the whole lifecycle of your program.
Sometimes that's not what you want – you have already running asyncio code
(perhaps using [`asyncio.run`](https://docs.python.org/3/library/asyncio-task.html#asyncio.run))
and just need a way how to start aiohttp server as a coroutine.

```python
async def run_app(app, bind_host='', bind_port=8080):
    runner = web.AppRunner(app)
    await runner.setup()
    site = web.TCPSite(runner, bind_host, bind_port)
    await site.start()
    while True:
        # https://github.com/aio-libs/aiohttp/blob/master/aiohttp/web.py#L347-L348 :)
        await asyncio.sleep(3600)
    await runner.cleanup()
```

See https://docs.aiohttp.org/en/stable/web_reference.html#running-applications

Example: https://github.com/messa/h2tg/blob/master/h2tg/main.py


### How to make a redirect

```python
async def handle(request):
    raise web.HTTPFound('/destination')

# Signature: HTTPFound(location, *, headers=None, reason=None,
#                      body=None, text=None, content_type=None)
```

Documentation: https://aiohttp.readthedocs.io/en/stable/web_quickstart.html#redirects


### How to return HTML response

```python
async def handle(request):
    return web.Response(text=html_page, content_type='text/html')
    
# Signature: Response(*, body=None, status=200, reason=None, text=None,
#                     headers=None, content_type=None, charset=None)
```


### How to return error response

```python
async def handle(request):
    raise web.HTTPInternalServerError(text='Something failed')

# Signature: HTTPInternalServerError(*, headers=None, reason=None,
#                                    body=None, text=None, content_type=None)
```

HTTP Exception hierarchy chart: https://aiohttp.readthedocs.io/en/stable/web_quickstart.html#exceptions


### How to return JSON response

```python
async def handle(request):
    data = {'some': 'data'}
    return web.json_response(data)

# Signature: json_response([data, ]*, text=None, body=None, status=200, reason=None,
#                          headers=None, content_type='application/json', dumps=json.dumps)
```


Routing
-------

Aiohttp offers multiple options how to match requests with handler code:


### Handler functions

```python
async def handle(request):
    return web.json_response({'foo': 'bar'})

app.add_routes([web.get('/', handle),
                web.get('/{name}', handle)])
                
# You can also specify a custom regex in the form {identifier:regex}:
app.add_routes([web.get(r'/{name:\d+}', handle)])
```


### RouteTableDef

```python
routes = web.RouteTableDef()

@routes.get('/get')
async def handle_get(request):
    ...

@routes.post('/post')
async def handle_post(request):
    ...

@routes.view("/view")
class MyView(web.View):
    async def get(self):
        ...

    async def post(self):
        ...

app.router.add_routes(routes)
```


### Views

```python
class MyView(View):

    async def get(self):
        resp = await get_response(self.request)
        return resp

    async def post(self):
        resp = await post_response(self.request)
        return resp

app.router.add_view('/view', MyView)
```


App data
--------

Documentation: https://aiohttp.readthedocs.io/en/stable/web_advanced.html#data-sharing-aka-no-singletons-please

> aiohttp.web discourages the use of global variables, aka singletons. Every variable should have its own context that is not global.

> So, **`Application`** and **`Request`** support a `collections.abc.MutableMapping` interface (i.e. **they are dict-like objects**), allowing them to be used as **data stores**.

```python
app['my_private_key'] = data

async def handler(request):
    data = request.app['my_private_key']
```


Sessions
--------

https://aiohttp.readthedocs.io/en/stable/web_quickstart.html#user-sessions


Forms
-----

https://aiohttp.readthedocs.io/en/stable/web_quickstart.html#http-forms


### Accessing POST data

```python
async def do_login(request):
    data = await request.post()
    login = data['login']
    password = data['password']
```


### Accessing POST JSON data

If the POST data were sent as JSON instead of the "classic" form data (x-www-form-urlencoded):

```python
async def do_login(request):
    data = await request.json()
```


### File uploads

```python
async def store_mp3_handler(request):
    reader = await request.multipart()
    # /!\ Don't forget to validate your inputs /!\
    # reader.next() will `yield` the fields of your form
    field = await reader.next()
    assert field.name == 'name'
    name = await field.read(decode=True)
    field = await reader.next()
    assert field.name == 'mp3'
    filename = field.filename
    # You cannot rely on Content-Length if transfer is chunked.
    size = 0
    with open(os.path.join('/spool/yarrr-media/mp3/', filename), 'wb') as f:
        while True:
            chunk = await field.read_chunk()  # 8192 bytes by default.
            if not chunk:
                break
            size += len(chunk)
            f.write(chunk)
    return web.Response(text='{} sized of {} successfully stored'.format(filename, size))
```


How to make a proxy
-------------------

How to resend request to another server: [frontend_proxy.py](https://github.com/messa/pyladies-courseware/blob/0cfba019f7f277c8bfe8adc850a88d3b1968b648/backend/cw_backend/views/frontend_proxy.py)


Logging
-------

See [aiohttp-request-id-logging](https://github.com/messa/aiohttp-request-id-logging)


Links, tutorials, blogposts
---------------------------

https://steelkiwi.com/blog/an-example-of-a-simple-chat-written-in-aiohttp/

https://github.com/timofurrer/awesome-asyncio
