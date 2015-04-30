---
layout: post
title: Flask development
author: Ben Fairless
---

## Introduction

Built on top of the excellent [12 Factor App manifesto](http://12factor.net/) by Adam Wiggins we have defined some core principles for applications to follow to ensure that they have a clean relationship with the underlying system and can be easily ported to other platforms, both on-premise and in-cloud, and that can scale effectively to deal with increased usage without significant application changes.

There is also a sample Flask application on [GitHub](https://github.com/) which you can use as a reference.

## WSGI end point

In order for our web servers to bring up web applications they need to know where the [**WSGI end point**](https://www.python.org/dev/peps/pep-0333/#the-application-framework-side) sits inside an application. In Flask, this is the main Application object usually created like this:

```python
from flask import Flask
app = Flask(__name__)
```

This should sit directly inside your main module (`application`) in the `__init__.py` file. This allows the application to be initiated by calling the `application:app` object.


## Logging

Logs provide an important window into the activity of a running application. This is absolutely critical for debugging and providing real-time support and also serves a vital role in providing an audit trail.

In order make applications easily portable they should not attempt to manage the routing or storage of log data, and instead should simply log events to an output stream which can be captured by the web server the application runs on.

Within a Flask application this can be easily achieved with the following code snippet:

```python
# When running in debug mode, Flask will automatically route the output stream to the console
if not app.debug:
    import logging

    # Create log format - 'YYYY-MM-DD HH:MM:SS [LOG LEVEL] MESSAGE'
    formatter = logging.Formatter('%(asctime)s [%(levelname)s] %(message)s','%Y-%m-%d %H:%M:%S')

    # Create console handler
    stdout = logging.StreamHandler()
    stdout.setLevel(logging.INFO)
    stdout.setFormatter(formatter)

    # Enable log handler
    app.logger.addHandler(stdout)
```

This will create a new log handler for applications logs and send all events in the output stream to `stdout` in the format `2015-05-29 18:21:06 [INFO] Sample log entry`. This allows us to capture the `stdout` stream and manage logs in a standardised manner.

## Health checks

Health checks are an important feature of building and running distributed applications. They provide insight into the status of the application and allow us to quickly react to incidents, removing unhealthy nodes from clusters. Health checks also allow us to identify if automatic deployments have not been successful.

All web applications should present a `/health` route which will return a relevant status code as well as a JSON object structured as follows:

```json
{
  "commit": "current commit in version control",
  "status": "short explanation of issue, should return 'Healthy' if good"
}
```

An example HTTP response would look something like this:

```bash
$ curl -i http://localhost:5000/health
    HTTP/1.1 500 INTERNAL SERVER ERROR
    Server: gunicorn/18.0
    Date: Wed, 29 Apr 2015 20:56:50 GMT
    Connection: close
    Content-Type: application/json
    Content-Length: 82

    {
      "commit": "0dce6fd0f7f9430567faf26095a1c10189fcec01",
      "status": "Could not connect to database"
    }
```
As you can see, this application is having difficulty connecting to it's datastore and has therefore returned a HTTP status code of `500 INTERNAL SERVER ERROR`.
