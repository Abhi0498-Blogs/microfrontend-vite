Full source code for this article is available on [GitHub](https://github.com/Abhi0498-Blogs/microfrontend-vite).

## In this article

- [In this article](#in-this-article)
- [Introduction](#introduction)
- [What is a Micro Frontend?](#what-is-a-micro-frontend)
- [Module Federation](#module-federation)
- [Why Vite?](#why-vite)
- [Creating a Micro Frontend using Vite + React](#creating-a-micro-frontend-using-vite--react)
  - [Setting Up the Host App](#setting-up-the-host-app)
  - [Setting Up the Remote App](#setting-up-the-remote-app)
  - [Creating Components in the Remote App](#creating-components-in-the-remote-app)
  - [Previewing the Remote App](#previewing-the-remote-app)
  - [Adding Module Federation to the Remote App](#adding-module-federation-to-the-remote-app)
  - [Serving the Remote App](#serving-the-remote-app)
  - [Adding Module Federation to the Host App](#adding-module-federation-to-the-host-app)
  - [Using Remote Components in the Host App](#using-remote-components-in-the-host-app)
  - [Serving the Host App](#serving-the-host-app)
- [Conclusion](#conclusion)

---

## Introduction

Micro Frontends: Enhancing Web Development with Vite and Module Federation

In this article, we'll explore the concept of Micro Frontends—a powerful architectural approach for web applications. Micro Frontends allow you to divide your front-end code into smaller, independently developed and deployable units. These units, known as micro frontends, offer numerous benefits, including increased development speed, scalability, and flexibility. By enabling different teams to work on separate parts of the front end while maintaining integration through an isolation layer, Micro Frontends help manage complexity and promote autonomy in front-end development.

## What is a Micro Frontend?

A Micro Frontend is an architectural approach for web applications where the front-end code is divided into smaller, independently developed and deployable units called micro frontends. This approach enhances development speed, scalability, and flexibility by allowing different teams to work on separate parts of the front end while maintaining integration through an isolation layer. It's a way to manage complexity and promote autonomy in front-end development.

## Module Federation

Module Federation is a key technology that enables a JavaScript application to load code dynamically from another application while sharing dependencies. When an application consuming a federated module lacks a required dependency, Webpack (the underlying technology) automatically fetches the missing dependency from the federated build source. This allows for efficient sharing and use of common libraries across multiple micro frontends.

## Why Vite?

While Module Federation was initially introduced in Webpack, the landscape of JavaScript development has evolved. Vite has emerged as a game-changer by providing lightning-fast build times. Combining Vite and Module Federation can unlock immense capabilities for developing micro frontends quickly and efficiently.

## Creating a Micro Frontend using Vite + React

Creating a micro frontend typically involves two main parts:

1. **Host Application**: This is the primary application that users interact with. It serves as the container for the micro frontends.

2. **Remote Application**: These are the micro frontends themselves, which act as building blocks for the host application.

Now that we have an understanding of the technologies we'll be using, let's dive into the practical implementation.

### Setting Up the Host App

To create a host application using Vite and React, run the following command:

```bash
# Using npm 6.x
npm create vite@latest host-app --template react

# Using npm 7+, add an extra double-dash:
npm create vite@latest host-app -- --template react
```

### Setting Up the Remote App

For the remote application, execute the following command:

```bash
npm create vite@latest todo-components -- --template react
```

This will create two folders, `host-app` and `todo-components`. Next, install the dependencies for both apps:

```bash
npm install
```

### Creating Components in the Remote App

In the `todo-components` app, create the following components:

```jsx
// components/List.jsx
import React from "react";

const List = (props) => {
  const { items } = props;
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
};

export default List;
```

```jsx
// components/Input.jsx
import React from "react";

const Input = (props) => {
  const { value, onChange, onSubmit } = props;
  return (
    <form onSubmit={onSubmit}>
      <div className="flex-row">
        <input type="text" value={value} onChange={onChange} />
        <button type="submit">Add</button>
      </div>
    </form>
  );
};

export default Input;
```

Now that the components are ready, make the following changes to the `App.jsx` file:

```jsx
// App.jsx
import { useState } from "react";
import reactLogo from "./assets/react.svg";
import viteLogo from "/vite.svg";
import "./App.css";
import Input from "./components/Input";
import List from "./components/List";

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <Input value={count} onChange={setCount} onSubmit={console.log} />
      <List items={["Learn React", "Learn Vite", "Make an awesome app"]} />
    </>
  );
}

export default App;
```

### Previewing the Remote App

To preview the components, run the following command in the `todo-components` app:

```bash
npm run dev
```

You should see the following output.

![Remote App Preview](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/47vwxju7ufh2b38l4w1h.png)

### Adding Module Federation to the Remote App

Now, let's add Module Federation to the `todo-components` app. First, install the necessary dependencies:

```bash
npm install @originjs/vite-plugin-federation --save-dev
```

Next, configure Module Federation in the `vite.config.js` file:

```js
// vite.config.js in todo-components
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "todo-components",
      filename: "remoteEntry.js",
      exposes: {
        "./List": "./src/components/List.jsx",
        "./Input": "./src/components/Input.jsx",
      },
      shared: ["react"],
    }),
  ],
  build: {
    modulePreload: false,
    target: "esnext",
    minify: false,
    cssCodeSplit: false,
  },
});
```

In this configuration:

- `name`: Specifies the name of the remote app.
- `filename`: Sets the name of the file generated by Module Federation.
- `exposes`: Lists the components to expose from the remote app.
- `shared`: Declares shared dependencies, such as React, to optimize the bundle size.

Now, build the remote app:

```bash
npm run build
```

This generates a `dist` folder in the `todo-components` app containing a `remoteEntry.js` file.

### Serving the Remote App

Serve the remote app locally:

```bash
npx serve dist
```

This serves the remote app on port 3000.

### Adding Module Federation to the Host App

To use the remote app components in the host app, set up Module Federation in the `vite.config.js` file of the host app:

```js
// vite.config.js in host-app
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import federation from "@originjs/vite-plugin-federation";

export default defineConfig({
  plugins: [
    react(),
    federation({
      name: "host-app",
      remotes: {
        todo_components: "http://localhost:3000/assets/remoteEntry.js",
      },
      shared: ["react"],
    }),
  ],
  build: {
    modulePreload: false,
    target: "esnext",

    minify: false,
    cssCodeSplit: false,
  },
});
```

In this configuration:

- `name`: Specifies the name of the host app.
- `remotes`: Lists the remote apps to be used by the host app. In this case, we have one remote app, "todo_components," and we provide the URL of its `remoteEntry.js` file.
- `shared`: Declares shared dependencies, such as React, to optimize the bundle size.

### Using Remote Components in the Host App

Now, you can import and use the remote app components in the host app's `App.jsx`:

```jsx
// App.jsx in host-app
import { useState } from "react";
import List from "todo_components/List";
import Input from "todo_components/Input";

function App() {
  const [newTodo, setNewTodo] = useState("");
  const [todos, setTodos] = useState([]);
  const onSubmit = () => {
    setTodos((prev) => [...prev, newTodo]);
    setNewTodo("");
  };

  return (
    <>
      <Input value={newTodo} onChange={setNewTodo} onSubmit={onSubmit} />
      <List items={todos} />
    </>
  );
}

export default App;
```

Here, we import the components from the remote app using the specified remote name, "todo_components."

### Serving the Host App

To serve the host app locally, run:

```bash
npm run dev
```

Now, you should see that the components from the remote app are being used seamlessly in the host app.

## Conclusion

In this article, we've explored the concept of Micro Frontends and demonstrated how to create a micro frontend architecture using Vite and React, enhanced with Module Federation. By leveraging the power of Vite's fast build times and Module Federation's dynamic code loading capabilities, you can efficiently develop and scale web applications in a modular and maintainable way. This approach empowers multiple teams to work on different parts of the app independently, promoting flexibility and agility in front-end development.

For further exploration and to apply these concepts to your own projects, feel free to adapt and expand upon the examples provided here. Micro Frontends, when used judiciously, can significantly streamline your web development workflow.
