---
title: Quick Introduction to JWT using Javascript
date: "2019-06-20"
description: "JWT is being heavily used to authenticate the SPAs in the APIs backends, and in this post we will get some introductions to this technique."
---

JWT(JSON Web Token) is being heavily used to authenticate SPAs(Single Page Applications) in backends(server applications), but before we get started, we should know what a JWT is. I've put a short summary below, but if you need more information, visit the [official JWT website](https://jwt.io/).

> JSON Web Token (JWT) is an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret or a public/private key...

 The process of authentication and validation is very simple, let's see how it happens:

  1. The SPA sends a login request with the user credentials
  2. The backend verifies that the credentials are valid
  3. If the credentials are valid, the backend will generate a new token using a *payload* containing some user identifier(such as id, or email) and a *secret*.
  4. The backend sends the token to the SPA.
  5. The SPA stores the token.
  6. The SPA uses the token as an *authorization header* in an HTTP request.
  7. The backend checks the token.
  8. If the token is valid, the backend returns the requested data.

Now that we are understanding the process we can turn it into code.

## Code sample

*Backend - Logging in and generating the token*

```javascript
const jwt = require('jsonwebtoken');

const signIn = async (req, res) => {
  const { email, password } = req.params

  try {
    const user = await doLogin(email, password);
    const token = jwt.sign({ userId: user.id }, 'your-secret'); // highlight-line

    return res.json({
      token
    })
  } catch(error) {
    ...
  }
}
```

*SPA - Sending the sign in request and getting the token*

```javascript
import axios from 'axios'
import Cookies from 'js-cookie'

const signIn = async (email, password) => {
  try {
    const credentials = await axios.post('/signIn', { email, password })
    Cookies.set('credentials', credentials)// highlight-line
  } catch(error) {
    ...
  }
}
```

*SPA - Sending a request to get some private data*

```javascript
import axios from 'axios'
import Cookies from 'js-cookie'

const getBooks = (email, password) => {
  try {
    const credentials = Cookies.getJSON('credentials') // highlight-line

    if(!credentials) {...}

    return axios.post('/books', null, {
      headers: {
        authorization: credentials.token // highlight-line
      }
    })
  } catch(error) {
    ...
  }
}
```

*Backend - Checking the request*

```javascript
const jwt = require('jsonwebtoken');

const books = async (req, res) => {
  try {
    const token = req.headers.authorization // highlight-line
    const { userId } = jwt.verify(token, 'your-secret'); // highlight-line

    return res.json(
      Books
        .where({
          user_id: userId
        })
        .all()
    )
  } catch(error) {
    ...
  }
}
```

## Security concerns
  - Use your jwt secret as an env variable like `process.env.JWT_SECRET`.
  - Use the expiration time on tokens and cookies. You do not want to have a leaked token living forever.
  - Do not put sensitive data inside the payload because the token can be decoded by anyone and expose the information.

*Example with 1 day expiration:*

```javascript
  // Backend
  jwt.sign(
    { userId: user.id },
    'your-secret',
    { expiresIn: '1d' } //highlight-line
  );

  // SPA
  Cookies.set(
    'credentials',
    credentials,
    { expires: 1 }//highlight-line
  )
```

I hope you have enjoyed this article and it has been useful to you. Thanks for reading!