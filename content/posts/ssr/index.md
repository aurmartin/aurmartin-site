---
title: "React server-side rendering from strach"
date: 2023-02-13T09:23:46+01:00
draft: false
---

Server-side rendering (SSR) is a technique that allows web pages to be rendered on the server and sent to the client as fully-formed HTML documents. While traditional client-side rendering (CSR) has many benefits, including faster navigation and a more dynamic user experience, it also has some drawbacks, such as slower initial page load times, poorer SEO performance, and limited accessibility for users with slow connections or outdated browsers.

React, a popular JavaScript library for building user interfaces, can be used for both CSR and SSR. In this article, we will focus on the latter, and show you how to implement React server-side rendering from scratch. We will start with a brief overview of the benefits and drawbacks of SSR, and then dive into the technical details of how to set up a basic SSR environment for a React application. We will cover topics such as server-side routing, data fetching, and server-side rendering of React components, and provide you with practical examples and code snippets along the way.

By the end of this article, you should have a solid understanding of how SSR works in React, and be able to apply this knowledge to your own projects. So let's get started!

## Server-side rendering process

The basic process of SSR in React is as follows:

1. The server receives a request from the client for a specific URL.
2. The React application is rendered on the server using [ReactDOMServer](https://reactjs.org/docs/react-dom-server.html) object.
3. The server sends the generated HTML back to the client as the initial response.
4. The client receives the HTML, hydrates the React application using [ReactDOMClient](https://reactjs.org/docs/react-dom-client.html) by attaching event listeners and re-rendering it if necessary.

## Setting up the environment

We will create a basic React application rendered on a [Node.js](https://nodejs.org/) server. To keep things focused on the concept of server-side rendering, we'll create everything from scratch and use as few external dependencies as possible.

You'll just need to have a working Node.js installation, you can find more informations on the [Node.js website](https://nodejs.org/).

## Initial application

We will use a really simple http server for this tutorial:

```javascript
const http = require("http");

const port = 8080;

const html = `
<div id="root"></div>

<script crossorigin src="https://unpkg.com/react@16.14.0/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16.14.0/umd/react-dom.development.js"></script>
<script>
  ReactDOM.render(
    React.createElement("div", null, "Hello world!"),
    document.getElementById("root")
  );
</script>
`;

http
  .createServer((req, res) => {
    res.end(html);
  })
  .listen(port, () => {
    console.log(`Server is running on http://localhost:${port}`);
  });
```

You can create a `server.js` file with this provided source code an run it in your terminal using node:
```bash
node server.js
```

Once the server is up and running, open http://localhost:8080/ in your browser.

If you want to see the initial response sent by the server, you can use the curl command in your terminal. Here's an example:
```text
➜  react-ssr-from-strach curl http://localhost:8080

<div id="root"></div>

<script crossorigin src="https://unpkg.com/react@16.14.0/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16.14.0/umd/react-dom.development.js"></script>
<script>
  ReactDOM.render(
    React.createElement("div", null, "Hello world!"),
    document.getElementById("root")
  );
</script>
```

As you can see, the #root element is empty, which means that the application is currently only being rendered on the client side.


## Enabling server-side rendering

To use server-side rendering we need to call `ReactDOMServer.renderToString` on the server before sending the response.

We need to install React and ReactDOM on server-side:

```bash
npm install --save react react-dom
```

Now we can try to use `ReactDOMServer.renderToString`. If you start a new node interactive terminal, you can play a bit with this function.
For example:

```javascript
> const React = require("react");
undefined
> const ReactDOMServer = require("react-dom/server");
undefined
> const element = React.createElement("div", null, "Hello world!");
undefined
> ReactDOMServer.renderToString(element)
'<div>Hello world!</div>'
```

As you can see, this function render a React element and return a string.

To render the HTML document containing our frontend application rendered inside, we need to:
1. Use `ReactDOMServer.renderToString` to render our application,
2. Insert this HTML inside our `index.html` file.

We will use [mustache](https://www.npmjs.com/package/mustache) as a template system to build our HTML response.

```bash
npm install --save mustache
```

Now, we are ready to implements the server-side rendering:

```javascript
import { App } from "./client.js";

# ...

const html = (serverSideRender) => {
  return fs
    .readFile("index.mustache", "utf-8")
    .then((data) => {
      return Mustache.render(data, { serverSideRender });
    })
    .catch((err) => console.error(err));
};

const serverSideRender = ReactDOMServer.renderToString(App());

html(serverSideRender).then((response) => res.end(response));
```

You can rename `index.html` to `index.mustache` and insert our string here:
```html
<div id="root">{{{serverSideRender}}}</div>
```

On the client side, we can replace `React.render` with `React.hydrate` to avoid re-rendering our whole application.

And voilà! Server-side rendering is as simple as that!
