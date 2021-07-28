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

For this example

```sh
npx sequelize migration:generate --name update-user-table
```

```sh
npx sequelize migration:generate --name more-user-table-updates
```

* Adding `defaultValue` to the new columns to prevent errors with older data
* Update seeder file
* Note on updating other seeders that rely on the demo user seeds to query for
    those users instead of just assuming the IDs
* Adding multiple columns at the same time in one migration
