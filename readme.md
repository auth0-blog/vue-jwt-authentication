# Vue.js JWT Authentication

This example shows how to do JWT authentication in Vue.js apps. It uses Auth0's [nodejs-jwt-authentication-sample](https://github.com/auth0/nodejs-jwt-authentication-sample), a NodeJS backend that serves Chuck Norris quotes.

## Installation

Clone the repo and then install the server submodule and dependencies.

```bash
git submodule update --init
cd server
npm install
cd ..
npm install
```

Once the application scripts are in place, start the server (which will provide the quotes) using:

```bash
node server/server.js
```

Afterwards, open a second Terminal window and run the [webpack development server](http://webpack.github.io/docs/webpack-dev-server.html). It will watch for changes with **hot reloading**:

```bash
npm run dev
```

## Important Snippets

The entry point for the app is at `src/index.js`. Here we import everything we need and set up routing.

```js
// src/index.js

import Vue from 'vue'
import App from './components/App.vue'
import Home from './components/Home.vue'
import SecretQuote from './components/SecretQuote.vue'
import Signup from './components/Signup.vue'
import Login from './components/Login.vue'
import VueRouter from 'vue-router'
import VueResource from 'vue-resource'
Vue.use(VueResource)
Vue.use(VueRouter)
import auth from './auth'

Vue.http.headers.common['Authorization'] = 'Bearer ' + localStorage.getItem('id_token');

// Check the user's auth status when the app starts
auth.checkAuth()

export var router = new VueRouter()

router.map({
  '/home': {
    component: Home
  },
  'secretquote': {
    component: SecretQuote
  },
  '/login': {
    component: Login
  },
  '/signup': {
    component: Signup
  }
})

router.redirect({
  '*': '/home'
})

router.start(App, '#app')
```

The `Login` and `Signup` Vue components are very similar and allow the user to enter their username and password.

```html
  <!-- src/components/Login.vue -->

  <template>
    <div class="col-sm-4 col-sm-offset-4">
      <h2>Log In</h2>
      <p>Log in to your account to get some great quotes.</p>
      <div class="alert alert-danger" v-if="error">
        <p>{{ error }}</p>
      </div>
      <div class="form-group">
        <input 
          type="text" 
          class="form-control"
          placeholder="Enter your username"
          v-model="credentials.username"
        >
      </div>
      <div class="form-group">
        <input
          type="password"
          class="form-control"
          placeholder="Enter your password"
          v-model="credentials.password"
        >
      </div>
      <button class="btn btn-primary" @click="submit()">Access</button>
    </div>
  </template>

  <script>
  import auth from '../auth'

  export default {

    data() {
      return {
        credentials: {
          username: '',
          password: ''
        },
        error: ''
      }
    },

    methods: {

      submit() {

        var credentials = {
          username: this.credentials.username,
          password: this.credentials.password
        }

        auth.login(this, credentials, 'secretquote')

      }
    }
    
  }
  </script>
```

These components utilize the `auth` service.

```js
//  src/auth/index.js

import {router} from '../index'

const API_URL = 'http://localhost:3001/'
const LOGIN_URL = API_URL + 'sessions/create/'
const SIGNUP_URL = API_URL + 'users/'

export default {

  user: {
    authenticated: false
  },

  login(context, creds, redirect) {
    context.$http.post(LOGIN_URL, creds, (data) => {
      localStorage.setItem('id_token', data.id_token)

      this.user.authenticated = true

      if(redirect) {
        router.go(redirect)        
      }

    }).error((err) => {
      context.error = err
    })
  },

  signup(context, creds, redirect) {
    context.$http.post(SIGNUP_URL, creds, (data) => {
      localStorage.setItem('id_token', data.id_token)

      this.user.authenticated = true

      if(redirect) {
        router.go(redirect)        
      }

    }).error((err) => {
      context.error = err
    })
  },

  logout() {
    localStorage.removeItem('id_token')
    this.user.authenticated = false
  },

  checkAuth() {
    var jwt = localStorage.getItem('id_token')
    if(jwt) {
      this.user.authenticated = true
    }
    else {
      this.user.authenticated = false      
    }
  },


  getAuthHeader() {
    return {
      'Authorization': 'Bearer ' + localStorage.getItem('id_token')
    }
  }
}
```

Finally, we can get protected Chuck Norris quotes with our `SecretQuote` component. This component is very similar to `Home` in which we are able to retrieve unprotected quotes. The difference is that `SecretQuote` attaches an `Authorization` header on the `GET` requests that are made from it.

```html
  <!-- src/components/SecretQuote.vue --> 
  
  <template>
    <div class="col-sm-6 col-sm-offset-3">
      <h1>Get a Secret Chuck Norris Quote!</h1>
      <button class="btn btn-warning" v-on:click="getQuote()">Get a Quote</button>
      <div class="quote-area" v-if="quote">
        <h2><blockquote>{{ quote }}</blockquote></h2>      
      </div>
    </div>
  </template>

  <script>
  import auth from '../auth'

  export default {

    data() {
      return {
        quote: ''
      }
    },

    methods: {
      getQuote() {
        this.$http
          .get('http://localhost:3001/api/protected/random-quote', (data) => {
            this.quote = data;
          }, { 
            headers: auth.getAuthHeader()
          })
          .error((err) => console.log(err))
      }
    },

    route: {
      canActivate() {
        return auth.user.authenticated
      }
    }

  }
  </script>
```

## What is Auth0?

Auth0 helps you to:

* Add authentication with [multiple authentication sources](https://docs.auth0.com/identityproviders), either social like **Google, Facebook, Microsoft Account, LinkedIn, GitHub, Twitter, Box, Salesforce, amont others**, or enterprise identity systems like **Windows Azure AD, Google Apps, Active Directory, ADFS or any SAML Identity Provider**.
* Add authentication through more traditional **[username/password databases](https://docs.auth0.com/mysql-connection-tutorial)**.
* Add support for **[linking different user accounts](https://docs.auth0.com/link-accounts)** with the same user.
* Support for generating signed [Json Web Tokens](https://docs.auth0.com/jwt) to call your APIs and **flow the user identity** securely.
* Analytics of how, when and where users are logging in.
* Pull data from other sources and add it to the user profile, through [JavaScript rules](https://docs.auth0.com/rules).

## Create a Free Auth0 Account

1. Go to [Auth0](https://auth0.com) and click Sign Up.
2. Use Google, GitHub or Microsoft Account to login.

## Issue Reporting

If you have found a bug or if you have a feature request, please report them at this repository issues section. Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/whitehat) details the procedure for disclosing security issues.

## Author

[Auth0](auth0.com)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
