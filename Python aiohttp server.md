aiohttp web server cheatsheet
=========================

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


### How to make a redirect

```python
async def handle(request):
    raise web.HTTPFound('/destination')
```

Routing
-------

Aiohttp offers multiple options how to match requests with handler code:

### Handler functions

```python
async def handle(request):
    data = {'some': 'data'}
    return web.json_response(data)    

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
