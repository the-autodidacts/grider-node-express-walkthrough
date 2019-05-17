# Express Passport Setup. Get a User Profile from Google
## 1. Express Api Setup 
* create server directory
* npm init the directory
* create index.js file in the dir
* require express in index.js
* create an instance of express() 
```javascript
// /index.js
 const app = express()
 ```
* create your first app.get('/', (req, res) => ) route to test express is working
* make the app listen on a port 
```javascript
// /index.js
PORT = 5000 
app.listen(PORT)
```

## 2. Optional Heroku deployment you can skip to OAuth Set-up below.
* do dynamic port binding through process.env 
```javascript 
// /index.js
PORT = proccess.env.PORT || 5000
```
* Specify Node and NPM environment in package.json 
```json  
// /package.json
  "engines": {
    "node": "8.1.1",
    "npm": "5.0.3"
  }
  ```
* specify your start script this can be done right under what we just did to specify our environments
```json
// /package.json
  "scripts": {
    "start": "node index.js"
    }
```
* create a .gitignore file at the document root and remember to add node_modules
### make sure you have the Heroku CLI installed and then 
* `heroku -v ` should give you your Heroku version number. 
* `heroku login`
* `heroku create` in your root app directory
* push to the remote git repo using second link supplied to you after you ran heroku create
* `git remote add <you remote git from heroku>`
* `git push heroku master`
* test your app with `heroku open` this should open your browser and the url of your app.


## 3. OAuth Set-up, installing and configuring PassportJS

### Install passport and the Google oauth20 strategy. Passport has many strategies for different OAuth services
`npm install --save passport `
`npm install --save passport-google-oauth20`

* Require passport and require GoogleStrategy in your index.js file( we will refactor later)
```javascript
// /index.js
  const passport = require('passport');
  const GoogleStrategy = require('passport-google-oauth20').Strategy
```

* Let passport know how to make use of the GoogleStrategy we just created. 
```javascript
// /index.js
passport.use(new GoogleStrategy());
```
* Shortly we will pass in a configuration object to the GoogleStrategy above.
 
