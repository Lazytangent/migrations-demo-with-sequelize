# Migrations with Sequelize

While this reading expects that you have used the Part 1 from the Authenticate
Me project to set up your backend Express server with Sequelize, this should be
easily applied to any backend server that uses an Object-Relational Mapping to
access a database with migrations and models.

## Why?

When your application has been used by some number of people already and your
database is storing information that makes your users want to continue to use
your application, you won't want to just throw away all that data if you need to
make changes to your database tables. This means that un-migrating to make a
change on a pre-existing migration, then re-migrating to apply those changes,
which clears your tables as a side effect, is not ideal. Instead, it is
preferred to generate a new migration that makes the changes you need to your
database tables, so that you can just run the new migration(s) without ever
purging your database tables of the data that have come from your users.

## Phase 0: Getting Started

Feel free to clone down this repository, `cd` into the `backend` directory, run
`npm install` to get the dependencies installed, creating a `.env` file
based off the `.env.example` file, creating a user in `psql` based off your
`.env` file, running `npx dotenv sequelize db:create`, `npx dotenv sequelize
db:migrate`, `npx dotenv sequelize db:seed:all`, then finally running `npm
start` to start the application to follow along.

### Overview of the application

This application consists of just a backend that's connected to a PostgreSQL
database. The setup is exactly the same as the setup from Authenticate Me Part
1.

## Phase 1: Creating a new migration

You can generate a new migration file with the following command, replacing the
`«name of migration»` filler text with the name of your new migration.

```sh
npx sequelize migration:generate --name «name of migration»
```

For this example, the first migration we'll make is to add a column to the
`Users` table, so run this command to generate that migration.

```sh
npx sequelize migration:generate --name update-user-table
```

In the new migration file, which should have a name that starts with the
timestamp and ends with `-update-user-table.js`, we will set up the `up` method
to use the `addColumn` method to add a `firstName` column to our `Users` table
and the `down` method to use the `removeColumn` method to remove the
`firstName` column from our `Users` table.

The contents of the migration file should now look something like this.

```js
// backend/db/migrations/«timestamp»-update-user-table.js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('Users', 'firstName', {
      type: Sequelize.STRING,
      allowNull: false,
      defaultValue: '',
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.removeColumn('Users', 'firstName', {});
  },
};
```

Notice how we can set the column's attributes in the third parameter in the
`addColumn` method. When adding a new column to a table with pre-existing data
that has the non-nullable attribute, add a `defaultValue` attribute as well so
that the migration doesn't error out because of rows in your database that won't
have data for that new column.

You can test this out by removing the `defaultValue` attribute and running this
migration with `npx dotenv sequelize db:migrate` to see the error message. Since
it would have failed to migrate since there was pre-existing data in our `Users`
table from the seeder file, we can make the necessary modification to the this
migration by adding the `defaultValue` attribute back, and re-running this
migration.

Check out the [Sequelize documentation] for more information on using migrations.

## Phase 2: Using one migration to do multiple things

In the previous phase, we set up a migration to do one thing, add one column to
one of our database tables. As you build out an application and the
corresponding database, you might need to add multiple columns or do multiple
things to your database, and making a single migration for each change might not
be the most ideal. In this next section, we'll go through making one migration
add two columns to our `Users` table.

```sh
npx sequelize migration:generate --name more-user-table-updates
```

* Adding `defaultValue` to the new columns to prevent errors with older data
* Update seeder file
* Note on updating other seeders that rely on the demo user seeds to query for
    those users instead of just assuming the IDs
* Adding multiple columns at the same time in one migration

[Sequelize documentation]: https://sequelize.org/master/manual/migrations.html#migration-skeleton
