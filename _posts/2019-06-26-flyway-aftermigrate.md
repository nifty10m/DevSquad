---
layout: post
title: Flyway and after Migrate
---

When automating deployment you have to use a tool for automated database migration. We have good experience using [Flyway](https://flywaydb.org) which we use also in our Spring Boot applications.

## Basic concept

The basic concept of Flyway is there are repeatable and versioned migration files either written in Java or SQL.

* The repeatable files are called `R__name_of_file.sql`
* The versioned files are called `V42_1_name_of_file.sql` where 42.1 is the version which should be tracked and is stored in the database.

Every script has a checksum so flyway can verify you did not change the script after applying.

## Before and after migrate

Flyway also supports scripts called `afterMigrate.sql` and `beforeMigrate.sql`. Those scripts are executed before or after each migration (whenever a new script is created or an existing script is changed). They are helpful if you want to apply `UPDATE` or other statements which are related to the result of other migration scripts.

A less know fact is, there can be more than one _after_ or _before_ script. I.e. you can create multiple after migrate script by naming your script:

* afterMigrate__do_basic_stuff.sql
* afterMigrate__finish_users.sql
* afterMigrate__read_deployment_version.sql

You just have to place those files in the db/migrations folder of your resource directory and all those scripts are executed in alphabetical order.

{% include twitter.html %}