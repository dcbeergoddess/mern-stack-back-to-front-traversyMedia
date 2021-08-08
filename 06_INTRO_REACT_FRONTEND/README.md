# Getting Started With React & The Frontend

## A Look At The The UI / Theme
* Ready to start with FrontEnd
* Created Custom HTML, CSS theme for the application (we'll be using it to grab from and use as JSX in React)
* Responsive Design
* Mainly CSS GRID w/ Flexbox

## Link To Theme Building Series On YouTube
* If you are interested in building the theme from scratch using HTML/CSS/Sass, TraversyMedia have a 4 part series on YouTube:
* [YouTube Theme Building Series](https://www.youtube.com/watch?v=IFM9hbapeA0&list=PLillGF-Rfqba3xeEvDzIcUCxwMlGiewfV)

## React & Concurrently Setup
* Set up React from our front end and run react server in our backend react server at the same time using `concurrently`
* Create REACT APPLICATION AND SET THIS UP IN CLIENT FOLDER `npx create-react-app client` (npx allows us to run create react app or other things like it without having to install it globally on our machine)
* CONCURRENTLY
    - set up scripts in package.json to allow us to start up both servers without having to cd into both and keep starting both up separately 
        - client: script to run client and prefix with client because we need to run it in the client folder
        - dev: will run both the server and the client -- we can do using concurrently (escape quotes with backslash)
        ```json
          "scripts": {
            "start": "node server",
            "server": "nodemon server",
            "client": "npm start --prefix client",
            "dev": "concurrently \"npm run server\" \"npm run client\""
          }
        ```
    - use `npm run dev` to start up both from the root of application
    - exit out with `ctrl + C` & `enter`
* install dependencies for the client side -- in client folder (cd into client, not in the root directory)
    - `npm i axios react-router-dom redux react-redux redux-thunk redux-devtools-extension moment react-moment`
        - **axios:** http requests (you can use fetch api, but we'll be doing some specific things with axios like creating some global headers and stuff like that)
        - **react-router-dom:** router 
        - **redux:** thunk -> middleware allows us to make a synchronous request in our actions | dev-tools: makes using redux dev tools a bit easier
        - **moment:** date and time library to format the date and time
        - **react-moment:** allows us to us moment withing the component 
* we are using our own git local version control right now so...
    - we're going to delete the .gitignore and README.md in the client folder
    - remove .git file from as well in terminal in client folder: `rm -rf .git` 
* in client side package.json --> we need to add a proxy so we don't have to type `http://localhost` etc. when we are using axios, make is so we can just do api/ whatever profile
    - add proxy after last item, browerslist (make sure it's after and not within)
    ```json
        "browserslist": {
          "production": [
            ">0.2%",
            "not dead",
            "not op_mini all"
          ],
          "development": [
            "last 1 chrome version",
            "last 1 firefox version",
            "last 1 safari version"
          ]
        },
        "proxy": "http://localhost:5000"
    ```
* RUN SERVER AGAIN



## Clean Up & Initial Components

## React Router Setup

## Register Form & useState Hook

## Request Example & Login Form