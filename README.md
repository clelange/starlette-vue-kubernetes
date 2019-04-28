# starlette-vue-kubernetes

Run Starlette with VueJS frontend on kubernetes

## Introduction

The idea is to build something like [flask-vue-kubernetes](https://testdriven.io/blog/running-flask-on-kubernetes/) to learn how to use async requests with [starlette](https://www.starlette.io/) to which later could be a backend (e.g. some external API) and display the results using [VueJS](https://vuejs.org/). At a later stage, some task queue could be added for even longer running tasks.

## Getting started with Starlette

In a new directory, in my case `starlette-vue-kubernetes`, create a base directory for Starlette and set up a new virtual environment. Mind, I'm using [pyenv](https://github.com/pyenv/pyenv with [pyenv-virtualenvwrapper](https://github.com/pyenv/pyenv-virtualenvwrapper)) here with Python 3.7.3 installed via [Homebrew](https://brew.sh/) on a Mac. You can as well just create a new virtual environment.

```shell
mkdir -p services/server
cd services/server
source /usr/local/bin/virtualenvwrapper.sh
pyenv shell 3.7.3
mkproject -a $PWD -p python3.7 starlette-server
```

Whenever you want to work on this project, you need to type `workon starlette-server`.
Now, let's install Starlette and uvicorn:

```shell
pip install starlette
pip install uvicorn
```

Then let's run the example as on the [Starlette homepage](https://www.starlette.io/). Put the following into a file called `example.py`:

```python
from starlette.applications import Starlette
from starlette.responses import JSONResponse
import uvicorn

app = Starlette(debug=True)

@app.route('/')
async def homepage(request):
    return JSONResponse({'hello': 'world'})

if __name__ == '__main__':
    uvicorn.run(app, host='0.0.0.0', port=8000)
```

Then let's run it:

```shell
python example.py
```

Now you can point your browser to [http://0.0.0.0:8000](http://0.0.0.0:8000) to see the JSON response.

This is all we need for now. Later, we might add some `time.sleep(10)` statements to mimic very slow backends. However, let's now first get to VueJS to consume the `{'hello': 'world'}` response.

To save the current state of the libraries we installed, let's freeze:

```shell
pip freeze > requirements.txt
```

All this should be added to version control.