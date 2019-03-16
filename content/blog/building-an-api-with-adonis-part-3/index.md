---
title: Building an API with AdonisJS (part 3)
date: '2019-03-16'
---

Hello folks! The third part of the series is finally here! 👏👏👏👏

If you are a new comer, this is a series that will cover all the steps we need to build an API using [AdonisJS](https://adonisjs.com). This is the third part of the series, and here are the links for the precious posts:

- [Part 1](https://danmiranda.io/bulding-an-api-with-adonisjs/)
- [Part 2](https://danmiranda.io/building-an-api-with-adonis-part-2/)

In this part, and I promise this will be shorter, we will cover how to implement the feature for a user to create a new event, setting a specific date, location and time.

So we'll learn how to create a new model, as the previous one was already created at the moment we scaffolded our application, how to create a new migration to properly set the columns we'll need in our table and how to stablish a relationship between to models.

So let's get our hands dirty...

## Creating the events table

This API will allow the user to schedule events, setting a location, the time, date and the event's title (name)

So we will need to set 4 columns:

- Title (string)
- Location (string)
- Date (date)
- Time (timestamp)

As this table will have a relationship with the user, as one may have any number of events we'll also need a column of the user's ID. This column will make a reference to the main column, User.

In Adonis, to create a new model we will do the following:

```bash
adonis make:model Event -c -m
```

What I'm doing here is telling adonis to make a new model, called `Event` and I'm passing two flags: `-c` and `-m`. These two flags will tell adonis to also create the controller (`-c`) and the migration (`-m`).

Now let's begin to structure our table. Head to the migration file `database/migrations/1551814240312_event_schema.js` 

Inside your class `EventSchema`, for the `up()` method, do the following:

```javascript
class EventSchema extends Schema {
  up () {
    this.create('events', (table) => {
      table.increments()
      table
        .integer('user_id')
        .unsigned()
        .references('id')
        .inTable('users')
        .onUpdate('CASCADE')
        .onDelete('SET NULL')
      table.string('title').notNullable()
      table.string('location').notNullable()
      table.datetime('date').notNullable()
      table.time('time').notNullable()
      table.timestamps()
    })
  }
```

Let's see what we are doing:

```javascript
table
    .integer('user_id')
    .unsigned()
    .references('id')
    .inTable('users')
    .onUpdate('CASCADE')
    .onDelete('SET NULL')
```

This piece of code above is the one responsible to create the user's ID column and reference to the User table.

First, we set the data type to integer with `table.integer('user_id')` setting the column name as `user_id` inside the parameter. 

With `.unsigned()` we set the column to only accept positive values (-1, -2, -3 are invalid numbers).

We then tell the column to reference to the user's id column with `.references('id)` in the User table with `.inTable('users').

If we happen to change the event's owner ID, then all the changes will reflect to the Event table, so the user's ID in the column `user_id` will also change (`.onUpdate('CASCADE')`).

In case the user's account ends up deleted, the event's `user_id` column of the events the deleted user owned will all be set to null `.onDelete('SET NULL')`. 

```javascript
table.string('title').notNullable()
table.string('location').notNullable()
table.datetime('date').notNullable()
table.time('time').notNullable()
```

Now we set the other columns:

- The title column, as a STRING with `table.string('title')`
- The location column, also as a STRING with `table.string('location')`
- The date column, as a DATETIME with `table.datetime('date')`
- And the time column, as a TIME with `table.time('time')

Notice that for all these columns in each one of them I also set `.notNullable()` because the user will have to set each of these values everytime he creates a new event.

After all this work we can run our migration:

```bash
adonis migration:run
```

To finish setting up the relationship between the Event and User tables we have two options:

- We set the relationship in the User's model
- We set the relationship in the Event's model

In this example, we will set the relationship in the User's model. We don't need to set the relationship in both models as Adonis' documentation itself states that:

> There is no need to define a relationship on both the models. Setting it one-way on a single model is all that’s required.

So let's go to `App/Models/User.js` and add the method `events()`.

```javascript
events () {
    return this.hasMany('App/Models/Event')
}
```

That's all we need to do! Now we'll be able to start creating our controller to create and list new events.

## Creating and saving a new event

First let's create our `store()` method to enable a user to create and save a new event. 

In `App/Controllers/Http/EventController.js` we'll do:

```javascript
async store ({ request, response, auth }) {
    try {
      const { title, location, date, time } = request.all() // info for the event
      const userID = auth.user.id // retrieving user id current logged

      const newEvent = await Event.create({ user_id: userID, title, location, date, time })

      return newEvent
    } catch (err) {
      return response
        .status(err.status)
        .send({ message: {
          error: 'Something went wrong while creating new event'
        } })
    }
  }
```

It's really simple. We retrieve the data coming from the request with `request.all()`

We also need to retrieve the logged user's ID, but we get this data saved in the auth object, provided by the context.

```javascript
const userID = auth.user.id
```

To use this controller we just create a new route, inside `Route.group()`:

```javascript
Route.post('events/new', 'EventController.store')
```

To test the event creation we send a request to this route, sending a JSON data following the structure below:

```JSON
{
	"title": "First event",
	"location": "Sao Paulo",
	"date": "2019-03-16",
	"time": "14:39:00"
}
```

If everything run smoothly the request will return you the created event:

```JSON
{
  "user_id": 10,
  "title": "First event",
  "location": "Sao Paulo",
  "date": "2019-03-16",
  "time": "14:39:00",
  "created_at": "2019-03-16 14:40:43",
  "updated_at": "2019-03-16 14:40:43",
  "id": 6
}
```

## Listing events

We will have two ways to list events in this API, list all events or by date.

Let's begin by listing all events. We'll create a method `index()`:

```javascript
async index ({ response, auth }) {
    try {
      const userID = auth.user.id // logged user ID

      const events = await Event.query()
        .where({
          user_id: userID
        }).fetch()

      return events
    } catch (err) {
      return response.status(err.status)
    }
  }
```

We'll list all events searching by the logged user's ID, so as we did before we use `auth.user.id` to get this information.

The way we query the data here will be a bit different than we previously did as we won't use any static method in this case.

```javascript
const events = await Event.query()
        .where({
          user_id: userID
        }).fetch()
```

We open the query with `.query()`  and then we set the where statement, passing an object as parameter to pass the filters to search for the data:

```javascript
.where({
    user_id: userID
})
```

Unlike the special static methods, we need to chain the method `.fetch()` to correctly retrieve the data.

This is easier to test, we just need to set a route for a GET request in `start/routes.js`:

```javascript
Route.get('events/list', 'EventController.index')
```

This request won't need any parameter. If successfully completed you'll have as return the list of all events inside an array:

```json
[
  {
    "id": 6,
    "user_id": 10,
    "title": "First event",
    "location": "Sao Paulo",
    "date": "2019-03-16T03:00:00.000Z",
    "time": "14:39:00",
    "created_at": "2019-03-16 14:40:43",
    "updated_at": "2019-03-16 14:40:43"
  }
]
```

Now we'll list the events by date, and for that we'll create a method called `show()`. 

```javascript
async show ({ request, response, auth }) {
    try {
      const { date } = request.only(['date']) // desired date
      const userID = auth.user.id // logged user's ID

      const event = await Event.query()
        .where({
          user_id: userID,
          date
        }).fetch()

      if (event.rows.length === 0) {
        return response
          .status(404)
          .send({ message: {
            error: 'No event found'
          } })
      }

      return event
    } catch (err) {
      if (err.name === 'ModelNotFoundException') {
        return response
          .status(err.status)
          .send({ message: {
            error: 'No event found'
          } })
      }
      return response.status(err.status)
    }
```

What we are doing is, retrieving the date sent in the request and the logged user's ID. Then again we manually query for events using the user's ID and the date he provided within his request.

Now we need to check if we have events in the given date or not. 

If there's no event the following piece of code runs and returns a message:

```javascript
if (event.rows.length === 0) {
    return response
        .status(404)
        .send({ message: {
            error: 'No event found'
        } })
}
```

If there's the event we simply return it.

Don't forget to create a route to call this controller when accessed:

```javascript
Route.get('events/list/date', 'EventController.show')
```

In this example we created an event to happen in March 16th, 2019. If we sent the following JSON in the request:

```json
{
	"date": "2019-03-16"
}
```

We receive as return:

```json
[
  {
    "id": 6,
    "user_id": 10,
    "title": "First event",
    "location": "Sao Paulo",
    "date": "2019-03-16T03:00:00.000Z",
    "time": "14:39:00",
    "created_at": "2019-03-16 14:40:43",
    "updated_at": "2019-03-16 14:40:43"
  }
]
```

If we, for example, look for an event in March 26th:

```json
{
	"date": "2019-03-26"
}
```

We'll receive the following:

```json
{
  "message": {
    "error": "No event found"
  }
}
```

## Deleting events

The only feature missing is the ability to delete an event. It'll be quite simple. We'll get, as usual, the logged user's ID and the event ID. Then we look for the event in the database. To make sure that the user only deletes his owned events we'll check if the logged user's ID is the same as the event being deleted and then proceed deleting the event.

Let's add some code to our `destroy()` method:

```javascript
async destroy ({ params, response, auth }) {
    try {
      const eventID = params.id // event's id to be deleted
      const userID = auth.user.id // logged user's ID

      // looking for the event
      const event = await Event.query()
        .where({
          id: eventID,
          user_id: userID
        }).fetch()

      /**
       * As the fetched data comes within a serializer
       * we need to convert it to JSON so we are able 
       * to work with the data retrieved
       * 
       * Also, the data will be inside an array, as we
       * may have multiple results, we need to retrieve
       * the first value of the array
       */
      const jsonEvent = event.toJSON()[0]

      // checking if event belongs to user
      if (jsonEvent['user_id'] !== userID) {
        return response
          .status(401)
          .send({ message: {
            error: 'You are not allowed to delete this event'
          } })
      }

      // deleting event
      await Event.query()
        .where({
          id: eventID,
          user_id: userID
        }).delete()
```

Just a side note: As we are working with the query builder we need to delete it 'manually', also using the query builder, but if you are, in another example, fetching data using the static methods provides by the models, you just need to using the static method `.delete()`.

Let's test our `destroy()` method. In your `start/routes.js` file add the following `delete` request:

```javascript
Route.delete('events/:id/delete', 'EventController.destroy')
```

As we are only sending all the data we need through the Url we won't need to send any data in the request's body.

This is it for this one guys! 

Today we learned how to create a new model, together with a controller and a migration file and also how to set relationship between different tables