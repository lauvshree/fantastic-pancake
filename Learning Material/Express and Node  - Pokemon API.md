## Lecture ✍️

#### Prerequisites
You should be comfortable with creating a web API with end points 

#### Pokemon API

We developed a web API with two end points in our previous exercises. Now we will tweek our code to see how we can make our own API which will return an in-memory representation of Pokemon just like what you got from the pokemon API (https://pokeapi.co/api/v2/pokemon)

In our code, we will include a pokemon obejct collection and create a new end point to return this array.

```js
const pokemon = [{
    id : 1,
    name : 'bulbsaur'
},
{
    id : 2,
    name : 'ivysaur'
},
{
    id : 3,
    name : 'venusaur'
}]

```
And we will also include the end point to retrieve the array.

```js
app.get("/pokemon", (req,res) => {
    return res.send(pokemon);
});
```

Let's go further and add another endpoint for it to return that one pokemon whose details we want. My end-point to retrieve pokemon 1 would look like *http://localhost:3333/pokemon/1*. But depending on which pokemon we want to retrive the id will vary. So we should make an end-point which can take variable. 

```js
app.get("/pokemon/:id", (req,res) => {
    //return details of just that pokemon whose id is passed
});
```

How do we retrieve the id that is passed? This can be done through the *req* (httpRequest) object. All of the parameters passed from the client are available through the req object as a dictionary called *params*. We can retrieve the parameters that we want from this dictionary. 

```js
app.get("/pokemon/:id", (req,res) => {
    const id = req.params.id;
    //return details of just that pokemon whose id is passed
});
```

You see how we retrieve the *id* from the params and store it in the local variable with the same name, so we can retrieve the pokemon later with that id. There is a nicer way to do this. It is called *object destructuring*.
This is very useful if you want to retain the same names for the variables as the parameters passed. In our case, we want to call the id paramater as id in our method scope. 

```js
app.get("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    //return details of just that pokemon whose id is passed
});
```

Object destructuring is most useful when you have to retrieve multiple parameters from the request object. We will look at this in detail later.
Based on this id, we have to retrieve the Pokemon from the collection we have. One thing we should keep in mind is that, all the parameters we get from the request object are only available as strings. We have to convert it into the data type we prefer, before using it. In this case, id will be returned as a String. But id in the pokemon object is an int. So, before we look for the object with the same id in the list, we should convert it to int type and then find the pokemon object that matches the id. We use the *find* method which is available in the Java script collections object, which takes a callback as parameter to retrieve the object.

```js
app.get("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    //Find the pokemon object which matches the id and return it in response.
    return res.send(pokemon.find((p) => p.id === parseInt(id)));
});
```

Now in the browser when you when you access the end-point [http://localhost:3333/pokemon/1] it will retrieve the pokemon object with 1 for you. 

But when you try you retrieve a non-existent id, it just returns an empty page. This is because if *find* doesn't find an object matching, it returns null. Ideally, we should be throwing an appropriate error message when a non-existent id is being accessed. In addition we can also send error codes to let the caller know the request was unsuccessful. More on response codes [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

```js
app.get("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    //Find the pokemon object which matches the id
    const mypoke = pokemon.find((p) => p.id === parseInt(id));
    if(!mypoke) { //if poke is not a valid object
        //Send error message as response along with 404 error code
        return res.status(404).send(`Pokemon with id ${id} not found`);
    } else {
        return res.send(mypoke);
    }
});
```

We have now successfuly retrieved all the pokemons and a specific pokemon we have. In CRUD (Create, Retrieve, Update, Delete), we have done the *R*. Now let's see how we can add a pokemon object. The *C* in CRUD. For this, we use a post request. But let's look at the things we need to do. 

### To-Do list for POST request
* Grab the values from the request object
* Validate the values and ensure they have a id and name
* Create a new Pokemon Object
* Insert the new pokemon object into the existing array
* Return the pokemon object or a relevant message saying the pokemon object has been added. 

The body cannot be passed to the API just like that. The web application needs to know how to handle the body that is passed in the request object. To achieve this, we need to let the application use a [middleware](https://expressjs.com/en/api.html#express.json).

```js
//Enable the web application to take JSON formatted input from client.
app.use(express.json())
```

Now let's include the code to handle all of the things we discussed in our to-do list above.

```js
app.post("/pokemon", (req,res) => {
    //Using object destructuring to store the parameters in local variables
    const {id,name} = req.body;
    if((!id) || (!name)) {
        res.status(400).send("Both id and name have to be set");
    } else if (!parseInt(id) || name.length < 3){ //if the id is passed but is not numeric or name is too short
        res.status(400).send("Data is invalid");
    } else if (pokemon.find((p) => p.id === parseInt(id))) {// pokemon with that id exists
        res.status(400).send("Pokemon with this id already exists");
    } else {
        const poke = {id: parseInt(id),name: name}
        pokemon.push(poke);
        return res.send(poke);
    }
});
```

We will use (postman)[https://www.getpostman.com/] to post the new pokemon. To post data from postman, we enter the endpoint and select post method in the postman GUI. In the body, we can send the JSON formatted input to the server. 
```js
{
        "id": "6",
        "name" : "charizard"
}
```

The above code will work perfectly for our purpose. You can try to post the new pokemon, as above and try to retrieve it. But look at the amount of validation we have had to do for just two parameters. Imagine a scenario where we have more parameters. The situation only becomes more cumbersome and the code would be less readable and confusing. JS offers a module for validating the data. For all the joy it brings, the module is rightly name *Joi*.

`npm install --save joi`

We will now include joi in our server application, to validate the post data.

```js
const Joi = require('joi')//Note that Joi starts with 'J' as it refers to a class.
```

We will now use Joi for validating our input by defining a schema and applying the schema to validate what is being posted.

```js
/*
We define our schema to ensure that name is a string of length 3 or more characters and it is definitely passed and id is an int which also is definitely passed.
*/
app.post("/pokemon", (req,res) => {
    const {id,name} = req.body;
    const schema = {
        id : Joi.number().required(),
        name : Joi.string().min(3).required()
    }
    const poke = {id: parseInt(id),name: name}
    const valid = Joi.validate(poke,schema)
    if(valid.error) { //If the validation throws an error
        res.status(400).send(valid.error.details[0].message);
    } else if (pokemon.find((p) => p.id === parseInt(id))) {// pokemon with that id exists
            res.status(400).send("Pokemon with this id already exists");
    } else {
        pokemon.push(poke);
        return res.send(poke);
    }
});

```

Repeat the POST request through postman and see how the web application serves the API request. 

### The final code
```js
//This will search for the module express in the node-modules and import it in for use in this appliction

const express = require('express');
const Joi = require('joi');

//Create an app which an instance of express
const app = new express();

/*  Take a port on which you want the server to listen to from the environment
    variables if there is one else use 3333
    The server is your own system - localhost or 127.0.0.1*/

const port = process.env.PORT || 3333;

/*  Set up new end point to the default /. 
    This will run the callback with request and response parameters mentioned 
    when the user accesses the path '/'*/

const pokemon = [{
    id : 1,
    name : 'bulbsaur'
},
{
    id : 2,
    name : 'ivysaur'
},
{
    id : 3,
    name : 'venusaur'
}];

app.use(express.json());

app.get("/", (req,res) => {
    return res.send("Hello World!");
});

app.get("/test", (req,res) => {
    return res.send("From inside test!");
});

app.get("/pokemon", (req,res) => {
    return res.send(pokemon);
});

app.get("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    //Find the pokemon object which matches the id
    const mypoke = pokemon.find((p) => p.id === parseInt(id));
    if(!mypoke) { //if poke is not a valid object
        return res.status(404).send(`Pokemon with id ${id} not found`);
    }
    return res.send(mypoke);
});

app.post("/pokemon", (req,res) => {
    const {id,name} = req.body;
    const schema = {
        id : Joi.number().required(),
        name : Joi.string().min(3).required()
    }
    const poke = {id: parseInt(id),name: name}
    const valid = Joi.validate(poke,schema)
    if(valid.error) { //If the validation throws an error
        res.status(400).send(valid.error.details[0].message);
    } else if (pokemon.find((p) => p.id === parseInt(id))) {// pokemon with that id exists
            res.status(400).send("Pokemon with this id already exists");
    } else {
        pokemon.push(poke);
        return res.send(poke);
    }
});

app.listen(port, () => {
    console.log(`listening at http://localhost:${port}`)
})
```

### Challenge

* Create Web API end point for put request which will take an id and modify the name of the pokemon. Include necessary validations.
* Create Web API end point for delete request which will take an id and delete the pokemon. Include necessary validations.
* Separate the validation logic to a separate function for reusability