## 4. OAuth Set-up, Creating a ClientID and Secret key on Google Developer Console. 
* Go to [console.developers.google.com](https://console.developers.google.com)
* Click on credentials
* Click on OAuth client ID from the popup 
* Select web application and click create
* Give it a name. 
* On the top right of the screen Where the hambuger is and it says GoogleApi. Verify that you are in the project that you want to create this credential for. If you are not click on the project name to switch to another project or to create a new project. 
* Copy the ClientID and the Secret Key. Do not loose these. 
## 5. OAuth Set-up, add your keys to a config file so you can start to use them in your app and pass them into your strategy to be used. 
* Create a config folder in your root directory and add a file, call it keys.js. Then module.exports an object with your clientID and ClientSecret this is the file where we will be adding any other secrets we need to keep out of our public git repo. 
* Add `keys.js` to your .gitignore
```javascript
// /config/keys.js
module.exports = {
  googleClientID: "yourclientIDalksdjfal9eifm94jgmqkjeoiawje",
  googleClientSecret: "YourSecretKeya134lksdfja243567lsdjf"
}
```

* Import keys into your index.js 
```javascript 
// /index.js
const keys = require(./config/keys)
```
* Pass a configuration object with our keys into our passport.use(new GoogleStrategy()) 
* In the next three code blocks we are just adding more info to the same block. You don't have to copy this three times. I'm just showing the incremental steps we will be taking. 

```javascript 
// /index.js
passport.use(new GoogleStrategy({
  clientID: keys.googleClientID, 
  clientSecret: keys.googleClientSecret
}))
```
* We also want to add another key to GoogleStrategy. A callback url. This is the url that Google is going to hit that will contain a code as a query string so that we can then go to Google and ask it for the users information. So after clientSecret also add `callbackURL: 'the/route/for/the/callback'`


```javascript 
// /index.js
passport.use(new GoogleStrategy({
  clientID: keys.googleClientID, 
  clientSecret: keys.googleClientSecret,
  callbackURL: '/auth/google/callback'
}))
```

* The second argument to `GoogleStrategy({},secondArg) ` is a callback function. This callback function has 4 paramaters that are of interest to us. `accessToken`, `refreshToken`, `profile` and `done` 
* lets console.log accessToken since its the first argument.

```javascript 
// /index.js
passport.use(new GoogleStrategy({
    clientID: keys.googleClientID, 
    clientSecret: keys.googleClientSecret,
    callbackURL: '/auth/google/callback'
  }, (accessTtoken) => {
    console.log(accessToken)
  })
)
```

* Make sure the GoogleStrategy() above gets a callback function as the second argument with paramater of accessToken you can console.log it. Like accessToken we also have access to refreshToken profile and done from this callback's params. 

## 6. Create your route handlers 

* Create two routes one for the google call back and one for "/auth/google"
* Both routes are get auth/google neither take a req or res instead the second arg is passport.authenticate() 

* For the auth route you need to pass the strategy "google" as the first argument.  
* The second argument is  a configuration object with `scope:` as the key and and array as the value with the strings `'profile'` and `'email'` in its first two indices.

```javascript 
// /index.js
  app.get(
    "/auth/google",
    passport.authenticate("google", {
      scope: ["profile", "email"]
    })
  );
```


* For the callback route only the strategy needs to be passed in as a string 
```javascript
// /index.js
app.get("/auth/google/callback", passport.authenticate("google"));
```

## 7. We're done lets refactor our code into folders
* create the following structure

    
```
.
|-- config
|   `-- keys.js
|-- index.js
|-- package.json
|-- routes
|   `-- authRoutes.js
`-- services
    `-- passport.js
```
    
* config folder will hold our keys.js file
```javascript
// /config/keys.js
module.exports = {
  googleClientID: "yourclientIDalksdjfal9eifm94jgmqkjeoiawje",
  googleClientSecret: "YourSecretKeya134lksdfja243567lsdjf"
}
```

* Services folder will contain passport specific things for now. We need to make sure this code runs so in the /index.js file we need to `require('./services/passport')`
```javascript
// /services/passsport.js

const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth20").Strategy;
const keys = require("../config/keys");

passport.use(new GoogleStrategy({
    clientID: keys.googleClientID, 
    clientSecret: keys.googleClientSecret,
    callbackURL: '/auth/google/callback'
  }, (accessToken) => {
    console.log(accessToken)
  })
)

```

* Export our authRoutes module routes as module.exports = (app) => {routes} because we will need to import this into our `index.js` file since that is where our server lives. The anonymous function within authRoutes that we are exporting is expecting an express instance in order to be able to create the route handlers hence the app parameter. 
```javascript
// /routes/authRoutes.js

const passport = require("passport"); 

module.exports = (app) => {
// notice that we are working with the express app object we need to send 
// this in from index.js since that is where it was defined. 
  app.get(
    "/auth/google",
    passport.authenticate("google", {
      scope: ["profile", "email"]
    })
  );

  app.get("/auth/google/callback", passport.authenticate("google"));
}
```

* in `index.js` fix require statements and invoke the routes module with app
`const authRoutes = require('/routes/authRoutes') `
`authRoutes(app)`
or simply 
`require('/routes/authRoutes')(app)`
also require passport from our services folder
`require('./services/passport')` <-- not having this will throw an error of "no strategy <strategy name> "

```javascript 
// /index.js
const express = require("express");
require("./services/passport");
const app = express();

app.get("/", (req, res) => res.json({ testing: "Testing" }));
//authRoutes returns a function from the module.exports we are passing app into that function making it available to authRoutes
require("./routes/authRoutes")(app);

PORT = process.env.PORT || 5000;
app.listen(PORT, console.log("listening on port: ", PORT));
```

# Next we set up MongoDB and Mongoose 

# Express Mongo and Mongoose Setup for OAuth after setting up PassportJS. 
## Authentication part using OAuth to uniquely identify an individual.

### 1. Create a [Mongo Atlas](https://cloud.mongodb.com) account. 
#####  We are using a remotely hosted instance if you want to install MongoDB on your machine and run the server locally you can you will just need to change the settings accordingly. 
* Create a new project should be on your top left
* Once created click on `Build a New Cluster` in the top right 
* Select AWS and Free tier
* Give your cluster a name. 
* Give it a user name and password. I recommend let the service provide a password for you since some characters have to be escaped it can cause some issues if you create your own password with characters like `@`
* copy that password and put it someplace safe.
* Click on CONNECT 
* Choose `connect your application`
* For your driver choose Node.js and your version. 
* copy the connection string you will replace `<password>` with the password you created earlier. In the next steps. 

### 2. Connecting Mongoose to MongoDB
* In terminal install Mongoose in your root project directory. 
`npm install --save mongoose`

* require mongoose in your index.js file and connect to it. 
```javascript
// /index.js
const mongoose = require('mongoose');
// This will be your localhost and a port if not using MongoAtlas. 
mongoose.connect("youconnectionURIFromStep1");
```
* A good idea would be to move your Connection URI to the config file since it stores private data like your DB user name and Password.
* add the connection uri to the keys.js file
```javascript
// /config/keys.js
module.exports = {
  mongoURI: "youconnectionURIFromStep1"
}
```
* Let's bring in `/config/keys.js` file

```javascript
// /index.js
const keys = require('./config/keys.js');
const mongoose = require('mongoose');
// This will be your localhost and a port if not using MongoAtlas. 
// were now pulling the URI from the keys file. 
mongoose.connect(keys.mongoURI);
```

### 3. Create Models and Schema to associate your model class to your Mongo collection 

* create a new folder in the root of your project called models.
* Create a User.js model file inside the models folder. This will be our User Model Class
* `require mongoose`
* pull the Schema property from the mongoose object

```javascript
// /models/User.js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
// line above can be destructured as such 
// const { Schema } = mongoose;
```

* Create a schema instance and pass it an object to describe the properties of User model and their types. We can add more properties later if we need them. They don't have to be declared at once. You are free to add and subtract properties as you please. 

```javascript
// /models/User.js
const userSchema = new Schema ({
  googleId: string;
})
```

* Create a model class and tell mongoose to be aware that this collection needs to be created. The first argument is the name of the collection to be created if it doesn't already exist, the second argument is the schema to use which is the schema we just created. 

```javascript 
// /models/User.js
mongoose.model('users', userSchema )
```

* Require this file in index.js so that it runs and the configuration loads. 
```javascript
// /index.js
// this line must be above the require('./services/passport') because in step 
// 4 we will call the User Model in the passport.js file if this line is below 
// it will not know that the User Model is available because of the order in which things load 
require('./models/User')
```

### 4. Persisting the User to the DB
* A good place to save the user is in the /services/passport.js file here the passport.use() method created an instance of the GoogleStrategy() the second argument to the GoogleStrategy instance was a callback function that had 4 parameters one of the paramaters was profile which contains the user profile associated with the Google account. This callback function is invoked everytime we go through the google auth flow. We can use the profile parameter to store or update the db record here. 

* Update the passport.js file to require in the mongoose library 
* Require the User class a little differently than we normally require by using mongoose  

```javascript
// /services/passport.js
const mongoose = require('mongoose');
const User = mongoose.model('users'); //the User object is our Model class we can use
//it to save it or persists to the db
```

* create a new user in the GoogleStrategy callback function and save it to the db by calling `.save()`

```javascript
// /services/passport.js

passport.use( new GoogleStrategy(
  {
    clientID: 'blah',
    clietnSecret: 'blah',
    callbackURL: '/auth/google/callback'
  }, 
  (accessToken, refreshToken, profile, done) => {
      new User({googleId: profile.id}).save()
    }
  )
);
```

* Creating validations to not create duplicate user records. The above works fine but if we go through the OAuth flow twice then we create a second record in the database. This is not what we want so lets modify the callback function to check if the uers exists first. If it exists we will skip user creation if not we will create a user. 



* Use the findOne method on the User class in the GoogleStrategy callback. All mongoose calls are async so use a promise `.then` or `async` `await`, instead of trying to assign the return value to a new user variable. We will use a promise below. 

```javascript 
// /services/passport.js
(accessToken, refreshToken, profile, done) => {
  User.findOne({googleId: profile.id})
    .then(existingUser => {
      if (existingUser) {
        console.log(existingUser) // we already have  a record with the id
      }
      else {
        new User({googleId: profile.id}).save()
      }
    })   
  }
);
```

* Call the done function in the callback to let passport know that we are done. Done takes two argument an error and the second argument is the User record. 

```javascript 
// /services/passport.js
(accessToken, refreshToken, profile, done) => {
  User.findOne({googleId: profile.id})
    .then(existingUser => {
      if (existingUser) {
        console.log(existingUser); // we already have  a record with the id
        done(null, existingUser);
      }
      else {
        new User({googleId: profile.id}).save()// remember .save is async so .then
        .then(user => {
          done(null, user);
        })
      }
    })   
  }
);
```

### 5. Encoding Users using Serialize and Deserialize to help set and retrieving cookies later.

* This step creates a cookie encodes and decodes it so that you can return to the app with out having to sign in. We will define a functiont that we will passs to serializeUser and to deserializeUser. That will essentially turn a user into a cookie and turn a cookie back into a user. 

* In passport.js call `passport.serializeUser()` and pass it a function that we will define. The first argument of our function will be the user model and the second is done. The user argument is what was found in the GoogleStrategy callback. That is what we pulled out of the database after creating or finding a user in the previous step. In the callback function invoke done with null and user.id. This confirms there are no errors and the user.id from our db lets serializeUser do its magic. 

```javascript
// /services/passport.js
passport.serializeUser((user, done)=> {
  done(null, user.id)
})

```

* define `deserializeUser()` . Take the id that we previously stuffed in the cookie and turn it back into a User model. 

```javascript
// /services/passport.js
passport.deserializeUser((id, done)=> {
  User.findById(id)
    .then(user =>{ 
      done(null, user)
    })
})

```

### 6. Handle Cookies with helper library. 
* Express has no idea how to handle cookies so we are going to use a library called cookie-session to help with that.

`npm install --save cookie-session`

* Tell express that it needs to make use of cookies in our app. First we need to require cookie-session and passport, this should go in our index.js

```javascript
// /index.js
const cookieSession = require('cookie-session');
const passport = require('passport');
```
##  Note 
####  we will be using several middlewares through out the project.
*  express uses `app.use()` which takes an argument to set up middlwares.When you see app.use we are just wiring up middleware

* Now we tell Express to use cookie session using `app.use()`, cookie-session takes a configuration object as an argument. The object has two properties. maxAge and keys. maxAge sets the expiration date of the cookie in milli seconds. Keys is a decription key that we are going to store in our /config/keys.js file like we have stored all our other secrets. keys value must be an array even if its just one key that you decide to use. 

```javascript 
// /index.js
app.use(
  cookieSession({
  maxAge: 30*24*60*60*1000,
  keys: [keys.cookieKey]
  })
)
```

* Now we tell Passport to use Cookies to handle authentication having not having these lines will cause 
  you to not get a user object when you authenticate you will get undefined instead at /auth/google
```javascript
// /index.js
app.use(passport.initialize());
app.use(passport.session());
```

* At this time we have done everything required for authentication flow. We just don't have a means of testing it out yet.
* If  a user has been throw the Auth flow and has cookie the following is what occurs
  * request comes in
  * cookie-session extract cookie data from your browser
  * passport pulls user id out of cookie data
  * deserializeUser turns the id in a user instance 
  * User model instance added to req object as `req.user`

### 7. Test authentication and log-out by adding new routes.
* add get route handlers to app in authRoutes to get current user and to log out.

```javascript
// /routes/authRoutes.js
app.get("/api/current_user", (req, res) = > {
  res.send(req.user) // just to test that we got back our user model instance.
});

app.get("/api/logout", (req, res) => {
  req.logout()
  res.send(req.user)
});
```


### 8. Putting it all together. 
* This is the way your directory should look once its all said and done. Let me show you all the files that we worked on to get this to work. 


```
.
|-- config
|   `-- keys.js
|-- index.js
|-- models
|   `-- User.js
|-- package.json
|-- routes
|   `-- authRoutes.js
`-- services
    `-- passport.js
```
```javascript
// /config/keys.js
module.exports = {
  googleClientID:
    "YourGoogleClientIDhflahkgoogleusercontent.com",
  googleClientSecret: "YourGo0gl3ClientSecret",
  mongoURI: "mongodb+srv://YourMongoDbUserName:YourMongoDBPassword@cluster0-xjhia.mongodb.net/test?retryWrites=true",
  cookieKey: 'YouCanPutAnyStringInHereTheLongerAndMoreRandomTheBetter'
};

```

```javascript
// /models/User.js
const mongoose = require('mongoose');
const {Schema} = mongoose;

//create a user schema with and object that describes all the different properties and the type of the property
//We can add more properties when ever we want.
const userSchema = new Schema({
    googleId: String
});

mongoose.model('users',userSchema)
```


```javascript
// /routes/authRoutes.js
const passport = require("passport"); 

module.exports = (app) => {
  app.get(
    "/auth/google",
    passport.authenticate("google", {
      scope: ["profile", "email"]
    })
  );

  app.get("/auth/google/callback", passport.authenticate("google"));

  // logout is a function that is automagically attached to the req object.
  // It takes the cookie and kills it and you are loggedout. 
  app.get('/api/logout', (req, res)=> {
    req.logout();
    //proves that they are signed out as undefined or no content
    res.send(req.user)
  })

  // user is automagically attached to the req object by passport.
  app.get('/api/current_user', (req, res)=> {
    res.send(req.user)
  })
};
```

```javascript
// /services/passport.js

const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth20").Strategy;
const keys = require("../config/keys");
const mongoose = require("mongoose");

//The User object is our Model Class we can create a new model instance and save it to the db
const User = mongoose.model("users");
          

//Generate a token and give to the user by defining a function called serializeUser
//Done is a call back first argument is error second arg is identifying 
//piece of info. This id is the mongo object id.
passport.serializeUser((user, done)=> {
  done(null, user.id)
});

//id is the id from the serializeUser user.id it is now used to deserialize the user. 
passport.deserializeUser((id, done) => {
  User.findById(id)
    .then(user => {
      done(null, user)
    });
});

passport.use(
  new GoogleStrategy(
    {
      clientID: keys.googleClientID,
      clientSecret: keys.googleClientSecret,
      callbackURL: "/auth/google/callback"
    },

    //this call back function has the profile passed in as an argument which we will
    //use to create a new User with the googleId <- one of the attributes we created
    //in the User.js Model and assign profile.id to it from Google
    //.save() saves it to the db.

    (accessToken, refreshToken, profile, done) => {
      User.findOne({ googleId: profile.id }).then(existingUser => {
        //existingUser is argument returned after the promise resolves if there is
        //user with googleId === profile.id it is a null value it is a mongoose record instance

        if (existingUser) {

          //first argument is the error so we pass null second argument is the user record
          done(null, existingUser);
        } else {
          //create a new Mongo user record.
          new User({ googleId: profile.id })
            .save()
            .then(user => done(null, user));
        }
      });
    }
  )
);

```


```javascript
// /index.js
const express = require('express')
const mongoose = require('mongoose')
const cookieSession = require('cookie-session')
const passport = require('passport')
const keys = require('./config/keys')

//Make sure you require the /models/User before /services/passport 
require('./models/User')
require('./services/passport')

const app = express();


//Using cookieSession which takes a configuration object with a maxAge to set the
//expiry on the cookie must be in milli secs hence my multiplaction (expires in 30 days)
//and and encription key to salt the cookie. You can pass an array of keys and 
//it will be set randomly for more security. Keep these in your config keys 
app.use(
    cookieSession({
        maxAge: 30 * 24 * 60 * 60 * 1000,
        keys: [keys.cookieKey]
    })
)

//Now tell passport to use cookies to handle authentication
app.use(passport.initialize());
app.use(passport.session());

mongoose.connect(keys.mongoURI)
require("./routes/authRoutes")(app)

//start listening on specified port
const port = process.env.PORT || "5000"
app.listen(port, console.log("listening on: ", port))

```

# Client Side Setup With React 

### 1. Create two seperate resources for production data.
* This means creating a new prod google api key and prod mongo atlas key
* This means adding your domain and ip to the mongo whitelist and the domain to Google 
* Change keys.js to ask if we are in prod or dev env if we are in dev return dev.js dev.js will store what keys.js was storing previously else use env variables.
* Don't commit the `dev.js` however now you can commit the `keys.js`
* Add your new production env variable to Heroku from heroku.com at reveal config variables 


```javascript 
// /server/config/keys.js
if (process.env.NODE_ENV === 'production'){
  //we are in production - return the production set of keys
  module.exports = require('./prod')
} else {
  //we are in development - return the dev keys.
  module.exports = require('./dev')
}
```

* prod js will reference the production variables. 
```javascript
// /server/config/prod.js 
module.exports = {
    googleClientID: process.env.GOOGLE_CLIENT_ID,
    googleClientSecret: process.env.GOOGLE_CLIENT_SECRET,
    mongoURI: process.env.MONGO_URI,
    cookieKey: process.env.COOKIE_KEY
};
```

* Either create a REDIRECT_URI key for production and for DEV and use in it passport service after callback URL becuase https doesn't trust heroku proxy 
* or add the option proxy: true to to the GoogleStrategy config object 

```javascript
// /server/services/passport.js
passport.use(
  new GoogleStrategy(
    {
      clientID: keys.googleClientID,
      clientSecret: keys.googleClientSecret,
      callbackURL: "/auth/google/callback",
      proxy: true
    },
    .
    .
    .
```

# Client Side Setup With React 

### 1. Create two seperate resources for production data.
* This means creating a new prod google api key and prod mongo atlas key
* This means adding your domain and ip to the mongo whitelist and the domain to Google 
* Change keys.js to ask if we are in prod or dev env if we are in dev return dev.js dev.js will store what keys.js was storing previously else use env variables.
* Don't commit the `dev.js` however now you can commit the `keys.js`
* Add your new production env variable to Heroku from heroku.com at reveal config variables 


```javascript 
// /server/config/keys.js
if (process.env.NODE_ENV === 'production'){
  //we are in production - return the production set of keys
  module.exports = require('./prod')
} else {
  //we are in development - return the dev keys.
  module.exports = require('./dev')
}
```

* prod js will reference the production variables. 
```javascript
// /server/config/prod.js 
module.exports = {
    googleClientID: process.env.GOOGLE_CLIENT_ID,
    googleClientSecret: process.env.GOOGLE_CLIENT_SECRET,
    mongoURI: process.env.MONGO_URI,
    cookieKey: process.env.COOKIE_KEY
};
```

* Either create a REDIRECT_URI key for production and for DEV and use in it passport service after callback URL becuase https doesn't trust heroku proxy 
* or add the option proxy: true to to the GoogleStrategy config object 

```javascript
// /server/services/passport.js
passport.use(
  new GoogleStrategy(
    {
      clientID: keys.googleClientID,
      clientSecret: keys.googleClientSecret,
      callbackURL: "/auth/google/callback",
      proxy: true
    },
    .
    .
    .
```

## Create React App Setting Up The Client!

### 2. In the server folder of your project create a react application and call it client

* It is important that you call this app client. 
* `npm create-react-app client`
* install concurrently to run express server and react app from the same terminal tab in the server directories package.json not the client package.json
* `npm install --save concurrently`
* edit server's package.json and add some start scripts

```
// /server/package.json
  "scripts": {
    "start": "node index.js",
    "server": "nodemon index.js",
    "client": "npm run start --prefix client",
    "dev": "concurrently \"npm run server\" \"npm run client\""
  }
```

* Now npm run dev will run both your express and your express client server. 

### 3. Setup a React proxy for your local development environment. 

* In the `client` directory run `npm install --save http-proxy-middleware` IT IS IMPORTANT THAT YOU INSTALL IN THE CLIENT DIRECTORY NOT THE SERVER
* Create `setupProxy.js` file in `client/src/` directory. There is no need to import this file anywhere, CRA looks for a file by this name and loads it.
*  Add your proxies to the `setupProxy.js` file:

```javascript
// /server/client/setupProxy.js
    const proxy = require('http-proxy-middleware')
     
    module.exports = function(app) {
        app.use(proxy(['/api', '/auth/google'], { target: 'http://localhost:5000' }));
    };

```

### 4. Install Packages for React

* We're going to install Redux, React Router and Thunk
* `npm install --save redux react-redux react-router react-router-dom thunk`

### 5. clean up boiler plate code and load up your own App component

* Remove everything in `/server/client/src/` except for the service worker *
* Add new `index.js` to src
* set up boiler plate for index.js It needs ReacDom.Render with two arguments the root component that we will create in the next step and the document root.

```jsx
// /server/client/index.js
import React from 'react';
import ReactDom from 'react-dom';
import App from './components/App' // being created in next step

ReactDom.render(<App /> ,
    document.querySelector("#root") //root dom node in our app 
);
```

* create a components directory in src and add `App.js` to it. This is our root component that will be loaded into our `index.js` 

```jsx 
// /server/client/components/app.js
import React from 'react';

const App = () => {
  return (
    <div>
      Testing
    </div>
  );
};
export default App;

```

### 6. Adding some Redux 

* import the provider and wrap the app component in it. 
* import createStore and applyMiddleware
* pass in reducers right now an empty array we will create one later
* pass in initial state just empty obj for now and applyMiddleware
* we will pass in Thunk shortly as an argument. 
```jsx
// /server/client/src/index.js
// .
// .
import { Provider } from 'react-redux';
import { createStore, applyMiddleware } from 'redux';

const store = createStore(() => [], {}, applyMiddleware())

ReactDom.render(<Provider> 
                  <App />
                </Provider> ,
    document.querySelector("#root") 
);
```

### 7. Add reducers

* Create reducers directory in the src directory
* inside the directory add `authReducer.js` and `index.js`
* insisde the authReducer create a reducer and immediately export it. 


```javascript
// /server/client/src/reducers/authReducer.js
export default function (state = {}, action){
  switch (action.type){
  default: 
    return state;
  }
}
```

* import the authReducer to the index.js in the reducers directory and combine with 
* the combineReducers call compliments of Redux

```javascript 
// /server/client/src/reducers/index.js
import { combineReducers } from 'redux';
import authReducer from './authReducer'; // we just created in the previous block

//the name of the keys is what will be available in our state object
export default combineReducers({
  auth: authReducer
})

```

* import your reducer to the top level index.js at the root /client/src directory
* replace our old dummy reducer with the imported one.


```javascript 
// /server/client/src/index.js
import reducers from './reducers'
// const store = createStore(() => [], {}, applyMiddleware())   <-- dummy reducer --:
const store = createStore(reducers , {}, applyMiddleware())

```
* at this point you can test your app it should be displaying testing.

### 8. Add some React router and navigation to your App component

* Import BrowserRotuer and Route
* create a few dummy components to test it out. 

```jsx
// /server/client/src/components/App.js
import { BrowswerRouter, Route } from 'react-router-dom';

const Header = () => <h2> Header </h2>
const Dashboard = () => <h2> Dashboard </h2> 
const SurveyNew = () => <h2> SurveyNew </h2> 
const Landing = () => <h2> Landing </h2> 
```

* No lets make use of it for navigation. 
* Wrap your links in the return statement in BrowserRouter component BroswerRouter only takes 1 child
* Add Wrap the component you want to link to with Route. 
* Route takes some configurations, `exact={true}`, `path="/path"`, `component={YouComponent}` exact can be shortened by dropping the `={true}`
* Not adding exact will match all the routes that path matches and render all of them.
* If you always want to display a component regardless of wether or not you hit that route but it where you want to display it with out a Route wrapper. In our example we use `Header`


```jsx
// /server/client/src/components/App.js
const App = ()  => {
  return (
    <div>
      <BrowserRouter>
        <div>
          <Header />
          <Route path="/" component={Landing}/>
          <Route path="/dashboard" component={Dashboard}/>
          .
          .
        </div>
      </BrowserRouter>
    </div>
  );
}
```

### 9. Create a class based component for the Header

* We will decide what to render depending on wether the user islogged in. 
* This will replace the dummy header we created in the previous block
* Create a class based componenet called Header in `Header.js`

```jsx 
// /server/client/src/components/Header.js
import React, { Component } from 'react';

class Header extends Component {
  render() {
    return (
    <div>
      Header!!
    </div>
    )
  }
}

export default Header;
```

* Now import into you App component and remove the old dummy header

```jsx
// /server/client/src/components/App.js
// const Header = () => <h2> Header </h2> <-- replace this --:
import Header from './Header';

```

### 10. Create an action creator to hit current_user route and update state Axios / Thunk
*  We want to check if an user is logged in so we will create an action creator that will check the current user route
*  We will conditional render parts of the header
*  It will say `log in with Google` or `log out` depending on wether the request returns a user
*  To make a request we are going to use axios instead of fetch so we will need to npm install it in the CLIENT directory
*  `npm install --save axios`
*  We will also make use of Thunk to dispatch actions asynchronously we already npm installed thunk earlier.

```jsx
// /server/client/src/index.js
import reduxThunk from 'redux-thunk';

const store = createStore(reducers, {}, applyMiddleware(reduxThunk))
```

#### ATTENTION
*  Create an action creator inside a file called `index.js` in a new directory `/server/client/src/actions`
*  We will import a type `FETCH_USER` that we will create in another file after this code block
*  This is not a regular action creator it is a redux thunk action creator so it will look different from our regular AC's
*  
#### DO NOT WRITE THIS BLOCK 
*  IT IS AN EXAMPLE OF A REGULAR ACTION CREATOR THE NEXT BLOCK IS WHAT WE NEED FOR ASYNC ACTIONS*

```javascript 
const fetchUser = () => {
  let request = axios.get('/api/current_user');
  
  // action object get returned immediately on non async functions
  return {
    type: FETCH_USER,
    payload: request
  }
}
```

#### WRITE THIS BLOCK
*  This is an async function so our function will return another function 
*  Redux Thunk knows if an action creator returns a function instead of an action it needs to deal with it  
*  It deals with this in the middleware we applied earlier by passing the dispatch function. 
*  We don't use that function until our api request is completed and we have the data we need.
*  THAT IS ALL REDUX THUNK DOES. 
*  
```javascript 
// /server/client/src/actions/index.js

import axios from 'axios';
import { FETCH_USER } from './types'

const fetchUser = () => 
  return function (){
    axios.get('/api/current_user')
    .then(res => dispatch({type: FETCH_USER, payload: res}));
  };
};
```
*  Create action types in that directory. `types.js` in `/server/client/src/actions/types.js`

```javascript 
// /server/client/src/actions/types.js
export const FETCH_USER = 'fetch_user'
```


### 11. Use the action creator in our App Component to Immediately know if user is logged in.

*  Refactor the App component to a class based component so we can use ComponentDidMount and call our action creator
*  Here is how our App component should look like now after refactoring into a class based component 

```jsx
// /server/client/src/components/App.js

import React, { Component } from 'react';
//Your router stuff should never go in the index.js just here in the App component
import { BrowserRouter, Route }  from 'react-router-dom';

import Header from './Header'

const Dashboard = () => <h2> Dashboard </h2> 
const SurveyNew = () => <h2> SurveyNew </h2> 
const Landing = () => <h2> Landing </h2> 

class App extends Component { 

  render () {
    return (
      <div className="container">
        <BrowserRouter>
          <div>
            <Header></Header>
            <Route exact path="/" component={Landing}/>
            <Route exact path="/surveys" component={Dashboard}/>
            <Route exact path="/surveys/new" component={SurveyNew}/>
            <Route />
          </div>
        </BrowserRouter>
      </div>
    );
  };
}
```

*  Use the `ComponentDidMount()` lifecycle method and wire the App component to be able to receive a action creators as props by using the connect helper from the react-redux library. 
* `import connnect`, `import actions`, call `ComponentDidMount()` in App class 
*  export app wrapped in connect helper



```javascript 
// /server/client/src/components/App.js
// ...
import { connect } from 'react-redux';
import * as actions from '../actions'; //pulls out all the actions creators that are in that directory
// ...
class App extends Component {
  componentDidMount(){
    this.props.fetchUser
  }
// ...
}

//first argument is mapStateToProps which we don't need right now second argument is actions we imported above.
export default connect(null, actions)(App);
```
*  You can put an console.log(action) in the authReducer to see when the action fires off and gets sent to the reducer.

### 12. Refactor our action creator (optional)

*   Arrow functions with async and await might make your action creator easier to read. Will work either way.

```javascript
// server/client/src/actions/index.js
import axios from 'axios';
import { FETCH_USER } from './types'

export const fetchUser = () => 
     async dispatch => {
        const res = await axios.get("/api/current_user")
        dispatch({type: FETCH_USER, payload: res}) 
    }
```

### 13. Not optional more on action creator and change of state in authReducer.JS
*  Instead of sending the entire response from axios only send the response data from the the block above it will look like the line below, fix accordingly if you did not refactor. 

`dispatch({type: FETCH_USER, payload: res.data}) `

*  In our authReducer we have 3 states for user logged in `null`, `user model`, `false`
*  If state is `null` then we are waiting to find out 
*  If state is the `user model` then the user is logged in
*  If state is `false` the user is logged out
*  
```javascript 
// /server/client/src/reducers/authReducer.js
import { FETCH_USER } from "../actions/types";

//three possible states null meaning we don't know a user object for logged in and false for logged out
export default function(state = null, action) {
  switch (action.type) {
    case FETCH_USER:
      return action.payload || false; //<-- coerces a false value from an empty string if payload does not contain any data
    default:
      return state;
  }
}
```

### 14. Back to the header! See Step 9. for our starting point. 
*  We want to use the connect helper from react-redux 
*  We want get our state from the store by calling mapStateToProps. We only care about the authReducer's state
*  We defined it earlier in the combine reducer index.js as {auth: authReducer}

```jsx
// /server/client/src/components/Header.js
import { connect } from 'react-redux';

{
// .
}

function mapStateToProps({ auth }){
  return { auth }
}

export default connect(mapStateToProps)(Header)
```

*  At this point we will conditionally render the header
*  If the authState is null we don't want to show anything.
*  If the user is logged authState true we want to display logout
*  if the user is logged out we want to display login with google


```jsx 
// /server/client/src/components/Header.js
import React, { Component } from 'react';
import { connect } from 'react-redux';

class Header extends Component {

  renderContent(){
    switch(this.props.auth) {
      case null: 
        return <div></div>
      case false: 
        return <a href="/auth/google">Login with Google</a>
      default: 
        return <a href="/api/logout">Logout</a>
    }   
  }

  render() {
    return (
    <div>
      <a href="/">Emaily</a>
      <ul>
        <li>
          {this.renderContent()}
        </li>
      </ul>
    </div>
    )
  }
}
function mapStateToProps({ auth }){
  return { auth }
export default connect(mapStateToProps)(Header);
```

### 15. Redirect the user on Auth
* add a route handler to the route /auth/google/callback 
* add a route handler to the route /api/logout to redirect user to /

```javascript
// /server/routes/authRoutes.js

  app.get("/auth/google/callback", 
  passport.authenticate("google"), //passport middle ware takes over here.
  (req, res) => { //after passpoprt does its thing we just use another route handler to redirect to whatever route is appropriate for our app.
    res.redirect('/surveys')
  }
  );
  
  app.get('/api/logout', (req, res) => {
  req.logout()
  res.redirect("/")
});
}
```

### 16. Separate the Landing Component into a new file 

*  In your App component delete the landing component that is currently there
*  create a new `Landing.js` file. and create a simple component. 

```jsx
// /server/client/src/components/Landing.js
import React from 'react';

const Landing = ({ textAlign: 'center'}) => {
  return (
    <div>
     <h1>I'm a Landing page</h1>
     Collect feedback from your users.
    </div>
  )
}

export default Landing;
```

* Import this into your app component. 


```jsx
// /server/client/src/components/
import Landing from './Landing';
```

### 17. Conditional Link on Header Logo

*  Make the header go to display `/` if logged out or to `/dashboard` if logged in by Usin Link instead of a tag.
*  Import the Link tag
*  replace the a tag with the Link tag
*  Tell the link tag where to redirect the user to when clicked by adding the `to` property and render the route
*  dependent on what the auth props resturns. 

```jsx 
// /server/client/src/components/Header.js
import { Link } from 'react-router';
// ...
class Header extends Component {
  
  renderContent(){
    // ...
  }

  render() {
    return (
    <div>
    <Link to={this.props.auth ? '/surveys' : '/'} 
      
    >Emaily</Link>
      <ul>
        <li>
          {this.renderContent()}
        </li>
      </ul>
    </div>
    )
  }
}
function mapStateToProps({ auth }){
  return { auth }
export default connect(mapStateToProps)(Header);
```

* At this point all should be well with the world 

# Setting Up Stripe With Node For Payments

### 1. Go to stripe.com and sign up to create an acoount. 

*  Create a new account for the app you are building. 
*  This is different than signing up. Ih happens after you sign up on the top left of the stripe dashboard
*  Click on developers on the left panel and click on API to reveal your keys.

### 2. Install React helper in your Client directory and put your keys in the right places

*  `npm install --save react-stripe-checkout`
*  Put keys in server. MAKE SURE KEY NAMES ARE SPELLED CORRECTLY 

```json
// /server/config/dev.js
module.exports = {  
  stripePublishableKey: "pk_test_Ieasdflkjas;ldfj;alsdjf;lasjdf",
  stripeSecretKey: "sk_test_al;sdjfoiwejflasjdl;kandvoiawjef"
}
```

```json
// /server/config/prod.js
module.exports ={
  stripePublishableKey: process.env.STRIPE_PUBLISHABLE_KEY,
  stripeSecretKey: process.env.STRIPE_SECRET_KEY
  }
```
  
### 3. Add environment keys to Heroku

* go to heroku dashboard click setting and reveal config variables. 
* add `STRIPE_PUBLISHABLE_KEY` and `STRIPE_SECRET_KEY`

### 4. Secret Keys on the React side using .env
* The client side only cares about the publishable key.
 With create react app our project can consume variables that were declared in the environment as if they were
 declared locally in our JS file. By Default we have `NODE_ENV` defined for us and any other Env variables starting with
 `REACT_APP_`
 
 * Create `.env.development`  and `.env.production` files in the root of our client directory
 * add your public key to both files like so
 
 `REACT_APP_STRIPE_KEY=pk_test_aksdfaoerkjasdf8aefoij`
* Test that you can get your key by putting a console.log at the end of your index.js component
 `console.log("STRIPE KEY IS ", process.env.REACT_APP_STRIPE_KEY)`
 you should be able to verify that you have access to you variables. 
 
 
 
### 5. Create a new component to configure react-stripe-checkout
*  Create a `Payments.js` file in your components directory
*  We are making our component that wraps the stripe-checkout component and pass it our own config options.



```jsx
// /server/client/components/Payments.js
import React, { Component } from 'react';
// Our component Payments just wraps StripeCheckout with our configuration options like amount token and our stripe key
import StripeCheckout from 'react-stripe-checkout';   

class Payments extends Component{
    render() {
        return (
        // amount is the amount in U.S. cents.
        // The token is expectin to receive a callback function
        // It will be called after we receive auth token from stripe and the token will be passed as an argument to the function
            <StripeCheckout
                amount={500}
                token={token => console.log(token)}
                stripeKey={process.env.REACT_APP_STRIPE_KEY}
            />
        )
    }
}

export default Payments;

```

### 6. Display our payments component

*  import our payments component
*  display our component if the user is logged in. 
*  In the default case of the switch statement. Add another li for the Payment component
*  Remove the fragments `<>` and wrap the li's in an array instead. 
*  Testing it should now display the Pay With Card button when logged in.

```jsx
// /server/client/components/Header.js
// ..
import Payments from './Payments'

class Header extends Component {

  renderContent() {
    switch(this.props.auth){
     // ..
      default: 
      return (
        [
          <li key="1"><Payments/></li>,
          <li key="2">
            <a href="/api/logout">Logout</a>
          </li>
        ]
      );
    }
    // ..
  }
  
  // ..

}
```
* You should now be able to console.log the token from earlier. Use a dummy card 4242 4242 4242 4242 to test transaction

### 7. Customize the text of the stripe payment pop up and the prebuilt button
*  In the payments component add more configuration to the StripeCheckout component
*  name gives the pop up a header 
*  description gives more information to the user of what they are paying for 

```jsx
// /server/client/components/Payments.js
// ..
      <StripeCheckout
          name="Emaily"
          description="$5 gets you 5 email credits."
          amount={500}
          token={token => console.log(token)}
          stripeKey={process.env.REACT_APP_STRIPE_KEY}
      />
```

*  To change the hiddeous Pay With Stripe button just pass a child component to the StripeCheckout Component.
```jsx
// /server/client/components/Payments.js
// ..
      <StripeCheckout
          // ..
      > <button>Add Credits</button> </StripeCheckout>
```

### 8. Take the token and send it to our Express server to follow up with Stripe.
*   Whenever we want to send something to our backend api with Redux we will have to send it using an action creator.
*   Create a new action. This action will async so use the same general type of action that we used for fetchUser

```javascript 
// /server/client/src/actions/index.js
//.

export const handleToken = (token) => async dispatch => {
//this is body of our real action creator. We don't have a route handler on the backend yet so lets make up the route.
// as the second argument pass the token that we got back from stripe.
  const res = await axios.post('/api/stripe', token)
  
// in the response back what type of action are we going to dispatch? 
// We are getting the same user model therefore we can use the same dispatch as before. 
dispatch({type: FETCH_USER, payload: res.data})
}
```

* Next we need to make sure our action creator gets called whenever we get a token from the StripeCheckout (Payments.js)
* import the connect helper and all of the action creators into the Payments.js component
* Use the props that we get from connect for the action  handleToken that we just built above. 

```jsx
// /server/client/src/components/Payments.js
import{ connect } from 'react-redux';
import * as actions from '../actions/index' ;

// .. 
//this is an attribute or prop of StripeCheckout
    token={token => this.props.hanldeToken(token)}
 
 // ..
 
 export default connect(null, actions)(Payments);

```

*  At this point we should be able to see the request being made on the XHR of the Nework tab in our dev tools.  

## 9. Set up route handler to handle /api/stripe

* Watch post request to /api/stripe
* Create a new file to handle payments and billing called `billingRoutes.js` in our routes folder.  
* put a `module.exports = (app) =>{}` inside of that of file with a `app.post('api/stripe')` handler 
* Jump to `index.js` and require the new route in the same place where we required authRoutes `require("./routes/authRoutes")(app)`

```javascript
// /server/routes/billingRoutes.js
module.exports = (app) =>{
  app.post('/api/stripe', (req, res) =>{
  //our token will be available in here. 
  })
}
```

```javascript
// /server/index.js
// ..
require("./routes/authRoutes")(app)
require("./routes/billingRoutes")(app)

//..
```
*  On the backend we are going to use some node middleware created specifically for stripe
*  Install stripe package on the server directory `npm install --save stripe` 
*  If you need to look at the documentation for stripe to charge a card you can find it at [Stripe API Reference](https://stripe.com/docs/api) 

*  Now we're going to set up the stripe library on our billingRoutes.js 
*  First import stripe from stripe. The stripe documentation shows that we need to call a second function on the require function that will take our secret key as an argument.
*  Becuase we need the key we will also need to require our `keys.js` 

```javascript
// /server/routes/billingRoutes.js
const keys = require('../config.keys');
const stripe = require('stripe')(keys.stripeSecretKey)
module.exports = (app) =>{
  app.post('/api/stripe', (req, res) =>{
  //our token will be available in here. 
  })
}
```
#### body parser
*  Install the `body-parser` npm package in your server directory  so that we can parse the info coming into to the post request from our front end. 
*  This will make our incoming request available under `req.body`
* `npm install --save body-parser` `
*  `const bodyParser = require("body-parser")` and `app.use(bodyParser.json())` with your other middlewares in your `index.js`
*  
```javascript
// /server/index.js
const bodyParser = require("body-parser")

// .. middlewares
app.use(bodyParser.json())
```
*  `console.log(req.body)` to make sure everything is coming in okay into /api/stripe

```javascript
  app.post('/api/stripe', (req, res) =>{
    console.log(req.body)
  })
```

* At this point if that console log spits out some card info we are ready to charge the card if not trouble shoot 
* Look for typos, in app.post, make sure you're using middlewares and that you installed all the packages. 

```json
// sample of console.log
{ id: 'tok_1EavJCFKo0M547C1brawQETC',
  object: 'token',
  card:
    { id: 'card_1EavJCFKo0M547C1NVVr6Crr',
      object: 'card',
      address_city: null,
      //..
    }
}
      
```

*  finish building route

```javascript
// /server/api/stripe.js
const keys = require("../config/keys");
const stripe = require("stripe")(keys.stripeSecretKey);

module.exports = (app) => {
    app.post("/api/stripe", (req, res) => {
        stripe.charges.create({
            amount: 500,  // amount we are charging
            currency: "usd",
            description: "5 email credits for $5", //can be anything
            source: req.body.id // this is our authorization from the front end.
        })
    });
};
```
### 9 Add credits to a users account (our backend)
*  Add a new credit attritube/property to our User model. Default it to 0

```javascript 
// /server/models.User.js
const userSchema = new Schema({
    googleId: String,
    credits: {type: Number, default: 0}
});
```

* Add .then new async await to get back  a charge object from our `/api/stripe` route that we we will use to add credits to our users account. 
* Here we I am console logging the charge and updating the user model we don't need to console log but you might want to do use the charge object for other for other configuration or validation
* Add 5 credit to the user using `req.user` supplied to us by passport and save the user
* Finally send back the user object. Notice we use a new user object equal to what mongo returns from .save() since req.user might now be stale. 
* Test by going to the network tab XHR and see if the user object comes back with some credits after going through the flow.  

```javascript 
// /server/api/stripe.js
   app.post("/api/stripe", async (req, res) => {
    // ..
        })
        console.log(charge)
        req.user.credits += 5
        const user = await req.user.save()
        res.send(user)
        
    });
```

### 10.  Add some validation to make sure that the user is logged in so no one hits this endpoing and goes through the flow with out being logged in. 
*  We will use route specific middleware 
*  Create a new `middlewares` directory at the server directory and add a file requireLogin.js
```javascript 
// /server/middlewares/requireLogin.js

module.exports = (req, res, next) => {
  if (!req.user) {
    // if the user is not logged in stop the process
    return res.status(401).send({ error: "You must log in!" });
  }

  next(); // if the user is logged in just go to the next middleware
};
```

*  require the middleware in the route that we want to use it. In our case for now we only want to use it in the `/api/stripe` route

```javascript 
// /server/api/stripe.js
const requireLogin = require('../middlewares/requireLogin')

module.exports = (app) => {
    app.post("/api/stripe", requireLogin, async (req, res) => {
        const charge = await stripe.charges.create({
            amount: 500,
            // ..
        
    });
```

### 11. Displaying and updating the credits

* Add another `<li>` in our Header component 
* `this.props.auth` is available in the header so all we have to use is `this.props.auth.credits` to display the credits.

```javascript
class Header extends Component {

  renderContent() {
    switch(this.props.auth){
      // ..
      default: 
      return (
        [
          <li key="1"><Payments/></li>,
          <li key="2"> Credits: {this.props.auth.credits}</li>,
          <li key="3">
            <a href="/api/logout">Logout</a>
          </li>
        ]
      );
    }
  }
  // ..
}

```



