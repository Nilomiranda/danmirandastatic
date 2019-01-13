---
title: Getting started with mongodb and mongoose
date: '2019-01-13'
---


So, in this guide we'll configure mongoose, create our first model (schema) and execute our first query

First things first, it'll be good to go if you already have a basic project configured with express installed.

Let's call this basic file as `index.js`

```javascript
const app = require('express')();

app.listen(3000);
```

Really simple initial configuration, an express app, and this app is listening to port number `3000`

Now it's time to get our hands dirty.

First, let's begin installing mongoose

```bash
npm install mongoose
```

Now, in our main file (`index.js`) let's start by requiring mongoose

```javascript
const mongoose = require('mongoose');
```



The first step here is configuring our mongodb connection

If your database has any authentication, with username and password, you should do as the following example:

``` javascript
mongoose.connect('mongodb://USER_NAME:PASS@localhost/DB_NAME')
```



Where `USER_NAME` should be your configured username for your mongodb configuration

`PASS` should be your password and `DB_NAME` should be your working database name



Now, if you don't have any special credentials to access your mongodb database, you just need to change the connection string a little bit:

```javascript
mongoose.connect('mongodb://localhost:27017/DB_NAME')
```



In this example, my mongodb doesn't require any credentials, so I just need to specify my localhost address, and my database name

```javascript
mongoose.connect('mongodb://localhost/dbconnexample')
```

Now with a connection enabled, we are able to create our first Schema.

In this example, we are going to create a simple post schema, containing a title, the content, and a field that'll save the date the post was created

```javascript
const postSchema = new mongoose.Schema({
    title: {
        type: String,
        required: true,
    },
    content: {
        type: String,
        required: true,
    },
    createdAt: {
        type: Date,
        default: Date.now,
    },
});
```

So, in this example above, we created a variable `postSchema` to keep the schema structure for our posts.

We create a schema by using `new mongoose.Schema()` and inside the arguments for the `Schema` method, we pass an object containing our data structure (or if you prefer, how our "table" will be).

Each field of our "table" can be an object with multiple different setting, telling our schema how our data will behave.

In this example I just used a few options that we have: `type, required, default`

But mongoose provides you with a big set of options to choose

You can more about how to define your schema in the following links:

https://mongoosejs.com/docs/guide.html

https://mongoosejs.com/docs/schematypes.html

Now that we have our schema defined, we can create a model based on that schema. We'll call this model as `Post`:

```javascript
mongoose.model('Post', postSchema);
```

The method `.model()` have an option to pass to arguments.

The first argument will be the name of your schema, as a `String` and will always be required. The second argument will be you Schema definition. For this second argument you can pass a variable or you can create the schema directly inside the `.model()` method, but personally I wouldn't recommend for the sake of good code readability.

Another important thing about the second argument is that it's optional. If you don't pass the second argument, mongoose will understand that you're only "calling" the previously created model. If you pass the second argument, mongoose will understand that you are defining a new model based on that specified schema.

With our schema defined and our model created based on that Schema, we can start populating our database with data. 

To insert data into our database, we can use mongoose's method `create()` .

So with that, we can create our first Post:

```javascript
Post.create({
    title: 'First post',
    content: 'Our first post content',
});
```

I'll be using Robo 3T as a client to look at our database

![structure](img/structure.png)

Now we can see our database `dbconnexample` in the list of databases and our 'posts' model inside the collections of our database. 

If we double click on posts we'll se this:

![table-content](img/table-content.png)

So here we can see that we've successfully created our document and populated it with some data.

We can se the our three fields we determined in our Schema:

* title
* content
* createdAt

And we can also see two other different fields that we didn't created by ourselves

* _id
* __v

The `_id` will automatically generate an id for each data inserted, in a completed different form that we are used to see in relational databases

The `__v` will work as a versioning method for our document. It'll be automatically incremented when we alter the structure of our document when it already has data in it.



```
An important note here: You don't need to necessarily create the database "dbconnexample" (or any other name you chose) before creating the Schema and running the script that will populate your database with data because mongoose will handle this dirty job for you.
```



So that's it for now. These are the very first steps to get you started with mongodb and mongoose in your nodejs project. 