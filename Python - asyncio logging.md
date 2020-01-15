Python asyncio logging
======================

Mandatory reading: 🙂

- asyncio docs: https://docs.python.org/3/library/asyncio.html
  - Czech intro: https://naucse.python.cz/lessons/intro/async/
- logging docs: https://docs.python.org/3/library/logging.html
  - cookbook: https://docs.python.org/3/howto/logging-cookbook.html


Trivial approach
----------------

```python
# 01_trivial.py

import asyncio
import logging

logger = logging.getLogger(__name__)

async def main():
    logging.basicConfig(level=logging.DEBUG)
    logger.info('Start')
    await asyncio.gather(process_request('foo'), process_request('bar'))
    logger.info('Done')

async def process_request(name):
    logger.info('Started processing request %s', name)
    await do_something()
    logger.info('Processing request done')

async def do_something():
    await asyncio.sleep(1)
    logger.info('Doing something')
    await asyncio.sleep(1)

asyncio.run(main())
```

Output:

```
INFO:__main__:Start
INFO:__main__:Started processing request foo
INFO:__main__:Started processing request bar
INFO:__main__:Doing something
INFO:__main__:Doing something
INFO:__main__:Processing request done
INFO:__main__:Processing request done
INFO:__main__:Done
```

❌ You cannot see what message belongs to what request


Explicit value passing to every funcion call
--------------------------------------------

```python
# 02_explicit.py

import asyncio
import logging

logger = logging.getLogger(__name__)

async def main():
    logging.basicConfig(level=logging.DEBUG)
    logger.info('Start')
    await asyncio.gather(process_request('foo'), process_request('bar'))
    logger.info('Done')

async def process_request(name):
    logger.info('[%s] Started processing request', name)
    await do_something(name)
    logger.info('[%s] Processing request done', name)

async def do_something(name):
    await asyncio.sleep(1)
    logger.info('[%s] Doing something', name)
    await asyncio.sleep(1)

asyncio.run(main())
```

Output:

```
INFO:__main__:Start
INFO:__main__:[foo] Started processing request
INFO:__main__:[bar] Started processing request
INFO:__main__:[foo] Doing something
INFO:__main__:[bar] Doing something
INFO:__main__:[foo] Processing request done
INFO:__main__:[bar] Processing request done
INFO:__main__:Done
```

✅ You can see what message belongs to what request<br>
❌ You have to pass the request name (or id or whatever) to function that does logging<br>
❌ You have to include the request name in every log message manually 


Use ContextVar instead of explicit passing
------------------------------------------

```python
# 03_contextvar.py

import asyncio
from contextvars import ContextVar
import logging

logger = logging.getLogger(__name__)

request_name = ContextVar('request_name')

async def main():
    logging.basicConfig(level=logging.DEBUG)
    logger.info('Start')
    await asyncio.gather(process_request('foo'), process_request('bar'))
    logger.info('Done')

async def process_request(name):
    request_name.set(name)
    logger.info('[%s] Started processing request', request_name.get())
    await do_something()
    logger.info('[%s] Processing request done', request_name.get())

async def do_something():
    await asyncio.sleep(1)
    logger.info('[%s] Doing something', request_name.get())
    await asyncio.sleep(1)

asyncio.run(main())
```

Output:

```
INFO:__main__:Start
INFO:__main__:[foo] Started processing request
INFO:__main__:[bar] Started processing request
INFO:__main__:[foo] Doing something
INFO:__main__:[bar] Doing something
INFO:__main__:[foo] Processing request done
INFO:__main__:[bar] Processing request done
INFO:__main__:Done
```

✅ You can see what message belongs to what request<br>
✅ You don't have to pass the request name to further function calls<br>
❌ You have to include the request name in every log message manually


Use logging filter to add to log message automatically
------------------------------------------------------

```python
# 04_logging_filter.py

import asyncio
from contextvars import ContextVar
import logging

logger = logging.getLogger(__name__)

request_name = ContextVar('request_name')

def add_request_name_filter(record):
    name = request_name.get(None)
    if name:
        record.msg = f'[{name}] {record.msg}'
    return True

async def main():
    logging.basicConfig(level=logging.DEBUG)
    logger.addFilter(add_request_name_filter)
    logger.info('Start')
    await asyncio.gather(process_request('foo'), process_request('bar'))
    logger.info('Done')

async def process_request(name):
    request_name.set(name)
    logger.info('Started processing request')
    await do_something()
    logger.info('Processing request done')

async def do_something():
    await asyncio.sleep(1)
    logger.info('Doing something')
    await asyncio.sleep(1)

asyncio.run(main())
```

Output:

```
INFO:__main__:Start
INFO:__main__:[foo] Started processing request
INFO:__main__:[bar] Started processing request
INFO:__main__:[foo] Doing something
INFO:__main__:[bar] Doing something
INFO:__main__:[foo] Processing request done
INFO:__main__:[bar] Processing request done
INFO:__main__:Done
```


✅ You can see what message belongs to what request<br>
✅ You don't have to pass the request name to further function calls<br>
✅ The request name is included in every log message automatically

🎉🎉🎉


Aiohttp
-------

See [aiohttp-request-id-logging](https://github.com/messa/aiohttp-request-id-logging)

