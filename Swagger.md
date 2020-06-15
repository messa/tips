Swagger / OpenAPI Specification
===============================

[swagger.io/specification/](https://swagger.io/specification/) or
[swagger.io/docs/specification/](https://swagger.io/docs/specification/)


Examples
--------

https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v3.0

```yaml
openapi: "3.0.0"
info:
  version: 1.0.0
  title: Swagger Petstore
  license:
    name: MIT
servers:
  - url: http://petstore.swagger.io/v1
paths:
  /pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      tags:
        - pets
      parameters:
        - name: petId
          in: path
          required: true
          description: The id of the pet to retrieve
          schema:
            type: string
      responses:
        '200':
          description: Expected response to a valid request
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pet"
        default:
          description: unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
components:
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
    Pets:
      type: array
      items:
        $ref: "#/components/schemas/Pet"
    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```


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


