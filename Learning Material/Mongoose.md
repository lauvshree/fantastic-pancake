## What is Mongoose 
Mongoose is an ORM for use in JS web applications. ORM stands for Object-relational mapping. It is a mechanism that makes it possible to address, access and manipulate objects without having to consider how those objects relate to their data sources, in this case Mongo DB.

### Prerequisites
* Mongo DB should be installed in your system and you must be comfortable working with the Mongo CLI to access the data on the Mongo DB
* Have a pokedex DB with pokemon collection which has some sample data to start with. For eg., *[{id:1,name:'bulbasaur'},{id:2,name:'ivysaur'}]*
* You should know to create web APIs with Express and Node JS
* Make sure the Mongo DB server is running

### Connecting to Mongo DB from your web application
In our earlier exercises we created web application which served end points returning the pokemon and also to create pokemon. But these were all transient/in-memory and lasted only as long as the application was running. To ensure that the data is persisted and is available to use for multiple instances of applications (same or different), we will use Mongo DB. The client will access the APIs served by the web application which will in-turn communicate with the database. 

![alt text](MongoFullSystem.png "Client Server Visualisation")

Let's look at the logical flow of control:
* The browser loads up a web page.
* The web page has a script from which we are making a fetch request to the localhost over port 3333 (Or whatever port your server is running on) asking for the /pokemon endpoint.
* The node and express layer services that request and calls the appropriate call back.
* Within the callback we will use mongoose which will inturn connect to the Mongo DB and get the documents from the Mongo DB. 
* The node and express layer processes this and sends it to the client.

### Installing Mongoose package
To install mongoose package we will go to the terminal and issue the command `npm i mongoose`. This will install the mongoose module in your system for use in your web application. 

Now we will go back to our api.js from few light years back, and make changes to use the mongoose module. Add the line `constant mongoose = require('mongoose');` to access mongoose.

In api.js we will now add lines to connect to the db. The port on which the Mongo server runs by default is 27017. If it is any different on your system run `lsof -i | grep mongo` to see the port on which it is listening. If the command returns nothing, the chances are that the server is not running.

```
mongoose.connect('mongodb://localhost:27017/pokedex')
```

Being an ORM, Mongoose completely abstracts the actual tedious communication with database through simple APIs it provides. The collections in the database are all represented as model in the ORM. So all the operations we do, we do on the model which inturn takes care of working on the actual DB. We will now define a model which will reflect the pokemon collection and then use it in our api.js. For this in the parent folder of api.js, we will create a folder named *models* where we will create all the models. For now, we need one model which will reflect our pokemon collection. Let's name it *pokemon.js*

```
//We will require the mongoose module to create a model

const mongoose = require('mongoose');

//We will define the schema of the pokemon collection and export it.

const pokemonSchema = new mongoose.Schema ({
	id : Number,
	name : String
	}, { collection: 'pokemon' });

module.exports = mongoose.model('Pokemon',pokemonSchema); 

```

In `mongoose.model('Pokemon',pokemonSchema)`, the first argument is the singular name of the collection your model reprsents. Mongoose automatically looks for the plural, lowercased version of your model name in the database. Thus, for in our case, the model Pokemon is for the pokemons collection in the database. But since we referred to our collection as *pokemon*, the plural for pokemon being pokemon, we have explicitly mentioned the collection name above.

Now back in our *api.js* we have to use the model in the our get request instead of the in-memory objects we were returning. We will replace the method serving the end-point with the following code.

```
const Pokemon = require('./models/pokemon');

app.get("/pokemon", (req,res) => {
    Pokemon.find({}).then(docs => {
        return res.send(docs);
    });
    
});

```

In the above code `Pokemon.find({})` will send a request to the DB server to fetch all the pokemon in the collection. In our case, the server is running locally. But in real life applications, that may not be the case. Our web application could be communicating with a DB server in a remote location. And there might be numerous web applications trying to connect with the same DB server at a given point of time. If we have to wait on the response, it will take unacceptably long. So what we make is an asynchronous request which returns a *promise*. We then give a *callback* which will be invoked as and when the response is received from the server. In the callback, we return the response to the client. Run your web server with `npm start` and access the end point to see what is retrieved. 

