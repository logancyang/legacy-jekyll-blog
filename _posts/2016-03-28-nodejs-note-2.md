---
layout: post
title: "Server-side Development Using NodeJS (2)"
comments: True
---

This is my note for the NodeJS class offered by HKUST on Coursera.

## Express Generator

- Scaffold out an Express application, build a REST API server.
- Install express-generator

```
 npm install express-generator -g
```

- Scaffolding an Express Application

```
express node-express-gen
cd node-express-gen
npm install
```

- Start the Express server

```
npm start
```

- Copy the `dishRouter.js`, `promoRouter.js` and `leaderRouter.js` to 
the routes folder within the Express application
- Copy the `index.html` and `aboutus.html` file from the `node-express/public` 
folder to the `public` folder in the new project.
- Open the `app.js` file and then update the code

```javascript
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var routes = require('./routes/index');
var users = require('./routes/users');
var dishRouter = require('./routes/dish-router');
var promoRouter = require('./routes/promo-router');
var leaderRouter = require('./routes/leader-router');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));

app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());

app.use(express.static(path.join(__dirname, 'public')));

app.use('/', routes);
app.use('/users', users);
app.use('/dishes',dishRouter);
app.use('/promotions',promoRouter);
app.use('/leadership',leaderRouter);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers
// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});

module.exports = app;
```

- Save the changes and run the server. You can then test the server by 
sending requests and observing the behavior.

## Intro to MongoDB

- NoSQL 4 broad categories
  - Document DB, e.g. MongoDB
  - Key-value DB, e.g. Redis
  - Column-family DB, e.g. Cassandra
  - Graph DB, e.g. Neo4J
- Document: A self-contained piece of info, e.g. JSON document
- Collection: Collection of documents
- Database: A set of collections

```
Database -> Collection -> Doc (JSON)
```

- Why NoSQL
  - Scalability: Availability, Consistency, Partition tolerance
  - Ease of deployment: No object-relation mapping required
- Document DB
  - Server can support multiple databases
  - A database: a set of collections
  - A collection: a set of docs
  - Document is JSON document with some additional features
- MongoDB Format
  - BSON: Binary JSON format
  - Supports length prefix on each value, easy to skip over a field
  - Info about the type of a field value
  - Additional primitive types not supported by raw JSON like UTC date time,
  raw binary, and ObjectId
- MongoDB ObjectId
  - Every doc in MongoDB must have a unique `_id` field
  - Default ObjectId created by MongoDB when you insert a doc

```
{
    "_id": ObjectId("somehash"),
    "name": "Uthapizza",
    "description": "Test"
}
```

- ObjectId is a 12 byte field: Timestamp (4), Machine ID (3), Proc ID (2),
Increment (3)
- `id.getTimestamp()` returns the timestamp in ISO time format
- Downloading and installing MongoDB

```
brew install mongodb --with-openssl
```

- Starting the server and interacting with it using Mongo REPL shell
  - Create a `mongodb` folder and a `data` folder inside it.

```
cd mongodb
mongod --dbpath=data
```
This will start the mongodb server.

- In another cmd window,

```
// Start MongoDB REPL
mongodb
// Print db name
> db
test
// Switch DB
> use conFusion
switched to db conFusion
> db.find()
...
> db.help()
...
> db
conFusion
// Insert entry
> db.dishes.insert({name: "Uthapizza", description: "Test"});
WriteResult({"nInserted": 1})
> db.dishes.find();
{"_id": ObjectId("..."), "name": ..., "description": ...}
> db.dishes.find().pretty()
{
    ...
}
> var id = new ObjectId()
> id
ObjectId("...")
> id.getTimestamp()
ISODate("...")
```

## Node and MongoDB

- Node MongoDB driver
  - A high-level API for a Node application to interact with MongoDB server
  - Installation: `npm install mongodb --save` inside the project folder
- The Node MongoDB driver supports several operations that can be performed from
a Node application
  - Connecting to a MongoDB
  - Insert, delete, update, query docs
- Supports both callback based and promise based interactions
- A simple Node-MongoDB Application

```
npm install mongodb --save
npm install assert --save
```

#### Exercise I

simple-server.js

```javascript
var MongoClient = require('mongodb').MongoClient
var assert = require('assert');

// Connection URL
var url = 'mongodb://localhost:27017/conFusion';
// Use connect method to connect to the Server, with callback func
MongoClient.connect(url, function (err, db){
  assert.equal(err, null);
  console.log("Connected to the server");

  var collection = db.collection("dishes");
  // Insert doc into db
  collection.insertOne({name: "Uthapizza", description: "test"},
    // Supply callback func for later available result
    function (err, result) {
      assert.equal(err, null);
      console.log("After insert:");
      console.log(result.ops);
      // Supply callback func for later available result
      collection.find({}).toArray(function (err, docs) {
        assert.equal(err, null);
        console.log("Found:");
        console.log(docs);

        db.dropCollection("dishes", function (err, result) {
          assert.equal(err, null);
          db.close();
        });
      });
    });
});
```

- Then make sure MongoDB is running, run `node simple-server`

```
Connected to the server
After insert:
[ { name: 'Uthapizza',
    description: 'test',
    _id: 56f8b23f705fab5f64bebbbc } ]
Found:
[ { _id: 56f8b23f705fab5f64bebbbc,
    name: 'Uthapizza',
    description: 'test' } ]
```

