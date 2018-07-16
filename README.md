# Netlify + Intercom Oauth &nbsp;&nbsp;&nbsp;<a href="https://app.netlify.com/start/deploy?repository=https://github.com/davidwells/intercom-netlify-oauth"><img src="https://www.netlify.com/img/deploy/button.svg"></a>

Add 'login with intercom' via Netlify Functions & Oauth!

<!-- AUTO-GENERATED-CONTENT:START (TOC) -->
- [About](#about)
- [How to Install and Setup](#how-to-install-and-setup)
- [Running Locally](#running-locally)
- [Deploying](#deploying)
- [How it works](#how-it-works)
  * [auth.js function](#authjs-function)
  * [auth-callback.js function](#auth-callbackjs-function)
<!-- AUTO-GENERATED-CONTENT:END -->

## About

This project sets up a "login with intercom" Oauth flow using netlify functions.

Here is a quick demo of the login flow, and the Oauth Access data you get back:

![intercom oauth demo](https://user-images.githubusercontent.com/532272/42738995-7a8de2a0-8843-11e8-8179-d1865ded82ab.gif)

You can leverage this project to wire up intercom login with your application.

## How to Install and Setup

1. **Clone down the repository**

    ```bash
    git clone git@github.com:DavidWells/intercom-netlify-oauth.git
    ```

2. **Install the dependencies**

    ```bash
    npm install
    ```

3. **Create an Intercom Oauth app**

    Lets go ahead and setup the intercom app we will need!

    [Create an intercom Oauth app here](https://app.intercom.com/developers/)

    You need to enable a 'test' app in your account. It's a tricky to find but you can create a TEST app in your intercom account under `Settings > General`

    `https://app.intercom.com/a/apps/your-app-id/settings/general`

    ![intercom-test-app-setup](https://user-images.githubusercontent.com/532272/42739711-0ec30506-8851-11e8-8c0a-b4b1d5bd4174.jpg)

    After enabling the test app, you can find it listed in your [intercom developer portal](https://app.intercom.com/developers/).

    We now need to configure the test app.

    Input the live "WEBSITE URL" and "REDIRECT URLS" in the app edit screen.

    ![itercom-oauth-app-settings](https://user-images.githubusercontent.com/532272/42740025-0ea5833c-8856-11e8-827a-369189b951a1.jpg)

    You will want to have your live Netlify site URL and `localhost:3000` setup to handle the redirects for local development.

    If you haven't deployed to Netlify yet, just insert a placeholder URL like `http://my-temp-site.com` but **remember to change this once your Netlify site is live with the correct URL**

    Our demo app has these `REDIRECT URLS` values that are comma separated

    ```bash
    https://intercom-login-example.netlify.com/.netlify/functions/auth-callback,
    http://localhost:3000/.netlify/functions/auth-callback
    ```

    Great we are all configured over here.

4. **Grab your the required config values**

    We need our intercom app values to configure our function environment variables.

    Navigate back to the main Oauth screen and grab the **App ID**, **Client ID**, and **Client Secret** values. We will need these to run the app locally and when deploying to Netlify.

    ![intercom-config-values](https://user-images.githubusercontent.com/532272/42739965-25d15c26-8855-11e8-925b-105c1fa381f5.jpg)

## Running Locally

Set your Intercom app id and Oauth values in your terminal environment

After creating your [Oauth app](https://app.intercom.com/developers/) and the steps above, plugin the required environment variables into your local terminal session like so:

On linux/MacOS:

```bash
export INTERCOM_APP_ID=INTERCOM_APP_ID
export INTERCOM_CLIENT_ID=INTERCOM_CLIENT_ID
export INTERCOM_CLIENT_SECRET=INTERCOM_CLIENT_SECRET
```

or in windows:

```bash
set INTERCOM_APP_ID=INTERCOM_APP_ID
set INTERCOM_CLIENT_ID=INTERCOM_CLIENT_ID
set INTERCOM_CLIENT_SECRET=INTERCOM_CLIENT_SECRET
```

Then run the start command

```bash
npm start
```

This will boot up our functions to run locally for development. You can now login via your intercom application and see the token data returned.

Making edits to the functions in the `/functions` will hot reload and you can build your app.

## Deploying

Use the one click deploy button to launch this!

[![Deploy to Netlify](https://www.netlify.com/img/deploy/button.svg)](https://app.netlify.com/start/deploy?repository=https://github.com/davidwells/intercom-oauth)

OR connect this repo with your Netlify account and add in your values.

In `https://app.netlify.com/sites/YOUR-SITE-SLUG/settings/deploys` add the  `INTERCOM_APP_ID`, `INTERCOM_CLIENT_ID`, and `INTERCOM_CLIENT_SECRET` values to the "Build environment variables" section of settings

![intercom-deploy-settings](https://user-images.githubusercontent.com/532272/42740147-ece388c8-8857-11e8-93af-a1dd721e345a.jpg)

## How it works

Once again, serverless functions come to the rescue!

We will be using 2 functions to handle the entire Oauth flow with intercom.

Here is a diagram of what is happening:

![intercom oauth netlify](https://user-images.githubusercontent.com/532272/42144429-d2717f24-7d6f-11e8-8619-c1bec1562991.png)

1. First the `auth.js` function is triggered & redirects the user to intercom
2. The user logs in via intercom and is redirected back to `auth-callback.js` function with an **auth grant code**
3. `auth-callback.js` takes the **auth grant code** and calls back into intercom's API to exchange it for an **AccessToken**
4. `auth-callback.js` now has the **AccessToken** to make any API calls it would like back into the intercom App.

This flow uses the [Authorization Code Grant](https://tools.ietf.org/html/draft-ietf-oauth-v2-31#section-4.1) flow. For more information on Oauth 2.0, [Watch this video](https://www.youtube.com/watch?v=CPbvxxslDTU)

Let's dive into the individual functions:

### auth.js function

The `auth.js` function creates an `authorizationURI` using the [`simple-oauth2` npm module](https://www.npmjs.com/package/simple-oauth2) and redirects the user to the intercom login screen.

Inside of the `auth.js` function, we set the `header.Location` in the lambda response and that will redirect the user to the `authorizationURI`, a.k.a the intercom oauth login screen.

<!-- AUTO-GENERATED-CONTENT:START (CODE:src=./functions/auth.js&header=/* code from /functions/auth.js */) -->
<!-- The below code snippet is automatically added from ./functions/auth.js -->
```js
/* code from /functions/auth.js */
import oauth2, { config } from './utils/oauth'

/* Do initial auth redirect */
exports.handler = (event, context, callback) => {
  /* Generate authorizationURI */
  const authorizationURI = oauth2.authorizationCode.authorizeURL({
    redirect_uri: config.redirect_uri,
    /* Specify how your app needs to access the user’s account. http://bit.ly/intercom-scopes */
    scope: '',
    /* State helps mitigate CSRF attacks & Restore the previous state of your app */
    state: '',
  })

  /* Redirect user to authorizationURI */
  const response = {
    statusCode: 302,
    headers: {
      Location: authorizationURI,
      'Cache-Control': 'no-cache' // Disable caching of this response
    },
    body: '' // return body for local dev
  }

  return callback(null, response)
}
```
<!-- AUTO-GENERATED-CONTENT:END -->

### auth-callback.js function

The `auth-callback.js` function handles the authorization grant code returned from the successful intercom login.

It then calls `oauth2.authorizationCode.getToken` to get a valid AccessToken from intercom.

Once you have the valid accessToken, you can store it and make authenticated calls on behalf of the user to the intercom API.

<!-- AUTO-GENERATED-CONTENT:START (CODE:src=./functions/auth-callback.js&header=/* code from /functions/auth-callback.js */) -->
<!-- The below code snippet is automatically added from ./functions/auth-callback.js -->
```js
/* code from /functions/auth-callback.js */
import getUserData from './utils/getUserData'
import oauth2, { config } from './utils/oauth'

/* Function to handle intercom auth callback */
exports.handler = (event, context, callback) => {
  const code = event.queryStringParameters.code
  /* state helps mitigate CSRF attacks & Restore the previous state of your app */
  const state = event.queryStringParameters.state

  /* Take the grant code and exchange for an accessToken */
  oauth2.authorizationCode.getToken({
    code: code,
    redirect_uri: config.redirect_uri,
    client_id: config.clientId,
    client_secret: config.clientSecret
  })
    .then((result) => {
      const token = oauth2.accessToken.create(result)
      console.log('accessToken', token)
      return token
    })
    // Get more info about intercom user
    .then(getUserData)
    // Do stuff with user data & token
    .then((result) => {
      console.log('auth token', result.token)
      // Do stuff with user data
      console.log('user data', result.data)
      // Do other custom stuff
      console.log('state', state)
      // return results to browser
      return callback(null, {
        statusCode: 200,
        body: JSON.stringify(result)
      })
    })
    .catch((error) => {
      console.log('Access Token Error', error.message)
      console.log(error)
      return callback(null, {
        statusCode: error.statusCode || 500,
        body: JSON.stringify({
          error: error.message,
        })
      })
    })
}
```
<!-- AUTO-GENERATED-CONTENT:END -->

So as we can see, using two pretty simple lambda functions we can now handle logins via Intercom or any other third party Oauth provider.
