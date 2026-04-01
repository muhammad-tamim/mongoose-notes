<h1 align="center">Mongoose Notes</h1>

- [Setup:](#setup)
- [Introduction:](#introduction)
  - [Schema:](#schema)
  - [Model:](#model)
- [Schemas:](#schemas)
- [Examples:](#examples)
  - [Example 1:](#example-1)

# Setup: 
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
A Schema defines the structure of your data.

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  age: Number
});
```

## Model: 
A Model is a wrapper around the schema used to interact with the database.

```
const User = mongoose.model('User', userSchema);
```

# Schemas: 

# Examples: 

## Example 1: 

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