For your reference, the api.js will look like this.

```
const express = require('express');
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/pokedex');

const app = new express();
app.use(express.json());
const port = process.env.PORT || 3333;

const Pokemon = require('./models/pokemon');

app.get("/pokemon", (req,res) => {
    Pokemon.find({}).then(docs => {
        return res.send(docs);
    });
    
});

//add more api end points here

app.listen(port, () => {
    console.log(`listening at http://localhost:${port}`)
})
```

Now let's change the api end point to get one pokemon based on the id. We are now going to get it from the database. We will use the findOne method to do the same. As we did in our previous examples, we will use object destructuring to get the id and then pass the id to findOne. You may recollect from our previous examples that since the name of the attribute is the same as the variable name, we don't have to explicitly say `{id:id}`. As before, if there is a pokemon matching the id passed through the request, we will return that pokemon, else we will return an error message.

```
app.get("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    Pokemon.findOne({id}).then(doc=> {
        if(doc != null) {
            return res.send(doc);
        } else {
            return res.status(404).send(`Pokemon with id ${id} not found`);
        }
    })
});

```

Retrieve a specific pokemon based on the id and check if the you get the desired output - If the id is valid, you should get the pokemon else you should get error message.

Now let's serve the post request through an endpoint and add the pokemon to our database. We can validate the data and check that there is no pokemon with same id in the database and then we can create an in-memory pokemon object with the id and name passed through the request object and then save the object in the DB. 

```
app.post("/pokemon", (req,res) => {
    const {id,name} = req.body;
    const poke = {id,name};
    const schema = {
        id : Joi.number().required(),
        name : Joi.string().min(3).required()
    }
    const valid = Joi.validate(poke,schema)
    if(valid.error) { //If the validation throws an error
        res.status(400).send(valid.error.details[0].message);
    } else {
        Pokemon.findOne({id}).then(doc => 
        {// pokemon with that id exists
            if(doc == null) {
                //Create the Pokemon Object
                const pokemon = new Pokemon({id,name});
                pokemon.save().then(doc =>{
                    res.send(doc);
                });
            } else {
                res.status(400).send("Pokemon with this id already exists");
            }
        });
    }
});

```
As you may see above, we first check if it already exists in the DB and then insert. As both are sent as fetch requests which return promises, we have a promise within a promise, each with its own callback. It might be overwhelming initially to get a hang of it but will eventually get simpler with practice. 

Now to update the documents that already exist in the database, we will make changes to the *put* end point created, as a part of additional tasks. We will make use of the `updateOne` method on the model which takes two parameters, the query string and the object to be updated. It returns a promise. What you get from the server is an object which indicates the number of records which matched the query, if the update operation ran successfully and the number of records that were updated. 

```
app.put("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    const {name} = req.body;
    const poke = {id,name};
    const schema = {
        id: Joi.number().required(),
        name : Joi.string().min(3).required()
    }
    const valid = Joi.validate(poke,schema)
    if(valid.error) { //If the validation throws an error
        res.status(400).send(valid.error.details[0].message);
    } else {
        Pokemon.updateOne({id},poke).then(doc => 
        {
            if(doc.n == 0) {
                res.send("No Matching Records were found");
            } else if (doc.ok == "1" && doc.nModified == "1") {
                Pokemon.findOne({id}).then(doc => 
                    {       
                        res.send(doc);
                    });
            } else {
                res.send("No changes seemd to have been made");
            }
        });
    }
});

```

Now what we are left with trying out is deletion of the documents from the database. 

```
app.delete("/pokemon/:id", (req,res) => {
    const {id} = req.params;
    Pokemon.deleteOne({id}).then(doc => 
    {
        if(doc.n == "0") {
            res.send(`No documents with id ${id} could be found`);
        } else if (doc.ok == "1" && doc.deletedCount == "1") {
            res.send(`Pokemon with id ${id} has been deleted`);
        }
    });
});

```
We have successully implemented APIs for CRUD with Mongoose ORM. 

### Tasks
Implement all of the CRUD operations with additional fields in each of the collection.