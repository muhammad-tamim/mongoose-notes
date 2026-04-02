<h1 align="center">Mongoose Notes</h1>

- [Setup for Express + JS + Mongoose:](#setup-for-express--js--mongoose)
- [Introduction:](#introduction)
  - [Schema:](#schema)
  - [Model:](#model)
  - [Operation Buffering:](#operation-buffering)
  - [Error Handling:](#error-handling)
- [Schema Types:](#schema-types)
- [Schema Type Options:](#schema-type-options)
  - [Common Schema Type Options:](#common-schema-type-options)
    - [required:](#required)
    - [default:](#default)
    - [unique:](#unique)
    - [select:](#select)
    - [immutable:](#immutable)
    - [validate:](#validate)
    - [Set:](#set)
    - [Get:](#get)
    - [Index:](#index)
    - [expires:](#expires)
  - [String-Specific Schema Type Options:](#string-specific-schema-type-options)
    - [lowercase, uppercase, trim:](#lowercase-uppercase-trim)
    - [match:](#match)
    - [enum:](#enum)
  - [Number-Specific Schema Type Options:](#number-specific-schema-type-options)
    - [min / max:](#min--max)
  - [ObjectId-Specific Schema Type Options:](#objectid-specific-schema-type-options)
    - [ref:](#ref)
  - [Example:](#example)
- [Schema Options:](#schema-options)
    - [timestamps:](#timestamps)
- [Examples:](#examples)
  - [Example 1: Express + JS + Mongoose](#example-1-express--js--mongoose)

# Setup for Express + JS + Mongoose: 
- step 1: 

```bash
npm init -y
```

- step 2: 
  
```bash
npm i express mongoose nodemon cors dotenv
```

- step 3: 

```js
// index.js

const express = require('express');
const cors = require('cors');
require('dotenv').config();
const mongoose = require('mongoose');

const app = express();
const port = process.env.PORT || 3000;

// middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI);


// Mongoose Schema + Model
const userSchema = new mongoose.Schema({
    name: String,
    email: String,
});

const UsersCollection = mongoose.model('User', userSchema);

// all cred operations 

app.get('/', (req, res) => {
    res.send('Hello World!');
});


// Server Start
app.listen(port, () => {
    console.log(`🚀 Server running on port ${port}`);
});
```

we can use this function when we works on modular architecture:
```js
async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log('✅ MongoDB connected with Mongoose');
  } catch (error) {
    console.error('❌ MongoDB connection failed:', error.message);
    process.exit(1);
  }
}
```

- step 4: 

```json
{
  "name": "mongoose",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "cors": "^2.8.6",
    "dotenv": "^17.3.1",
    "express": "^5.2.1",
    "mongoose": "^9.3.3",
    "nodemon": "^3.1.14"
  }
}
```

- step 5:

```
MONGODB_URI=mongodb://localhost:27017/usersDB
```

# Introduction: 
Mongoose is an ODM (Object Data Modeling) library for Node.js that simplifies working with MongoDB. It adds structure and validation to MongoDB by providing:
- Schemas 
- Models 
- Built-in validation and middleware

So Mongoose gives us SQL-like discipline for a NoSQL database.

## Schema:
A Schema defines the structure of our data.

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: Number
});
```

## Model: 
A Model is a wrapper between database and schema. For the help of model we can do operation to the MongoDB with maintaining the rules of schema.

```js
const User = mongoose.model('User', userSchema);
```

## Operation Buffering: 
Mongoose lets you start using your models immediately, without waiting for mongoose to establish a connection to MongoDB. That's because mongoose buffers model function calls internally.

```js
mongoose.connect('mongodb://127.0.0.1:27017/myapp');
const MyModel = mongoose.model('Test', new Schema({ name: String }));
// Works
await MyModel.findOne();
```

## Error Handling: 
- Connection level: 

```js
mongoose.connect('mongodb://127.0.0.1:27017/test').
  catch(error => handleError(error));

// Or:
try {
  await mongoose.connect('mongodb://127.0.0.1:27017/test');
} catch (error) {
  console.log(error);
}
```

- Schema Level(Validation Errors): 

```js
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    match: [/^\S+@\S+\.\S+$/, "Invalid email format"]
  },
  age: {
    type: Number,
    min: [18, "Must be at least 18"]
  }
});
```



# Schema Types: 
A SchemaType defines the type and behavior of a single field inside a Schema.

```js
const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
});
```
here, 
- name → SchemaType = String
- age → SchemaType = Number
- email → SchemaType = String + extra rules

Below are complete list of all supported schema types: 
```
String, Number, Date, Buffer, Boolean, ObjectId, Array, Mixed, Union, Decimal128, Map, Schema, UUID, BigInt, Double, Int32
```

```js
const mongoose = require('mongoose');

const exampleSchema = new mongoose.Schema({
  name: String,                           // String
  age: Number,                            // Number
  isActive: Boolean,                      // Boolean
  createdAt: Date,                        // Date

  file: Buffer,                           // Buffer (binary data) e.g. Buffer.from("hello")

  userId: mongoose.Schema.Types.ObjectId, // ObjectId

  tags: [String],                         // Array

  anything: mongoose.Schema.Types.Mixed,  // Mixed (any type) Dangerous

  price: mongoose.Schema.Types.Decimal128, // Decimal128 (high precision) e.g. price: "19.99"

  settings: {
    type: Map,
    of: String                            // Map (key-value pairs) 
  },

  nested: new mongoose.Schema({           // Schema (subdocument)
    field: String
  }),

  uuid: mongoose.Schema.Types.UUID,       // UUID

  bigCounter: BigInt,                     // BigInt

  preciseNumber: mongoose.Schema.Types.Double, // Double

  smallNumber: mongoose.Schema.Types.Int32,    // Int32
});

const Example = mongoose.model('Example', exampleSchema);
```


Most Used Schema Types: 
```
String, Number, Boolean, Date, ObjectId, Array
```

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,                 // String
  age: Number,                 // Number
  isActive: Boolean,           // Boolean
  createdAt: Date,             // Date
  profileId: mongoose.Schema.Types.ObjectId, // ObjectId
  tags: [String],              // Array of strings ["developer", "mern", "learner"]
});

const User = mongoose.model('User', userSchema);
```

# Schema Type Options: 
## Common Schema Type Options: 
### required: 
Make field mandatory.

```js
email: {
  type: String,
  required: true
}
```

```js
required: [true, "Email is required"]
```

### default: 
Set default value.

```js
isActive: {
  type: Boolean,
  default: true
}
```

```js
createdAt: {
  type: Date,
  default: Date.now
}
```
Note: Don’t do Date.now() ❌ → it runs once. Use Date.now ✅

### unique: 
creates MongoDB unique index for that fields. So for that same data not allowed twice. if found it gives mongodb error. 

```js
email: {
  type: String,
  unique: true
}
```

Note: unique works on mongodb level, not mongoose level. SO mongoose can't validate uniqueness. so always handle duplicate error manually.

### select:
Control field visibility. Useful for secret fields that we don't want to show anyway on response.

```
password: {
  type: String,
  select: false
}
```
### immutable: 
Prevent updates after creation.

```js
email: {
  type: String,
  immutable: true
}
```

### validate: 
Custom validation.

```js
email: {
  type: String,
  validate: {
    validator: (v) => v.includes('@'),
    message: "Invalid email"
  }
}
```

```js
tags: {
  type: [String],
  validate: v => v.length <= 5
}
```


### Set: 
A setter is a function that runs when create data to MongoDB.

```js
name: {
  type: String,
  set: v => v.trim() // { name: "   Tamim   " } --> "Tamim"
}
```

```js
await User.create(req.body);
```

Note: set only works creation operations, it does not run on update operations by default. To enable we need to use runValidators. 

```js
await Users.findByIdAndUpdate(filter, { name: "   Tamim   " }, 
{
    new: true,
    runValidators: true
  });
```

### Get: 
A getter is a function that runs when we read data from a document.

```js
name: {
  type: String,
  get: v => v.toUpperCase()
}
```

```js
const user = await User.findById(filter);
console.log(user.name); // "TAMIM"
```

Note: Getter does NOT run in JSON output by default. To enable it, use getters: true inside toJSON options.

```js
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
});

userSchema.set('toJSON', { getters: true });
```

or: 

```js
const user = await User.findById(filter);
user.toJSON({ getters: true });
```

or: 

```js
schema.set('toJSON', { getters: true }); // now all schemas can works with getter
```

### Index: 

```js
email: {
  type: String,
  index: true
}
```

- Without index: 

```js
await User.findOne({ email: "a@gmail.com" });
```

MongoDB will scan every document in the collection

- With index: 
MongoDB uses a data structure (like B-Tree) that time complexity is O(log n) super fast.

### expires: 
auto delete data after expires time.
```js
createdAt: {
  type: Date,
  expires: 3600 // seconds
}
```

## String-Specific Schema Type Options:
### lowercase, uppercase, trim: 

```js
email: {
  type: String,
  lowercase: true,
  trim: true
}
```

### match: 
regex validation.

```js
email: {
  type: String,
  match: [/^\S+@\S+\.\S+$/, "Invalid email"]
}
```

### enum: 

```js
role: {
  type: String,
  enum: ["user", "admin"]
}
```

## Number-Specific Schema Type Options: 

### min / max:

```js
age: {
  type: Number,
  min: 18,
  max: 60
}
```


## ObjectId-Specific Schema Type Options: 
### ref:
ref tells Mongoose this ObjectId points to a document in another collection (Profile)

```js
profileId: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "Profile"
}
```

MongoDB doesn’t have joins like SQL.. so ref allows populate function on mongoose, that help to get that id data: 

Without ref: 


```js
profileId: {
  type: mongoose.Schema.Types.ObjectId,
}
```

```js
const user = await User.findById(id);

// output
{
  "name": "Tamim",
  "profileId": "65f1abc123..."
}
```

With ref + populate: 

```js
profileId: {
  type: mongoose.Schema.Types.ObjectId,
  ref: "Profile"
}
```

```js
const user = await User.findById(id).populate("profileId");

// output
{
  "name": "Tamim",
  "profileId": {
    "_id": "65f1abc123...",
    "bio": "Full-stack dev",
    "age": 22
  }
}
```

## Example: 

```js
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },

  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: /^\S+@\S+\.\S+$/
  },

  password: {
    type: String,
    required: true,
    select: false
  },

  age: {
    type: Number,
    min: 18
  },

  role: {
    type: String,
    enum: ["user", "admin"],
    default: "user"
  },

  profileId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: "Profile"
  }
});
```

# Schema Options: 
Schema options are configuration settings for the entire schema, not individual fields. 

Below a complete list of schema options in mongoose. 

```
timestamps, versionKey, collection, strict, minimize, toJSON / toObject, 
id, _id, capped, bufferCommands, strictQuery, autoIndex, 
```

Note: For most of the case we skip schema options thats why this chapter is not much important as scheme type options.

### timestamps:
Automatically adds `createdAt` and `updatedAt`: 

```js
const schema = new mongoose.Schema({}, {
  timestamps: true
});
```


# Examples: 

## Example 1: Express + JS + Mongoose 

```js
// index.js

const express = require('express');
const cors = require('cors');
require('dotenv').config();
const mongoose = require('mongoose');

const app = express();
const port = process.env.PORT || 3000;

// middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI);


// Mongoose Schema + Model
const userSchema = new mongoose.Schema({
    name: String,
    email: String,
});

const UsersCollection = mongoose.model('User', userSchema);

// all cred operations 

// root
app.get('/', (req, res) => {
    res.send('Hello World!');
});

// CREATE
app.post('/users', async (req, res) => {
    const user = req.body
    const result = await UsersCollection.create(user);
    res.send(result);
});

// READ ALL
app.get('/users', async (req, res) => {
    const result = await UsersCollection.find()
    res.send(result)
})

// READ ONE
app.get('/users/:id', async (req, res) => {
    const filter = req.params.id
    const result = await UsersCollection.findById(filter);
    res.send(result);
});

// PATCH UPDATE
app.patch('/users/:id', async (req, res) => {
    const filter = req.params.id
    const { name, email } = req.body
    const updatedData = { name, email }
    const updatedDoc = {
        new: true,
        runValidators: true
    }
    const result = await UsersCollection.findByIdAndUpdate(filter, updatedData, updatedDoc);
    res.send(result);
});

// PUT UPDATE 
app.put('/users/:id', async (req, res) => {
    const filter = req.params.id
    const updatedData = req.body
    const updatedDoc = {
        new: true,
        runValidators: true,
        overwrite: true,
    }
    const result = await UsersCollection.findByIdAndUpdate(filter, updatedData, updatedDoc);
    res.send(result);
});

// DELETE
app.delete('/users/:id', async (req, res) => {
    const filter = req.params.id
    const result = await UsersCollection.findByIdAndDelete(filter)
    res.send(result)
});


// Server Start
app.listen(port, () => {
    console.log(`🚀 Server running on port ${port}`);
});
```