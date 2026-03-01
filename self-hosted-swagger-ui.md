Self Hosted SwaggerUI
=====================

When using FastAPI, it is common to use SwaggerUI to see the docs and test the API
endpoints. By default, FastAPI uses a CDN(like jsdelivr) to serve the needed CSS and JS
files for the UI. This can cause issues if you want to use the docs in an environment
in which you don't have access to jsdelivr.

It seems like a good idea to try to self-host these files so that we don't have to rely
on a CDN and be able to access the docs offline; so let's do it!

Code Changes
------------

To do this, we first need to download the CSS and JS files and serve them statically.
You can download the latest version of [SwaggerUI from GitHub](https://github.com/swagger-api/swagger-ui/releases).

We'll only need 3 files from the `dist/` directory: `swagger-ui-bundle.js`,
`swagger-ui-standalone-preset.js`, and `swagger-ui.css`

After downloading them we will have to serve the files. Your directory structure could
be something like this:

```
project/
├── main.py
└── static/
    └── swagger/
        ├── swagger-ui.css
        ├── swagger-ui-bundle.js
        └── swagger-ui-standalone-preset.js
```

Then we can serve the static files like so:

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

This will serve files under `static/` at `/static/` path.

---

Now that we took care of serving static files, we should continue by serving our own
custom docs page.

To do that, we will first have to disable FastAPI's default docs serve URL:

```python
app = FastAPI(docs_url=None, ...)
```

Then we can serve our own docs at `/docs`:

```python
from fastapi.openapi.docs import get_swagger_ui_html
from fastapi.responses import HTMLResponse


@app.get("/docs", include_in_schema=False)
async def custom_swagger_ui_html() -> HTMLResponse:
    return get_swagger_ui_html(
        openapi_url=app.openapi_url,
        title="API Docs",
        swagger_js_url="/static/swagger/swagger-ui-bundle.js",
        swagger_css_url="/static/swagger/swagger-ui.css",
        swagger_favicon_url="/static/swagger/favicon.ico",  # Optional favicon path
    )
```

And, done!

Now we have a SwaggerUI docs page which can be accessed even with limited to no internet
access.

Full code example
-----------------

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.openapi.docs import get_swagger_ui_html

app = FastAPI(docs_url=None)
app.mount("/static", StaticFiles(directory="static"), name="static")


@app.get("/docs", include_in_schema=False)
async def custom_swagger_ui_html() -> HTMLResponse:
    return get_swagger_ui_html(
        openapi_url=app.openapi_url,
        title="API Docs",
        swagger_js_url="/static/swagger/swagger-ui-bundle.js",
        swagger_css_url="/static/swagger/swagger-ui.css",
        # swagger_favicon_url="/static/swagger/favicon.ico",
    )


@app.get("/ping")
def pong():
    return {"ping": "pong!"}
```
