---
title: How to Store Tokens Safely
date: "2019-10-29"
description: "Let's talk about how to store your tokens safely using cookies and lambdas."
---

How should tokens be stored safely? It's a common question within the frontend community, and there are a few alternatives. In this post, I'll show you how to do this using cookies and lambdas.

Before we get started, know that using local storage is not a good idea. Malicious scripts can iterate over local storage and send the info to an external API. So, if you have any kind of sensitive data there, you're screwed.

The easiest way of storing a token safely is by just keeping it in memory. This option has some pitfalls in UX. So what should we do?

Cookies are a good tool for this scenario. Of course, you should follow some guidelines to make it safe to do so, such as set the cookie as `secure`, `httpOnly`, and `SameSite=strict`. The `secure` flag ensures that the cookie is sent only over `https` connections, the `httpOnly` flag makes the cookie inaccessible on the client, so malicious scripts cannot see your sensitive data in `document.cookies`, and the `SameSite=strict` flag blocks cookies from being sent to an external domain.

Since the cookie is not available on the client, we need to create some backend logic to fetch the token and store it in memory. You can create a BFF(backend for frontends) using a web framework like Rails, Django, Phoenix, Laravel, etc., or you can use lambdas which are very easy and fast to move on. I recommend that you use Netlify Functions or Zeit. Let's see how it works.

### Login Flow

![Login Flow](./login-flow.png)

In this flow, we creating a token and storing it in a `httpOnly` cookie, to be protected by our backend, preventing it from being accessed by malicious front-end/client scripts. After that, the app can store the token in memory and use it to run your logic. 

*Notice that the function is acting as the cookie guardian where the token is stored.*

Here is a more detailed explanation:

1. A user accesses the login page.
2. When the user attempts to authenticate, the credentials are sent to the login function.
3. The login function validates the credentials and creates a token if the credentials are valid. If the credentials are not valid, the login function returns an "unauthorized" error.
4. For valid credentials, the login function creates an `httpOnly` cookie, which prevents access from malicious front-end/client scripts.
5. (6 and 7) The function returns the token and the app saves it in memory.


One problem: if the user refreshes the page, their browser loses the token and they become unauthenticated, which might crash my app. What we should do? We need to validate the user session through a validate function.

### Validation Flow

![Validate Flow](./validate-flow.png)

We need to check if the user is already logged in. The app sends a request to the validate function, which checks if there is any token stored in the cookie and if it is valid. If the token is valid, the validate function returns it to the app, which stores the token in memory. Here is a more detailed description:

1. A user accesses the app.
2. The app makes a request to the validate function.
3. (4 and 5) The function checks if an authorization cookie exists, that the cookie contains a token, and whether the token is valid. If none of these things are true, the function returns an "unauthorized" error.
6. If the token is valid, the function returns the token and the app saves it in memory. 

Now that we know how to improve your security by storing a token safely, we can implement it. In the next article, I'll show you how using JS, React and FaunaDB. I hope this article has been helpful to you. For further information, check out the following references:

Thanks for your time, see ya!

## References
- [https://www.rdegges.com/2018/please-stop-using-local-storage/](https://www.rdegges.com/2018/please-stop-using-local-storage/)
- [https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))
- [https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))
- [https://www.owasp.org/index.php/HttpOnly](https://www.owasp.org/index.php/HttpOnly)
- [https://auth0.com/docs/security/store-tokens](https://auth0.com/docs/security/store-tokens)
