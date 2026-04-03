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
- [CRUD Operation:](#crud-operation)
  - [Create (POST)](#create-post)
    - [create():](#create)
    - [insertMany():](#insertmany)
  - [Read or Query(GET)](#read-or-queryget)
    - [find():](#find)
      - [find() Helpers:](#find-helpers)
        - [lean():](#lean)
        - [limit():](#limit)
        - [skip():](#skip)
        - [sort():](#sort)
        - [select():](#select-1)
        - [populate():](#populate)
        - [hint():](#hint)
        - [collation():](#collation)
    - [findOne():](#findone)
    - [findById():](#findbyid)
    - [countDocuments():](#countdocuments)
    - [distinct():](#distinct)
    - [aggregate():](#aggregate)
      - [Common Aggregation Stages:](#common-aggregation-stages)
  - [Update( PATCH/PUT )](#update-patchput-)
    - [PATCH (partial update - recommended):](#patch-partial-update---recommended)
      - [updateOne():](#updateone)
      - [findByIdAndUpdate():](#findbyidandupdate)
      - [updateMany():](#updatemany)
    - [Patch Operators:](#patch-operators)
      - [$set:](#set-1)
      - [$unset:](#unset)
      - [$inc:](#inc)
      - [$push:](#push)
      - [$pull:](#pull)
      - [$addToSet:](#addtoset)
      - [$pop:](#pop)
      - [$rename:](#rename)
      - [$mul:](#mul)
    - [PUT (Full Replace):](#put-full-replace)
      - [replaceOne():](#replaceone)
      - [findOneAndReplace():](#findoneandreplace)
  - [Delete (DELETE)](#delete-delete)
    - [deleteOne():](#deleteone)
    - [findByIdAndDelete():](#findbyidanddelete)
    - [deleteMany():](#deletemany)
    - [findOneAndDelete():](#findoneanddelete)
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

Note: The first argument is the singular name of the collection our model is for. Mongoose automatically convert plural and lowercased version of our model name.


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


# CRUD Operation:

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
  },
  age: Number,
}, { timestamps: true });

const userCollection = mongoose.model('User', userSchema);
```

## Create (POST)

### create(): 
```js
app.post('/users', async (req, res) => {
    const user = req.body;
    const result = await usersCollection.create(user);
    res.send(result); 
});
```
### insertMany():

```js
app.post('/users/bulk', async (req, res) => {
    const users = req.body;
    const result = await usersCollection.insertMany(users);
    res.send(result); 
});
```

## Read or Query(GET)
### find():

```js
app.get('/users', async (req, res) => {
    const result = await usersCollection.find();
    res.send(result);
});
```

#### find() Helpers:

##### lean():
Always use .lean() for queries like find or smiler heavy apis, because .lean() Returns plain JS objects instead of full Mongoose documents..

```js
app.get('/users', async (req, res) => {
    const result = await usersCollection.find().lean();
    res.send(result);
});
```

##### limit():

```js
app.get('/users', async (req, res) => {
    const result = await usersCollection.find().limit(10)
    res.send(result);
});
```

##### skip():

```js
app.get('/users', async (req, res) => {
    const page = parseInt(req.query.page); // http://localhost:3000/users?page=${page}
    const limit = 5;
    const skip = (page - 1) * limit;

    const result = await usersCollection.find().skip(skip).limit(limit)

    res.send(result);
});
```
- Page 1 → skip 0
- Page 2 → skip first 5
- Page 3 → skip first 10

##### sort():

```js
app.get('/users', async (req, res) => {
    const result = await usersCollection.find().sort({ createdAt: -1 })  // -1 = descending, 1 = ascending
  
  res.send(result);
});
```

##### select(): 
Controls which fields are returned.

```js
app.get('/users', async (req, res) => {
  const users = await User.find()
    .select('name email'); // include only these

  res.send(users);
});
```

Alternative:

```js
.select({ password: 0 }) // exclude field
```

##### populate(): 
ref tells Mongoose that a field (usually an ObjectId) references a document in another collection. Since MongoDB doesn’t support joins like SQL, Mongoose provides populate() to manually “join” related data by replacing the referenced ObjectId with the actual document.


```js
const postSchema = new mongoose.Schema({
  title: String,
  author: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'User' 
    }
});
```

```js
app.get('/posts', async (req, res) => {
  const posts = await Post.find()
    .populate('author'); // join user

  res.send(posts);
});
```

```js
// output
{
  "title": "....",
  "author": {
    "_id": "65f1abc123...",
    "name": "Tamim",
    "email": "t@gmail.com"
    "bio": "Full-stack dev",
    "age": 22
  }
}
```
  


customization:

```js
.populate({
  path: 'author',
  select: 'name email'
});
```

##### hint():
Tells MongoDB which index to use.
```js
app.get('/users', async (req, res) => {
  const users = await User.find({ email: req.query.email })
    .hint({ email: 1 });

  res.send(users);
});
```

##### collation():
Handles case-insensitive / language-aware sorting.

```js
app.get('/users', async (req, res) => {
  const users = await User.find()
    .collation({ locale: 'en', strength: 2 })
    .sort({ name: 1 });

  res.send(users);
});
```
- "apple" and "Apple" treated same


### findOne():

```js
app.get('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const result = await usersCollection.findOne(filter);
    res.send(result);
});
```

```js
app.get('/users/:email', async (req, res) => {
    const email = req.params.email;
    const filter = {email};
    const result = await usersCollection.findOne(filter);
    res.send(result);
});
```

### findById(): 

```js
app.get('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = { _id :id };
    const result = await usersCollection.findById(filter);
    res.send(result);
});
```

### countDocuments():

```js
app.get('/users', async (req, res) => {
    const role = req.query.role; // GET http://localhost:3000/users?role=user
    const filter = {role}
    const total = await usersCollection.countDocuments(filter);
    res.send( total );
});
```

### distinct():
Returns all unique values of a specific field as array of object.

```js
app.get('/users/email', async (req, res) => {
    const result = await usersCollection.distinct("email");
    res.send( result ); // ["a@gmail.com", "b@gmail.com"] 
});
```

```js
app.get('/users/email', async (req, res) => {
    const result = await usersCollection.distinct('email', { role: 'admin' }); // Give me unique emails of users where role = admin
    res.send( result ); // ["admin1@gmail.com", "admin2@gmail.com"] 
});
```

### aggregate():
- aggregate() is a method used to process data through a pipeline.
- A pipeline is an array of stages. Each step in the pipeline is called a stage. Each stage processes the documents and passes them to the next stage.

Note: Each stage starts with $ and the full array we called it pipeline

```js
usersCollection.aggregate([
  { $stage1: { ... } },
  { $stage2: { ... } },
  ...
])
```

Common Aggregation Operators:


| Operator                   | Purpose                    |
| -------------------------- | -------------------------- |
| `$sum`                     | Sum values                 |
| `$avg`                     | Average value              |
| `$min`                     | Minimum value              |
| `$max`                     | Maximum value              |
| `$first`                   | First value in group       |
| `$last`                    | Last value in group        |
| `$push`                    | Add value to array         |
| `$addToSet`                | Add unique values to array |
| `$concat`                  | Concatenate strings        |
| `$substr`                  | Substring                  |
| `$gte`, `$lte`, `$eq` etc. | Comparison operators       |
| `$cond`                    | Conditional expression     |


Note: Mongoose has lots of aggregation operators,

```js
// Arithmetic & Statistical:
$sum, $avg, $min, $max, $first, $last, $stdDevPop, $stdDevSamp

// String Operators:
$concat, $substr, $substrBytes, $substrCP, $toUpper, $toLower, $trim, $split, $indexOfBytes, $indexOfCP

// Array Operators
$push, $addToSet, $size, $filter, $map, $reduce, $arrayElemAt, $concatArrays, $slice, $reverseArray, $indexOfArray

// Conditional & Comparison Operators:
$cond, $ifNull, $eq, $ne, $gt, $gte, $lt, $lte, $and, $or, $not, $switch

// Type Conversion Operators:
$toInt, $toDouble, $toString, $type, $convert, $round, $trunc, $floor, $ceil
```

#### Common Aggregation Stages:

- $match: Filters documents based on specified conditions (similar to a query filter).   

```js
app.get('/users/adults', async (req, res) => {
  const adults = await usersCollection.aggregate([
    { $match: { age: { $gte: 18 } } }
  ])
  res.send(adults);
});
```

- $group: Groups documents by a key and applies aggregation operators (e.g., $sum, $avg, $count).

```js
app.get('/users/city-count', async (req, res) => {
  const cityCounts = await usersCollection.aggregate([
    { $group: { _id: "$city", totalUsers: { $sum: 1 } } }
  ])
  res.send(cityCounts);
});
```
Explanation: Groups users by city and counts how many users are in each city.

Input:
```
[
  { name: "Alice", city: "New York" },
  { name: "Bob", city: "London" },
  { name: "Charlie", city: "New York" },
  { name: "David", city: "Tokyo" }
]
```

Output:
```
[
  { _id: "New York", totalUsers: 2 },
  { _id: "London", totalUsers: 1 },
  { _id: "Tokyo", totalUsers: 1 }
]
```

Example 2: 

```js
app.get('/orders/total-revenue', async (req, res) => {
  const revenue = await ordersCollection.aggregate([
    { $group: { _id: null, totalRevenue: { $sum: "$amount" } } }
  ])
  res.send(revenue);
});
```
Explanation: Sums all order amounts to get total revenue.

Input: 
```
[
  { orderId: 1, amount: 100 },
  { orderId: 2, amount: 200 },
  { orderId: 3, amount: 150 }
]
```
Output:
```
[
  { _id: null, totalRevenue: 450 }
]
```

Example 2: 

```js
app.get('/users/average-age', async (req, res) => {
  const avgAge = await usersCollection.aggregate([
    { $group: { _id: null, averageAge: { $avg: "$age" } } }
  ])
  res.send(avgAge);
});
```
Explanation: Calculates the average age of all users (25 + 30 + 20) / 3 = 25.

Input:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 20 }
]
```
Output:
```
[
  { _id: null, averageAge: 25 }
]
```

Example 3:

```js
app.get('/users/age-range', async (req, res) => {
  const ageRange = await usersCollection.aggregate([
    { $group: { 
        _id: null, 
        maxAge: { $max: "$age" },
        minAge: { $min: "$age" }
    } }
  ])
  res.send(ageRange);
});
```
Explanation: Finds the oldest and youngest user ages.

Input: 
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 20 }
]
```
Output:
```
[
  { _id: null, maxAge: 30, minAge: 20 }
]
```

Example 4: 

```js
app.get('/cities/users-list', async (req, res) => {
  const cityUsers = await usersCollection.aggregate([
    { $group: { 
        _id: "$city", 
        userNames: { $push: "$name" }
    } }
  ])
  res.send(cityUsers);
});
```
Explanation: Groups users by city and creates an array of user names for each city.

Input: 
```
[
  { name: "Alice", city: "New York" },
  { name: "Bob", city: "London" },
  { name: "Charlie", city: "New York" }
]
```
Output:
```
[
  { _id: "New York", userNames: ["Alice", "Charlie"] },
  { _id: "London", userNames: ["Bob"] }
]
```

- $project: Controls which fields are included, excluded, or reshaped in the output.

```js
app.get('/users/names', async (req, res) => {
  const names = await usersCollection.aggregate([
    { $project: { name: 1, age: 1, _id: 0 } }
  ])
  res.send(names);
});
```
Explanation: Returns only name and age fields, excludes _id and city.

Input: 
```
[
  { _id: 1, name: "Alice", age: 25, city: "New York" },
  { _id: 2, name: "Bob", age: 30, city: "London" }
]
```

Output:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 }
]
```

- $sort: Orders documents by one or more fields (ascending or descending).

```js
app.get('/users/sorted', async (req, res) => {
  const sorted = await usersCollection.aggregate([
    { $sort: { age: -1 } }  // -1 = descending, 1 = ascending
  ])
  res.send(sorted);
});
```
Explanation: Sorts users by age in descending order (oldest first).

Input:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 20 }
]
```
Output:
```
[
  { name: "Bob", age: 30 },
  { name: "Alice", age: 25 },
  { name: "Charlie", age: 20 }
]
```

- $limit: Restricts the number of documents passed to the next stage.

```js
app.get('/users/top-3', async (req, res) => {
  const topUsers = await usersCollection.aggregate([
    { $sort: { age: -1 } },
    { $limit: 3 }
  ])
  res.send(topUsers);
});
```
Explanation: Returns only the top 3 oldest users.

Input:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 20 },
  { name: "David", age: 35 },
  { name: "Eve", age: 28 }
]
``` 
Output:
```
[
  { name: "David", age: 35 },
  { name: "Bob", age: 30 },
  { name: "Eve", age: 28 }
]
```

- $skip: Skips a specified number of documents (commonly used for pagination).

```js
app.get('/users/page-2', async (req, res) => {
  const page2 = await usersCollection.aggregate([
    { $skip: 2 },
    { $limit: 2 }
  ])
  res.send(page2);
});
```
Explanation: Skips the first 2 documents and returns the next 2.


- $unwind: Deconstructs an array into separate documents

```js
app.get('/users/hobbies', async (req, res) => {
  const hobbies = await usersCollection.aggregate([
    { $unwind: "$hobbies" }
  ])
  res.send(hobbies);
});
```
Explanation: Creates a separate document for each hobby in the array.

Input:
```
[
  { name: "Alice", hobbies: ["reading", "gaming"] },
  { name: "Bob", hobbies: ["swimming"] }
]
```
Output:
```
[
  { name: "Alice", hobbies: "reading" },
  { name: "Alice", hobbies: "gaming" },
  { name: "Bob", hobbies: "swimming" }
]
```

- $lookup: Performs a left outer join with another collection and adds matching documents as an array.

```js
app.get('/users/orders', async (req, res) => {
  const usersWithOrders = await usersCollection.aggregate([
    { $lookup: {
        from: "ordersCollection",
        localField: "_id",
        foreignField: "userId",
        as: "ordersHistory"
    } }
  ])
  res.send(usersWithOrders);
});
```

Explanation: Joins users with their orders from the orders collection.

Users Collection:
```
[
  { _id: 1, name: "Alice" },
  { _id: 2, name: "Bob" }
]
``` 
Orders Collection:
```
[
  { orderId: 101, userId: 1, product: "Laptop" },
  { orderId: 102, userId: 1, product: "Mouse" },
  { orderId: 103, userId: 2, product: "Keyboard" }
]
```
Output:
```
[
  {
    _id: 1,
    name: "Alice",
    ordersHistory: [
      { orderId: 101, userId: 1, product: "Laptop" },
      { orderId: 102, userId: 1, product: "Mouse" }
    ]
  },
  {
    _id: 2,
    name: "Bob",
    ordersHistory: [
      { orderId: 103, userId: 2, product: "Keyboard" }
    ]
  }
]
```
- $addFields: Adds new fields or modifies existing fields without removing others.

```js
app.get('/users/adult-flag', async (req, res) => {
  const users = await usersCollection.aggregate([
    { $addFields: { isAdult: { $gte: ["$age", 18] } } }
  ])
  res.send(users);
});
```

Explanation: Adds a new isAdult field that checks if age >= 18.

Input:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 16 },
  { name: "Charlie", age: 18 }
]
```
Output:
```
[
  { name: "Alice", age: 25, isAdult: true },
  { name: "Bob", age: 16, isAdult: false },
  { name: "Charlie", age: 18, isAdult: true }
]
```
- $count: Returns the total number of documents in the pipeline as a single result.

```js
app.get('/users/count', async (req, res) => {
  const result = await usersCollection.aggregate([
    { $match: { age: { $gte: 18 } } },
    { $count: "adultCount" }
  ])
  res.send(result);
});
```
Explanation: Counts how many users have age >= 18.

Input:
```
[
  { name: "Alice", age: 25 },
  { name: "Bob", age: 16 },
  { name: "Charlie", age: 30 }
]
```
Output:
```
[
  { adultCount: 2 }
]
```
- $facet: Runs multiple independent pipelines in parallel on the same input and returns combined results.

```js
app.get('/users/stats', async (req, res) => {
  const stats = await usersCollection.aggregate([
    { $facet: {
        "ageStats": [
          { $group: { _id: null, avgAge: { $avg: "$age" } } }
        ],
        "cityStats": [
          { $group: { _id: "$city", count: { $sum: 1 } } }
        ]
    } }
  ])
  res.send(stats);
});
```
Explanation: Runs two different aggregations at the same time - one for age stats and one for city stats.

Input: 
```
[
  { name: "Alice", age: 25, city: "New York" },
  { name: "Bob", age: 30, city: "London" },
  { name: "Charlie", age: 20, city: "New York" }
]
```
Output:
```
[
  {
    ageStats: [{ _id: null, avgAge: 25 }],
    cityStats: [
      { _id: "New York", count: 2 },
      { _id: "London", count: 1 }
    ]
  }
]
```


## Update( PATCH/PUT )
Note: In mongoose, always use:
- new: true = return updated document
- runValidators: true = enforce schema validation
 
### PATCH (partial update - recommended):
#### updateOne():

```js
app.patch('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const updatedData = req.body;
    const updatedDoc = { 
        $set: {
          name: updatedData.name
          age: updatedData.age
        }
      }
    const result = await usersCollection.updateOne(filter, updatedDoc, {
      new: true,
      runValidators: true,
    });

    res.send(result);
});
```


#### findByIdAndUpdate():

```js
app.patch('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const updatedData = req.body;
    const updatedDoc = { 
        $set: {
          name: updatedData.name
          age: updatedData.age
        }
      }
    const result = await usersCollection.findByIdAndUpdate(filter, updatedDoc, {
      new: true,
      runValidators: true,
    });

    res.send(result);
});
```

Note: if we used overwrite: true, then its works like put, means if no data found it create new. its not recommended for patch, because patch means update not create new if not found. 

And also this is not recommended to add all req.body data as updatedData.

```js
app.put('/users/:id', async (req, res) => {
    const id = req.params.id
    const filter = {_id: id}
    const updatedData = req.body
    const updatedDoc = {
        new: true,
        runValidators: true,
        overwrite: true,
    }
    const result = await UsersCollection.findByIdAndUpdate(filter, updatedData, updatedDoc);
    res.send(result);
});
```

#### updateMany():

```js
app.patch('/users/update-city', async (req, res) => {
    const { oldCity, newCity } = req.body;

    const result = await usersCollection.updateMany(
        { city: oldCity },      // find condition
        { $set: { city: newCity } } // update operation
    );

    res.send(result);
});
```




### Patch Operators: 

| Operator       | Purpose             |
| -------------- | ------------------- |
| `$set`         | Update fields       |
| `$unset`       | Remove fields       |
| `$inc`         | Increment numbers   |
| `$push`        | Add to array        |
| `$pull`        | Remove from array   |
| `$addToSet`    | Add unique to array |
| `$pop`         | Remove first/last   |
| `$rename`      | Rename field        |
| `$mul`         | Multiply value      |
| `$setOnInsert` | Set on insert       |

#### $set: 
Update a fields

```js
app.patch('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const updatedData = req.body;
    const updatedDoc = { 
        $set: {
          name: updatedData.name,s
          age: updatedData.age
        }
      }
    const result = await usersCollection.findByIdAndUpdate(filter, updatedDoc, {
      new: true,
      runValidators: true,
    });

    res.send(result);
});
```

#### $unset: 
Remove a field.

```js
app.patch('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const updatedData = req.body;
    const removedDoc = { 
        $unset: {
          name: updatedData.name,
          age: updatedData.age
        }
      }
    const result = await usersCollection.findByIdAndUpdate(filter, removedDoc, {
      new: true,
      runValidators: true,
    });

    res.send(result);
});
```

#### $inc: 
Increment / decrement numbers.

```js
app.patch('/users/:id/inc', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $inc: {
        loginCount: 1,
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

```js
app.patch('/users/:id/dec', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $inc: {
        loginCount: -1,
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $push: 
Add a item to an array in a filed

```js
app.patch('/users/:id/push', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $push: {
        hobbies: "coding"
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $pull:
remove a item to an array in a filed

```js
app.patch('/users/:id/pull', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $pull: {
        hobbies: "coding"
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $addToSet: 
Add a unique item to an array in a filed

```js
app.patch('/users/:id/add-to-set', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $push: {
        hobbies: "coding"
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $pop: 
remove first or last item to an array in a filed

```js
app.patch('/users/:id/pop', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $pop: {
        hobbies: 1 // 1 = last, -1 = first 
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $rename: 
Rename field.

```js
app.patch('/users/:id/rename', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
    $rename: {
        username: "name"
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

#### $mul:
Multiply value.

```js
app.patch('/users/:id/mul', async (req, res) => {
  const id = req.params.id;
  const updatedDoc = {
     $mul: {
        score: 2, // -2 for decrement
      }
  }
  const result = await User.findByIdAndUpdate(id, updatedDoc, { new: true });

  res.send(result);
});
```

### PUT (Full Replace):
#### replaceOne():

```js
app.put('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const newData = req.body;
    const result = await usersCollection.replaceOne(filter, newData, {
      new: true,
      runValidators: true,
      upsert: true
    });
    res.send(result);
});
```


if we used overwrite instead of upsert it Replaces whole document, but if not exist it can't create like upsert do.

```js
app.put('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const newData = req.body;
    const result = await usersCollection.replaceOne(filter, newData, {
      new: true,
      runValidators: true,
      overwrite: true
    });
    res.send(result);
});
```

#### findOneAndReplace():
Replace a document and return it. So here we don't need to use new: true.

```js
app.put('/users/replace/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}
    const newUserData = req.body;

    const result = await usersCollection.findOneAndReplace( filter, newUserData, {
       returnDocument: 'after', // return the replaced document
       runValidators: true , 
       }  
    );

    res.send(result);
});
```

## Delete (DELETE)
### deleteOne():

```js
app.delete('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = {_id: id}

    const result = await usersCollection.deleteOne(filter);

    res.send(result);
});
```

### findByIdAndDelete():

```js
app.delete('/users/:id', async (req, res) => {
    const id = req.params.id
    const result = await UsersCollection.findByIdAndDelete(id)
    res.send(result)
});
```

### deleteMany():

```js
app.delete('/users/:role', async (req, res) => {
    const filter = req.params.role
    const result = await usersCollection.deleteMany(filter);
    res.send(result);
});
```
Note: it removes all filed that have role (e.g. admin).

### findOneAndDelete():

```js
app.delete('/users/:email', async (req, res) => {
    const filter = req.params.email
    const result = await usersCollection.findOneAndDelete(filter);
    res.send(result);
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

// PUT Replace
app.put('/users/:id', async (req, res) => {
    const id = req.params.id;
    const filter = { _id: id }
    const newData = req.body;
    const result = await usersCollection.replaceOne(filter, newData, {
        new: true,
        runValidators: true,
        upsert: true
    });
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