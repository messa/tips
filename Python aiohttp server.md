aiohttp web server cheatsheet
=========================

Homepage, documentation: https://aiohttp.readthedocs.io/en/stable/

Github: https://github.com/aio-libs/aiohttp

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

