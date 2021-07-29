# Migrations with Sequelize

While this reading expects that you have used the Part 1 from the Authenticate
Me project to set up your backend Express server with Sequelize, this should be
easily applied to any backend server that uses an Object-Relational Mapper to
access a database with migrations and models.

## Why?

If you need to make changes to your database tables and you have a large number
of active users, you should always retain the already collected data you've
received from those active users. Un-migrating then re-migrating to make changes
to the database clears your tables as a side-effect. That is not ideal. It is
preferred that you generate a new migration containing only the needed changes.
This will prevent you from purging the already collected data from your database
tables.

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

For this example, the first migration you'll make is to add a column to the
`Users` table, so run this command to generate that migration.

```sh
npx sequelize migration:generate --name update-user-table
```

In the new migration file, which should have a name that starts with the
timestamp and ends with `-update-user-table.js`, you will set up the `up` method
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

Notice how you can set the column's attributes in the third parameter in the
`addColumn` method. When adding a new column to a table with pre-existing data
that has the non-nullable attribute, add a `defaultValue` attribute as well.
This will prevent the migration from error-ing out due to rows in the database
not having data for the new column.

To test this out, remove the `defaultValue` attribute and run the migration with
`npx dotenv sequelize db:migrate`. Notice the error message. Because there is
pre-existing data in our Users table from the seeder file, the migration fails.
You can make the necessary modification by re-adding the `defaultValue`
attribute and re-running the migration.

Check out the [Sequelize documentation] for more information on using migrations.

## Phase 2: Using one migration to do multiple things

In the previous phase, you set up a migration to do one thing, add one column to
one of our database tables. As you build out an application and the
corresponding database, you might need to add multiple columns or do multiple
things to your database. Making a single migration for each change might not be
ideal. In this next section, you'll go through making one migration add two
columns to our `Users` table.

Just like the previous phase, you will generate a migration with the following
command.

```sh
npx sequelize migration:generate --name more-user-table-updates
```

In the newest migration file, you will edit the `up` and `down` methods to use
transactions. This will ensure that either both columns are added or neither is
added. This helps create snapshots that allow every migration to be run in
either direction without creating changes that cause failure of subsequent
migrations.

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
`transaction` method returns a Promise, which is what you need to return from
both our `up` and `down` methods, and it accepts a callback that gets called
with the transaction object and should return a Promise, which is why you return
the `Promise.all` with our `addColumn` methods called inside. The same thing
happens in the `down` method, except with `removeColumn` method calls instead.

## Wrapping Up

In this exercise, you have gone through how to make migrations to update your
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
