## Lecture ✍️

### What is Mongo DB
Mongo DB is what we will use for persistence in our MERN full-stack development exercise. Mongo is the 'M' of 'MERN'. Persistence is storing the data in some manner, so that the data is accessible even if the application is shutdown and restarted or across multiple instances of applications. You may recollect from the previous API exercises that the Pokemons we created through POST request don't exist when we restarted the web server. This is because, they were not persisted or stored. The storage of data electronically is called *Database*. Mongo will now serve as our data or persistence layer. Mongo DB is a document database or NoSQL database. Rather than storing the data across tables in a relational database, all the data will be stored in documents which are just text files with data represented as JSON.

#### Prerequisites
* Install (Mongo DB)[https://docs.mongodb.com/manual/administration/install-community/]
* You should be comfortable with CLI and JSON.

### Accessing Mongo DB
We will now create an instance of Mongo DB in our system and access it through a CLI. To access Mongo DB, we will need to run two terminals; one will run the Mongo DB server and the other for us to run the Mongo CLI.
The Mongo CLI is the client which will send requests to Mongo DB server, which will respond with a JSON file. It is very similar to the client-server architecture we saw in the case of Web server and Web API. 

## A comparison of Relational DBMS and Mongo DB
<table>
    <tr>
        <th>Relational Database</th>
        <th>Mongo DB</th>
    </tr>
    <tr>
        <td>Database</td>
        <td>Database</td>
    </tr>
        <td>Tables</td>
        <td>Collection</td>
    <tr>
        <td>Rows/Records</td>
        <td>Documents</td>
    </tr>
    <tr>
        <td>Columns</td>
        <td>Fields</td>
    </tr>
</table>

As mentioned in the table above, a record in a relational Database, is represented as a document in Mongo DB. Documents in MongoDB are BSON, which is a binary data format that is like JSON, but includes additional type data.

We will start Mongo DB as a service from one terminal and connect to the DB from another terminal. The instructions here are for Mac OS. If you are using windows or other OS, please visit the Mondo DB site given above and follow the instructions specific to your OS.

### Run Mongo DB
From a terminal issue the following command to start the Mongo DB service

```
brew services start mongodb-community@4.0
```

### Connect and use Mongo DB
To connect to the Mongo DB, as mentioned we will open a new shell and issue the following command. 

```
mongo
```

You will now see the prompt, where we can start issuing commands. We will issue the following command to list all the default databases that are present.

```
show dbs
```

This should list 3 databases namely *admin, config and local*.
We will now create a new database for keeping all the pokemon data. Let's name it *pokedex*.

```
use pokedex
```

This will create a DB name Pokedex, if it doesn't exist and switch to that DB. But the DB will not be listed in the *show dbs* until the database has some data because, as mentioned before, in Mongo the database is in the form of a file. The file is not created until the collections are created. 

Let us create a collection called **pokemon** in our database. The current database is referred to as **db**. Since *use pokedex* switched to the pokedex database, **db** will now refer to **pokedex** in our CLI's context.

```
db.createCollection('pokemon')
```

Now that we have created our collection, the *show dbs* should list our database. Check to see if that's the case. To see all the collections in your database we can use the following command.

```
show collections
```

This should list *pokemon* the collection we created in our database earlier. Once you see it, you are all set to explore CRUD with Mongo DB.


## CRUD Operations

### Create
To add data to our collection, we use the insert command. We start inserting the data using the following command. 

```
db.pokemon.insert({
```

You will see that pressing *enter* after issuing the command takes you to the next line preceded with **...** . This means that the CLI is waiting for data to be entered. Like in Javascript, the end of a command is assumed when there are closing brackets matching the opening ones. Let type the data we want to insert and finish it with the appropriate brackets.

```
    id:1,
    name:'bulbsaur',
    height:'6 cm'
})
```

You should get *WriteResult({ "nInserted" : 1 })*, which indicates that the data was successfully inserted. 0 would indicate failure in inserting the data. 

### Retrieve
Let's now retrieve all the data in the pokemon collection. 

```
db.pokemon.find()
```
You will see when the data is retreived that the first item in the document retrieved is something like this - *"_id" : ObjectId("5ce2022ca19fe24218a7831f")*. This is because MongoDB maintains a unique key for all the documents. If you want to use the id you are providing as this unique key, you may do so by prefixing '_' for the id. 

```
db.pokemon.insert({
    _id:1,
    name:'bulbsaur',
    height:'6 cm'
})
```

To retrieve the data in a more readable format we can use the **pretty** method on the output of find.

```
db.pokemon.find().pretty()
```

In this case, we inserted just one pokemon data. What if we want to enter more than one. We can either run the above command multiple times or pass the data as an array. 

```
db.pokemon.insert([{id:2,name:'ivusaur',height:'99.1 cm'},{id:3,name:'venusaur',height:'2.01 m'}])
```

The array is indicated in the above command with square brackets; each unit of data is within curly brackets, {*data here*} and comma separated from each other. The output will indicate that there was a bulk insert attempted and will also tell the user how many units of data were added.

Now when you do a *db.pokemon.find()* you should see all the three pokemons listed. You can also make other keys unique or combination of keys by creating index. It is not within the scope of this document. You can refer to [https://docs.mongodb.com/master/crud/]


#### Task 
Insert 5 more pokemons with their respective ids using single insert or bulk insert and retrieve them all with the find command.

In all of the above cases, we have retrieved all the data in the pokemon collection. We will now look at how to retrieve just one record or document from the collection based on conditions. Let's retrieve the pokemon whose id is 3.

```
db.pokemon.find({id:3})
```

We can retrieve the data based on any of the fields in the document. 

```
db.pokemon.find({name:'ivysaur'})
db.pokemon.find({height:'2.01 m'})
```

We can also retrieve multiple documents based on a condition. Let's retrieve all the documents where the name of the pokemon contains "saur" (which is all the pokemons. Try any string you want :smile:)

```
db.pokemon.find({name: /saur/})
```
Or if you want to retrieve the first one which matches the condition amongst many such documents which match, we use *findOne*.

```
db.pokemon.findOne({name: /saur/})
```

We can also use $eq,$gt,$lt to search numeric values based on conditions. 

### Tasks

Can you find out what each of these commands do? What will they return?

```
db.pokemon.find( { id: { $eq: 5 } } )

db.pokemon.find( { id: { $lt: 4 } } )

db.pokemon.find( { id: { $gt: 2 } } )

```

### Update
Now that we are familiar with C and R of the **CRUD**, let's look at U. To update a document, we need to finrst identify a document based on a condition and then update the document. So update in Mongo DB takes two parameters; first one to identify the document to be updated and the second one which has the changes. 
Let's change the document which has id as 1 to have a different name. 

```
db.pokemon.update({id:1},{$set:{name:'diffsaur'}})
```

This command identifies the document with id 1 and changes the name to diffsaur. Let's do a *db.pokemon.find()* to see the records in the collection to ensure our changes have happened. Update will by default only update the first document that is matched. If you want to update multiple documents at once, we have to specify the third and fourth boolean parameters which are for 
* **upsert** - This is a combination of update and insert (true/false - if this should be an “upsert” operation; that is, if the record(s) do not exist, insert one. Else update the existing one. Upsert by default is true and only inserts/updates a single document) and 
* **multiple** (true/false - indicates if all documents matching criteria should be updated rather than just one.)

Let's try to make an update to a non-existent document.

```
db.pokemon.update({id:19},{$set:{name:'newsaur',height:'5 m'}})
```

This command when run will not update any document as there is no document with id 19. You can verify the same running *db.pokemon.find()*

If we run the same command with *upsert=true* option, it will see that there is no matching document which it can update and will insert a new one. 

```
db.pokemon.update({id:19},{$set:{name:'newsaur',height:'5 m'}},{upsert:true})
```

Now what if we decide to have an additional field for moves for all the pokemon? We can add a column to the existing documents with the following command.

```
db.pokemon.update({},{$set:{"moves":''}},false,true)
```
Here the '{}' implies there is not conditions or restrictions and all the documents need to be updated. 

### Delete

Now that we know how to retrieve all documents and also selective documents based on a criteria, deleted the documents is very simple. The parameters for deletion are the same as find.

```
db.pokemon.remove({name: /diff/})
```

This will retrieve all records where the name contains the word 'diff' and delete them. Remove has to be handled diligently. *db.pokemon.remove({})* will remove all the records in the collection.

### Nested queries
Mongo DB is not meant for relational purpose. For that we have RDBMS. However, Mongo DB supports nested querying. Just to show case let us create another collection, which has the moves and insert some moves into it. 

```
db.createCollection('moves')

db.moves.insert([{ _id:1, name:'pound'}, { _id :2, name:'karate chop'},{_id :3, name:'double slap'}])
```

We will now modify the pokemon collection to include moves.

```
db.pokemon.update({id:1},{$set:{moves:[2,3]}})

db.pokemon.update({id:2},{$set:{moves:[1,3]}})

db.pokemon.update({id:3},{$set:{moves:[1,2]}})
```

Now we have two collections. One with the pokemon details where the moves have just the ids and the other with the moved where the id of the move and the name of the move are stored. To get all the pokemons which have the move 'karate chop', we will issue a nested query like this.

```
db.pokemon.find({moves: db.moves.findOne({'name': 'karate chop'})._id}).pretty()
```
But as mentioned before, this defeats the purpose of NoSQL and is not ideal. 

### Populating DB from JSON

Inserting data into Mongo DB through a CLI can be very tedious. We can however import the documents from a json array stored in a file. Let's import the data from [pokemon.json](./pokemon.json). We will quit the mongo CLI using the `quit()` command. We will traverse to the directory where we store package.json in the local system. We will first remove all the current documents in the pokemon collection with `db.pokemon.remove({})` and add the data from the file into that collection.

```
mongoimport --jsonArray --db pokedex --collection pokemon --file pokemon.json
```

Here we are mentioning the data is a jsonArray which we are importing in the database named pokedex inside a collection named pokemon from a file name *pokemon.json*. Now let's go back into Mongo CLI and see if the data has been imported.

```
use pokedex

db.pokemon.find().pretty()
```

This will list all the pokemon from the collection (which was imported from the file).
