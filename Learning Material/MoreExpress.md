### More on express

## Making your own middleware

In express apps, middlewares are used by the express app. These are call back methods which take 3 parameters. `function (req,res,next)`. Some middleware we have used include, cors(), express.json(),express.static("dirname") etc., Let's try to make our own middleware for our server. All the requests to the server will be routed through this middleware.

```
const express = require('express');
const cors = require('cors');
const path = require('path');
const app = new express();

app.use(express.json())
app.use(cors());


const users = [
    {
        email:'luca@gmail.com',
        firstname:'luca',
        lastname:'coote'
    },
    {
        email:'sam@gmail.com',
        firstname:'sam',
        lastname:'collins'
    }
];

const process_requests = (req, res, next) => {
    console.log("New request received at "+Date());
    if(req.url.endsWith(".css") || req.url.endsWith(".png") || 
    req.url.endsWith(".jpg") || req.url.endsWith(".html")) {
        //stream the requested file
        res.sendFile(path.join(__dirname + req.url));    
    } else {
        next();
    }
}

const getUsers = (req,res) => {
    res.status(200).send(users);
}

app.use('/',process_requests);

app.get('/users',getUsers)

app.get('/myjs.js',(req,res)=>{
    res.sendFile(path.join(__dirname + '/myjs.js'));
})

app.listen(3333, () => {
    console.log(`listening at http://localhost:3333`)
})

```

What this does is provides for all the css files, html files, png files and jpg files as static files. All other requests made to the server are routed through next() and looks for a mapping end-point which can process the request. 


## CRUD is Create Retrieve Update and Delete 

We have in the past managed to post data to the server and get data from the server. We will now look at how to send PUT and DELETE request to the server.

### PUT Request to update the data on the server

```
const express = require('express');
const cors = require('cors');
const Joi = require('joi');

const users = [
    {
        email:'luca@gmail.com',
        firstname:'luca',
        lastname:'coote'
    },
    {
        email:'sam@gmail.com',
        firstname:'sam',
        lastname:'collins'
    }
];

const app = new express();

app.use(express.json());
app.use(express.urlencoded());
app.use(cors());

const firstname_schema = Joi.string().min(3);
const lastname_schema = Joi.string().min(2);

const modifyUser = (request,response) => {
    //Get the values set in the request body
    const {email, firstname, lastname} = request.body;

    if(!email) {
        response.status(400).send("Email id is required");
    } else {
        userObj = users.find((user) => user.email == email);

        if(firstname) {
            const valid = Joi.validate(firstname,firstname_schema)
            if(!valid) {
                response.status(400).send(valid.error.details[0].message);
            } else {
                userObj.firstname = firstname;
            }
        }
        if(lastname) {
            const valid = Joi.validate(lastname,lastname_schema)
            if(!valid) {
                response.status(400).send(valid.error.details[0].message);
            } else {
                userObj.lastname = lastname;
            }
        }
        response.status(400).send(userObj);    
    }

}

const getUser = (request,response) => {

    const {email} = request.query;
    if(!email) {
        response.status(400).send("Email id is required");
    } else {
        userObj = users.find((user) => user.email == email);
        if(!userObj) {
            response.status(400).send("No such user found");
        } else {
            response.status(200).send(userObj);
        }
    }
};

app.put("/modify",modifyUser);
app.get("/user",getUser);


app.listen(3333, () => {
    console.log(`listening at http://localhost:3333`)
})
```
### DELETE request to delete data on the server

```
const express = require('express');
const cors = require('cors');

const users = [
    {
        email:'luca@gmail.com',
        firstname:'luca',
        lastname:'coote'
    },
    {
        email:'sam@gmail.com',
        firstname:'sam',
        lastname:'collins'
    }
];

const app = new express();

app.use(express.json());
app.use(express.urlencoded());
app.use(cors());

const deleteUser = (request,response) => {
    const {email} = request.query;

    if(!email) {
        response.status(400).send("Email id is required");
    } else {
        userObj = users.find((user) => user.email == email);
        if(!userObj) {
            response.status(400).send("No such user found");
        } else {
            users.pop(userObj);
            response.status(400).send("Deleted\n"+userObj.email);    
        }
    }

}

const getUser = (request,response) => {

    const {email} = request.query;
    if(!email) {
        response.status(400).send("Email id is required");
    } else {
        userObj = users.find((user) => user.email == email);
        if(!userObj) {
            response.status(400).send("No such user found");
        } else {
            response.status(200).send(userObj);
        }
    }
};

app.delete("/delete",deleteUser);
app.get("/user",getUser);


app.listen(3333, () => {
    console.log(`listening at http://localhost:3333`)
})

```

