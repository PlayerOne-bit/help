# __MERN Stack Tutorial:__

## I. __Set Up__

First, we need node package manager to install frameworks and dependencies via terminal
[Install Node First](https://nodejs.org/en/download)

__Note:__ To run npm, git, or any commands in any IDE with terminal...
Go to environment variable, paste the path of the installed command to the user variables.
1. In windows search, `System Properties`
2. `Environment Variables`
3. `Path (User variables)`
4. `Edit`
5. `New`
6. Paste the `Parent folder path of the installed .exe`
7. `OK` everything

Learn [GIT](GIT.md)?

## __II. Backend Introduction__

So In the terminal of your [Visual Studio Code](https://code.visualstudio.com/download?_exp_download=d53503e735)

To create a `package.json` run this command:
```npm
npm init
```
This would help install node dependencies such as:
- Express.js for backend framework
- dotenv to store keys inside .env file
- nodemon to run program more dynamically and fast change by save
- mongoose for MongoDB database 
```npm
npm install express dotenv nodemon mongoose
```
### Follow this structure:
```
|__config/              # Database and environment configurations
|__controllers/         # Handles incoming HTTP requests and responses
|__middlewares/         # Auth guards, logging, and error handlers
|__models/              # Defines database schemas / ORM models
|__routes/              # Maps URL endpoints to specific controllers
|__services/            # Contains core business logic and calculations
|__.env                 # Stores private environment variables and credentials
|__.gitignore           # List of sensitive file names for git control
|__index.js             # Entry point of the application to start the server
|__package-lock.json    # Locks down the exact versions of installed packages
|__package.json         # Lists project dependencies, metadata, and 
```
Create an `index.js`
```javascript
//load .env variables
require('dotenv').config();

//import dependencies
const express = require('express');
const mongoose = require('mongoose');

//models
const ModelName = require('./models/ModelName');

//routes
const routesName = require('./routes/routesName');

//Start Express.js App
const app = express();
const PORT = process.env.PORT || 5000;

//Global Middlewares
app.use(express.json());

//Connect to Database
mongoose.connect(process.env.MONGO_URI)
  .then(() => console.log('Connected to MongoDB!'))
  .catch((err) => console.error('MongoDB connection error:', err));

//Check if backend is working
app.get('/', (req, res) => {
  res.send('Your backend is running!');
});

//API Routes Registration
app.use('/api/routes',routesName);

//Start server listener
app.listen(PORT, ()=>console.log(`Server is running on port ${PORT}`));

```

# TO BE CONTINUED... 09/10/2026 4:42 last update