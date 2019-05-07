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

## Communicating from frontend to backend

The next step is to enable the two VueJS frontend to receive information from/query the Starlette backend.

Strip down `<template>` section of `client/src/components/HelloWorld.vue` to only look like this:

```html
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>
```

Also, have a look at the remainder of the file. You will see that the message on the webpage when running `npm run dev` and pointing your browser to [http://localhost:8080](http://localhost:8080) is the message (`msg`) defined in the `<script>` section. In order to be able to communicate with the backend, we will now use the [axios](https://github.com/axios/axios) library:

```shell
npm install axios
```

Now we can attempt to make a call to our backend by extending the `<script>` section of `HelloWorld.vue` as follows:

```javascript
<script>
import axios from 'axios';

export default {
  name: 'HelloWorld',
  data() {
    return {
      msg: 'Welcome to Your Vue.js App',
    };
  },
  methods: {
    getMessage() {
      const path = 'http://localhost:8000/';
      axios.get(path)
        .then((res) => {
          this.msg = `Hello, ${res.data.hello}!`;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
  },
  created() {
    this.getMessage();
  },
};
</script>
```

Start the Starlette server in a new terminal window via `python example.py` and refresh your frontend webpage. Unfortunately, nothing will happen. To figure out why this is so, open the Developer Console of your browser. You will see a message like this:

`Access to XMLHttpRequest at 'http://localhost:8000/' from origin 'http://localhost:8080' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

The CORS, which means Cross-Origin Resource Sharing, policy of the backend webserver does not allow requests from a different host for security reasons. This can be changed easily by following the instructions listed in the [CORSMiddleware](https://www.starlette.io/middleware/#corsmiddleware) documentation. Under [this link](https://medium.com/@xinganwang/a-practical-guide-to-cors-51e8fd329a1f) you can find more general information on CORS with some more examples.

Adjust the `example.py` file by adding a line at the top:

```python
from starlette.middleware.cors import CORSMiddleware
```

and below the line creating the `App`, add the following:

```python
# A list of origins that should be permitted to make cross-origin requests
app.add_middleware(CORSMiddleware,
    allow_origins=['http://localhost:8080'],
)
```

Since we know exactly where the request is coming from, we can explicitly list it in the configuration. Restart the webserver and you will see that the "Welcome to Your Vue.js App" message changes to "Hello, world!". We have established communication between frontend and backend!