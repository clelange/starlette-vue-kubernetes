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


## Getting started with VueJS

We will roughly follow the instructions for [Developing a Single Page App with Flask and Vue.js](https://testdriven.io/blog/developing-a-single-page-app-with-flask-and-vuejs/) for this purpose. In case you do not yet have Node.js, you can [download it](https://nodejs.org/en/) or install it via [Homebrew](https://brew.sh/).
You might already have the vue-cli installed, so to make sure you use a recent version, update all globally installed packages by running `npm update -g`.

To install `vue-cli`, run the following as outlined in the corresponding [installation instructions](https://cli.vuejs.org/guide/installation.html):

```shell
npm install -g @vue/cli
npm install -g @vue/cli-init
```

Now change to the `services` directory in the project directory, e.g. as above `starlette-vue-kubernetes` and run the following command to initialise a new Vue project called `client` with the [webpack](https://github.com/vuejs-templates/webpack) config:

```shell
vue init webpack client
```

This will ask a couple of questions. Keep `client` as project name, feel free to change the description and author to your wanting. Then choose the following:

1. Vue build: ` Runtime + Compiler` (default)
1. Install vue-router?:  `Yes` (default)
1. Use ESLint to lint your code?:  `Yes` (default)
1. Pick an ESLint preset: ` Airbnb`
1. Set up unit tests:  `No`
1. Setup e2e tests with Nightwatch:  `No`
1. Should we run npm install for you after the project has been created: `Yes, use NPM` (default)

This will install a lot of things. To see if things worked, run:

```shell
cd client
npm run dev
```

The application should then be running at [http://localhost:8081](http://localhost:8081).

This concludes the installation of VueJS, now add the client directory to version control.