#### Exercise II

- Develop a Node module containing some common MongoDB operations
- Use the Node module in the application and communicate with the MongoDB server

operations.js

```javascript
var assert = require('assert');

exports.insertDocument = function(db, document, collection, callback) {
  // Get the documents collection
  var coll = db.collection(collection);
  // Insert some documents into the collection
  coll.insert(document, function(err, result) {
    assert.equal(err, null);
    console.log("Inserted " + result.result.n + 
      " documents into the document collection " + collection);
    callback(result);
  });
};

exports.findDocuments = function(db, collection, callback) {
  // Get the documents collection
  var coll = db.collection(collection);

  // Find some documents, {} means select all
  coll.find({}).toArray(function(err, docs) {
    assert.equal(err, null);
    callback(docs);
  });
};

exports.removeDocument = function(db, document, collection, callback) {

  // Get the documents collection
  var coll = db.collection(collection);

  // Delete the document
  coll.deleteOne(document, function(err, result) {
    assert.equal(err, null);
    console.log("Removed the document " + document);
    callback(result);
  });
};

exports.updateDocument = function(db, document, update, collection, callback) {

  // Get the documents collection
  var coll = db.collection(collection);

  // Update document
  coll.updateOne(document, { $set: update }, null, function(err, result) {

    assert.equal(err, null);
    console.log("Updated the document with " + update);
    callback(result);
  });
};
```

server.js

```javascript
var MongoClient = require('mongodb').MongoClient;
var assert = require('assert');

var dboper = require('./operations');

// Connection URL
var url = 'mongodb://localhost:27017/conFusion';

// Use connect method to connect to the Server
MongoClient.connect(url, function (err, db) {
  assert.equal(null, err);
  console.log("Connected correctly to server");

  // db, document, collection, callback func
  dboper.insertDocument(db, { name: "Vadonut", description: "Test" },
    "dishes", function (result) {
    console.log(result.ops);

    // db, collection, callback func
    dboper.findDocuments(db, "dishes", function (docs) {
      console.log(docs);
    // db, document key, document update, collection, callback func
    dboper.updateDocument(db, { name: "Vadonut" },
      { description: "Updated Test" }, "dishes", function (result) {
        console.log(result.result);

        dboper.findDocuments(db, "dishes", function (docs) {
          console.log(docs);

          db.dropCollection("dishes", function (result) {
            console.log(result);
            db.close();
          });   
        });
      });
    });
  });
});
```

## Mongoose ODM

- MongoDB does not impose structure of documents, can be any documents
- Mongoose ODM
  - Object Data Model, Object Document Mapping
  - ORM: Object Relational Mapping
- Add structure to documents with schema
- Schema
  - Structure of the data: fields, types
  - Can do validation
  - Types: String, Number, Date, Buffer, Boolean, Mixed, ObjectId, Array
- Schema is used to create a Model function
- Embedded documents schema

#### Exercise III

- Install Mongoose

```
npm install mongoose --save
```

./models/dishes.js

```javascript
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
require('mongoose-currency').loadType(mongoose);
var Currency = mongoose.Types.Currency;

var commentSchema = new Schema({
  rating:  {
    type: Number,
    min: 1,
    max: 5,
    required: true
  },
  comment:  {
    type: String,
    required: true
  },
  author:  {
    type: String,
    required: true
  }
}, {
  timestamps: true
});

// create a schema
var dishSchema = new Schema({
  name: {
    type: String,
    required: true,
    unique: true
  },
  image: {
    type: String,
    default: ''
  },
  category: {
    type: String,
    default: ''
  },
  label: {
    type: String,
    default: ''
  },
  price: {
    type: Currency,
    default: ''
  },
  description: {
    type: String,
    required: true
  },
  // Sub-doc
  comments:[commentSchema]
}, {
  timestamps: true
});

var Dishes = mongoose.model('Dish', dishSchema);

// make this available to our Node applications
module.exports = Dishes;
```

server.js

```javascript
var mongoose = require('mongoose');
var assert = require('assert');

var Dishes = require('./models/dishes');

// Connection URL
var url = 'mongodb://localhost:27017/conFusion';mongoose.connect(url);
var db = mongoose.connection;

db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function () {
  // we're connected!
  console.log("Connected correctly to server");

  // create a new dish
  Dishes.create({
    name: 'Uthapizza',
    description: 'Test',
    comments: [
      {
        rating: 3,
        comment: 'This is insane',
        author: 'Matt Daemon'
      }
    ]
  }, function (err, dish) {
    if (err) throw err;
    console.log('Dish created!');
    console.log(dish);

    var id = dish._id;

    // get all the dishes
    setTimeout(function () {
      Dishes.findByIdAndUpdate(id, {
        $set: {
          description: 'Updated Test'
        }
      }, {
        new: true
      })
      .exec(function (err, dish) {
        if (err) throw err;
        console.log('Updated Dish!');
        console.log(dish);

        dish.comments.push({
          rating: 5,
          comment: 'I\'m getting a sinking feeling!',
          author: 'Leonardo di Carpaccio'
        });

        dish.save(function (err, dish) {
          console.log('Updated Comments!');
          console.log(dish);

          db.collection('dishes').drop(function () {
            db.close();
          });
        });
      });
    }, 3000);
  });
});
```