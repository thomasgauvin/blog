---
layout: post
title:  "How to create a React frontend and Express backend app"
permalink: "create-react-frontend-express-backend-app"
date:   2022-01-23 15:22:41 -0400
image_path: uploads/React_express.png
image_alt: Screenshot of the simple React Express full-stack application built
description: I often come back to guides to create new React and Express applications, so in this blog post I document how to create a simple React frontend and Express backend application.
---

![](uploads/React_express.png)

React and Express are two popular technologies to use when developing a full-stack web application. The Express app provides a REST API that is called asynchronously by the React application. This guide will get you started with a React frontend and a Express backend app as quickly as possible.

## Prerequisites

* [NodeJS](https://nodejs.org/en/)

## 1. React Frontend

The first step is to set up our React frontend project [as documented on the React website](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app).

1. From your project folder, run `npx create-react-app frontend` in a terminal session, replacing `frontend` with the name you want for your React app. This will use the `create-react-app` npm package to create the `frontend` folder and all the necessary files for your React app within it.

    ```
    npx create-react-app frontend
    ```

2. Change directories to the frontend folder with `cd frontend`. You will now be able to run your frontend React app by entering `npm start` in a terminal session

    ```
    cd frontend
    npm start
    ```

## 2. Express Backend

The second step is to set up the Express backend project [as documented on the Express website](https://expressjs.com/en/starter/installing.html).

1. From your root project folder, create a new folder `backend` (name it however you want to name your backend).

    ```
    cd ..
    mkdir backend
    ```

2. Within the `backend` folder, run `npm init` in a terminal session to create the NodeJS project. This will take you through a series a steps to define your NodeJS backend project (you can leave it as default/empty).

    ```
    cd backend
    npm init
    ```

3. Run `npm install express` within the `backend` folder to install Express to your backend project. 

    ```
    npm install express
    ```

4. Run `npm install cors` within the `backend` folder to install the [CORS package](https://expressjs.com/en/resources/middleware/cors.html). [CORS, which stands for cross-origin resource sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), will allow our Express backend to respond to network requests from another origin, in this case the origin of our React application. Without this, our Express backend (which will be hosted on localhost:3001) would not accept requests from our React frontend (which will be hosted on lcoalhost:3000).

5. Create a new file called `index.js`. This is the default entry point for the NodeJS project. (If you've specified a different file name as entry point during step 2, create that file.)

    ```
    touch index.js
    ```

6. Let's start with simple boilerplate code for our backend project by pasting the following within `index.js` using your preferred code editor, [taken from the Express documentation](https://expressjs.com/en/starter/hello-world.html).

    ``` javascript
    const express = require('express')
    const cors = require('cors')

    const app = express()
    app.use(cors());
    const port = 3001

    app.get('/', (req, res) => {
        res.send({
            message: "Hello World from Express API backend!"
        })
    })

    app.listen(port, () => {
    console.log(`Example app listening on port ${port}`)
    })
    ```

7. We can now run our Express app using the command `node index.js`.

    ```
    node index.js
    ```

In the above code snippet, we first start by importing the required packages. Then, we initialize the Express app, and indicate to the app that it must use the cors package as a middleware. This will allow requests from all origins, which is acceptable for our demo purposes. Ideally, requests should only be allowed from specific origins for security reasons.

Then, we indicate our first endpoint, a GET, and we define the response. Finally, we start the Express app.

## Improving our backend (Optional)

Within our `package.json` file, we can add the following to the `scripts` object so that we can run our backend project with the `npm start` command instead.

```json
"start": "node index.js"
```

As a result, our package.json will look like this:

```json
{
    "name": "backend",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "node index.js"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
        "express": "^4.17.2"
    }
}
```

Now, we can start our project using the `npm start` command.

## Conclusion

We are now able to make API calls from our frontend React app to our backend Express project, using the [fetch command](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).
 
A typical call from our frontend React project to our backend Express project would look like this:

```javascript
fetch('localhost:3001')
  .then(response => response.json())
  .then(data => console.log(data.message));
```

For demonstration, I called the API from the frontend in `App.js`. Upon page load, I fetch the backend API and store
the response in a state variable `message`. It is then displayed in the component.

```jsx
import logo from './logo.svg';
import './App.css';
import { useState, useEffect } from 'react';

function App() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    fetch('http://localhost:3001')
      .then(res => res.json())
      .then(data => setMessage(data.message));
  }, [])
  
  return (
    <div className="App">
      <header className="App-header">
        <p>
          {message}
        </p>
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
      </header>
    </div>
  );
}

export default App;
```