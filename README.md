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

Feel free to

1. Clone down this repository
2. `cd` into the `backend` directory
2. Run `npm install` to get the dependencies installed
4. Create a `.env` file based off the `.env.example` file
5. Create a user in `psql` based off your `.env` file
6. Run `npx dotenv sequelize db:create`
7. Run `npx dotenv sequelize db:migrate`
8. Run `npx dotenv sequelize db:seed:all`
9. Run `npm start` to start the application to follow along.

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

Just like the previous phase, we will generate a migration with the following
command.

```sh
npx sequelize migration:generate --name more-user-table-updates
```

In the newest migration file, we will edit the `up` and `down` methods to use
transactions to make sure that both columns get added, or neither will get
added. This helps create snapshots so that every migration can be consistently
ran in either direction without causing changes that would make the migration
not run correctly the next time.

The contents of the migration file should now look something like this.

```js
// backend/db/migrations/«timestamp»-more-user-table-updates.js
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(t => {
      return Promise.all([
        queryInterface.addColumn('Users', 'middleInitial', {
          type: Sequelize.STRING,
          allowNull: false,
          defaultValue: '',
        }, { transaction: t }),
        queryInterface.addColumn('Users', 'lastName', {
          type: Sequelize.STRING,
          allowNull: false,
          defaultValue: '',
        }, { transaction: t }),
      ]);
    });
  },

  down: (queryInterface, Sequelize) => {
    return queryInterface.sequelize.transaction(t => {
      return Promise.all([
        queryInterface.removeColumn('Users', 'middleInitial', { transaction: t }),
        queryInterface.removeColumn('Users', 'lastName', { transaction: t }),
      ]);
    });
  },
};
```

The transaction prevents one change from happening if at least one other change
fails, which will make the migration easier to use and re-use since it will
consistently only add both columns, remove both columns, or do nothing. The
`transaction` method returns a Promise, which is what we need to return from
both our `up` and `down` methods, and it accepts a callback that gets called
with the transaction object and should return a Promise, which is why we return
the `Promise.all` with our `addColumn` methods called inside. The same thing
happens in the `down` method, except with `removeColumn` method calls instead.

## Wrapping Up

In this exercise, we have gone through how to make migrations to update your
database tables to avoid simply un-migrating your database and potentially
losing precious user data.

Some other things to consider is that if you do make changes to your seeder
files, it might also be good to consider making new seeders to update the
previous seeders just so that your seeder files also follow the same idea. Also,
you seeder files that rely on another set of seeds, like Posts that need a
`userId` property, should query for the user(s) in question rather than assuming
that the user's id will be consistent. For the most part the ids should be
consistent, but in the off chance that you have un-seeded and re-seeded your
seed data, then the ids for the Users or any other table might have reset.

Feel free to explore the [Sequelize docs] for more examples and descriptions of
the methods mentioned in this exercise.

[Sequelize documentation]: https://sequelize.org/master/manual/migrations.html#migration-skeleton
[Sequelize docs]: https://sequelize.org
