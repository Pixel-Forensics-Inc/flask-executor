Flask-Executor
==============

[![Build Status](https://travis-ci.org/dchevell/flask-executor.svg?branch=master)](https://travis-ci.org/dchevell/flask-executor)
[![Coverage Status](https://coveralls.io/repos/github/dchevell/flask-executor/badge.svg)](https://coveralls.io/github/dchevell/flask-executor)
[![PyPI Version](https://img.shields.io/pypi/v/Flask-Executor.svg)](https://pypi.python.org/pypi/Flask-Executor)
[![GitHub license](https://img.shields.io/github/license/dchevell/flask-executor.svg)](https://github.com/dchevell/flask-executor/blob/master/LICENSE)

Sometimes you need a simple task queue without the overhead of separate worker processes or powerful-but-complex libraries beyond your requirements. Flask-Executor is an easy to use wrapper for the `concurrent.futures` module that lets you initialise and configure executors via common Flask application patterns. It's a great way to get up and running fast with a lightweight in-process task queue.

Installation
------------

Flask-Executor is available on PyPI and can be installed with:

    pip install flask-executor


Quick start
-----------

Here's a quick example of using Flask-Executor inside your Flask application:

```python
from flask import Flask
from flask_executor import Executor

app = Flask(__name__)

executor = Executor(app)


def send_email(recipient, subject, body):
    # Magic to send an email
    return True


@app.route('/signup')
def signup():
    # Do signup form
    executor.submit(send_email, recipient, subject, body)
```


Contexts
--------

When calling `submit()` or `map()` Flask-Executor will wrap `ThreadPoolExecutor` callables with a
copy of both the current application context and current request context. Code that must be run in
these contexts or that depends on information or configuration stored in `flask.current_app`,
`flask.request` or `flask.g` can be submitted to the executor without modification.


Futures
-------

You may want to preserve access to Futures returned from the executor, so that you can retrieve the
results in a different part of your application. Flask-Executor allows Futures to be stored within
the executor itself and provides methods for querying and returning them in different parts of your
app::

```python
@app.route('/start-task')
def start_task():
    executor.submit_stored('calc_power', pow, 323, 1235)
    return jsonify({'result':'success'})

@app.route('/get-result')
def get_result():
    if not executor.futures.done('calc_power'):
        return jsonify({'status': executor.futures._state('calc_power')})
    future = executor.futures.pop('calc_power')
    return jsonify({'status': done, 'result': future.result()})
```


Decoration
----------

Flask-Executor lets you decorate methods in the same style as distributed task queues like
Celery:

```python
@executor.job
def fib(n):
    if n <= 2:
        return 1
    else:
        return fib(n-1) + fib(n-2)

@app.route('/decorate_fib')
def decorate_fib():
    fib.submit(5)
    fib.submit_stored('fibonacci', 5)
    fib.map(range(1, 6))
    return 'OK'
```


Documentation
-------------

Check out the full documentation at [flask-executor.readthedocs.io](https://flask-executor.readthedocs.io)!
