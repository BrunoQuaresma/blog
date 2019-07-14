---
title: Code Splitting in React with Lazy Components
date: "2019-07-14"
description: "Let's learn how to increase our load speed using code splitting and lazy components."
---

Often during application development, we do not care much about the size of our bundle. However, as the code base grows, we need to think about it not only for speed and performance, but also to take responsibility for users' data consumption--especially on mobile devices. So how do we load our assets as quickly as possible, and only when the user needs them, by taking advantage of concurrent browser requests?

![Number of maximum of simultaneous requests](./requests.png)
**Font: [https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html](https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html)**

A good option for this is to use the **Code Split** technique. This technique consists of splitting the code into smaller chunks and loading these chunks only when they are required. Let's see a use case.

In [Fauna Dashboard](https://dashboard.fauna.com), we have pages where we are using code editors with heavy dependencies even after gzip. We do not want to put this in the main chunk, because this will make the first rendering longer and more costly than necessary on pages where the code editor is not needed. For this case, splitting code is a good option and I'll show you how to do it without any headaches. To get started, let's code something and run some analyses.

Let's use [create-react-app](https://github.com/facebook/create-react-app) to start a new React project and create the components:

```bash
create-react-app lazy-demo
cd lazy-demo
yarn add @reach/router react-ace
```

**App.jsx**
```jsx
import React from "react";
import { Router, Link } from "@reach/router";
import Home from "./Home";
import Code from "./Code";

const App = () => {
  return (
    <Router>
      <Home path="/" />
      <Code path="code" />
    </Router>
  );
};

export default App;
```

**Home.jsx**
```jsx
import React from "react";
import { Link } from "@reach/router";

const Home = () => (
  <>
    <h1>Home</h1>
    <Link to="/code">Go to code editor</Link>
  </>
);

export default Home;
```

**Code.jsx**
```jsx
import React from "react";
import CodeEditor from "./CodeEditor";

const Code = () => (
  <>
    <h1>Code</h1>
    <CodeEditor />
  </>
);

export default Code;
```

**CodeEditor.jsx**
```jsx
import React, { Lazy } from "react";
import AceEditor from "react-ace";

import "brace/mode/java";
import "brace/theme/github";

const CodeEditor = () => (
  <AceEditor mode="java" theme="github" onChange={() => {}} name="code" />
);

export default CodeEditor;
```

Now let's take a look at how our bundle is generated and how to optimize it using (source-map-explorer)[https://github.com/danvk/source-map-explorer#readme].

```bash
yarn add source-map-explorer
```

**package.json**
```json
 "scripts": {
    "analyze": "source-map-explorer 'build/static/js/*.js'", // highlight-line
```

```bash
yarn build
yarn analyze
```

After running the analyses, we can see a simple chart in the browser:

![Analyze before splitting](./analyze-1.png)

We can also look at the time it takes to load the page. For a better comparison, I'm loading this page with cache disabled and on a slow 3G connection.

![Time to load resources in browser](./load-browser.png)

As we can see in the image, all our code is in only one piece. It does not use the power of parallel requests and loads unnecessary code the first time, which causes some delay in the first rendering. To do this, we can turn our **CodeEditor** component into a slow component.

**LazyCodeEditor**
```jsx
import React, { Suspense } from 'react'
const CodeEditor = React.lazy(() => import('./CodeEditor'))

export default function LazyCodeEditor(props) {
  return (
    /* Here you can pass a better fallback if you want to. */
    <Suspense fallback={<></>}>
      <CodeEditor {...props} />
    </Suspense>
  )
}
```

- **React.lazy** allow us to do dynamic imports.
- **Suspense** will render the component only when it is resolved. You can use the **fallback** props to pass a loader component.

So we're loading the **CodeEditor** only when this code is called and not at build time. The **Suspense** component will wait for the **CodeEditor** to be resolved and display it, meanwhile displaying the fallback. Now, to see the difference, we'll import the **LazyCodeEditor** instead of the **CodeEditor** and run some analyses again.

**Code.jsx**
```jsx
import React from "react";
import LazyCodeEditor from "./LazyCodeEditor";

const Code = () => (
  <>
    <h1>Code</h1>
    <LazyCodeEditor />
  </>
);

export default Code;
```

```bash
yarn build
yarn analyze
```

We can see the `brace` and `react-ace` split into a different chunk:

![Analyze after splitting](./analyze-2.png)

And we can see that initial load time has decreased.

![Time to load resources in browser after splitting](./load-browser-2.png)

Our app only loads the chunks related to the code-editor when the user needs them:

![Loading chuncks to code editor](./code-page.png)

Now, we have an app which uses **Code Splitting** to have better performance and user experience as well. I hope you have enjoyed this article and it has been useful to you. Thanks for reading!

#### References
- [https://reactjs.org/docs/code-splitting.html](https://reactjs.org/docs/code-splitting.html)