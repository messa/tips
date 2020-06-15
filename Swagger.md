Swagger / OpenAPI Specification
===============================

[swagger.io/specification/](https://swagger.io/specification/) or
[swagger.io/docs/specification/](https://swagger.io/docs/specification/)


Rendering to HTML
-----------------

### Redoc

https://github.com/Redocly/redoc

I found Redoc through [FastAPI](https://fastapi.tiangolo.com/).

**Usage:** build a single self-contained HTML file

```shell
$ npx redoc-cli bundle my_cool_api.json
```

**Usage:** static HTML that loads JSON and renders it in the browser

```html
<script src="https://cdn.jsdelivr.net/npm/redoc/bundles/redoc.standalone.js"> </script>
<redoc spec-url="url/to/your/spec"></redoc>
```

See https://github.com/Redocly/redoc#deployment


