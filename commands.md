Masonite ORM comes with several terminal commands you can run to speed up your development. Here are a list of commands and their descriptions

Below are examples to use Masonite ORM standalone but if you are using Masonite ORM with Masonite then you can replace the calls to `masonite-orm` with your applications `python craft` file.

For example you would replace the call for:

```
$ masonite-orm model User
```

with 

```
$ python craft model User
```

# Migrations

Migration commands are used to create migration files, roll them back and refresh your database

## Creating

You can create migration files easily. Migration files will create a class with an `up` method and a `down` method. You should perform your schema logic in the `up` method and then reverse what you did in the `down` method.

```
$ masonite-orm migration create_users_table
```

| Argument   |      Description      |  Example |
|----------|:-------------|------:|
| `name_of_migration` |  The name of the migration file to create | `create_users_table` |

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--create {name}` |  Makes a migration file for creating a new table | `--create users` |
| `--table {name}` |  Makes a migration file for altering an existing table | `--table users` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |

## Migrating

Migrating will run each unmigrated migration's `up` method. Each group of migrations that are ran will create a batch number and store information in the `migrations` table. The batch number will allow groups of migrations to be rolled back if needed.

```
$ masonite-orm migrate
```


| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--migration {name}` |  Specify a specific migration to rollback. This will default to rolling back the previous batch of migrations. | `--migration create_users_table` | 
| `--connection {name}` |  The name of the connection to use to rollback the migrations | `--connection staging` |
| `--show` |  If passed this command will output the SQL that will run to modify the schema and will not actually run the SQL. | `--show` |
| `--force` |  If the `APP_ENV` environment variable is set to `production` then a prompt will ask you if you really want to migrate as a safety check. Using this flag will ignore the prompt and migrate anyway | `--force` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |

## Rollback

You can rollback your migration files as well. This would run your migration files that are already migrated but in reverse order. This will run the `down` method on each migration file in reverse order to "undo" the migration changes that were previously ran.

When a group of migrations are migrated, that group will create a batch number. The rollback command will only rollback the last batch, or the migrations that were ran in the last group of migrations that you ran.

```
$ masonite-orm migrate:rollback
```


| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--migration {name}` |  Specify a specific migration to rollback. This will default to rolling back the previous batch of migrations. | `--migration create_users_table` | 
| `--connection {name}` |  The name of the connection to use to rollback the migrations | `--connection staging` |
| `--show` |  If passed this command will output the SQL that will run to modify the schema and will not actually run the SQL. | `--show` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |

## Resetting

While the rollback method will rollback the previous batch of migrations, the reset command will rollback all migrations.

```
$ masonite-orm migrate:reset
```

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--migration {name}` |  Specify a specific migration to rollback. This will default to rolling back the previous batch of migrations. | `--migration create_users_table` | 
| `--connection {name}` |  The name of the connection to use to rollback the migrations | `--connection staging` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |

## Refreshing

Refreshing migrations is a simple way to redo all of your migrations. First it will rollback all migrations and then automatically migrate all the migrations again.

```
$ masonite-orm migrate:refresh
```


| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--migration {name}` |  Specify a specific migration to refresh. This will default to refreshing everything. | `--migration create_users_table` | 
| `--connection {name}` |  The name of the connection to use to refresh the migrations | `--connection staging` |
| `--seed` |  Whether the seed:run command should be ran after the database schema is refreshed | `--seed` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |
| `--seed-directory=databases/seeds` |  Specifies where the seed directory is | `--seed-directory=databases/seeds` |

## Getting Migration Status

Sometimes its good to know the status of the migrations so you can know if you have any migrations that need to be ran:

```python
$ masonite-orm migrate:status
```

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--connection {name}` |  The name of the connection to use to refresh the migrations | `--connection staging` |
| `--directory=databases/migrations` |  Specifies where the migration directory is | `--directory app/databases/migrations` |

# Creating Models

You can create a new model class quickly. There are also several options you can pass to this command to quickly create migrations and seeds quickly.

```
$ masonite-orm model {name}
```

| Argument   |      Description      |  Example |
|----------|:-------------|------:|
| `{name}` |  The name of the model to create | `User` |

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--migration {name}` |  Specify a specific migration to refresh. This will default to refreshing everything. | `--migration create_users_table` | 
| `--migration` |  Whether to create a migration file as well | `--migration` |
| `--seed` |  Whether the seed:run command should be ran after the database schema is refreshed | `--seed` |
| `--create` |  Whether the created migration should have the create option. | `--create` |
| `--table` |  Whether the created migration should have the table option. | `--table` |
| `--pep` |  Whether the created file should follow pep8 | `--pep` |
| `--directory=app` |  Which file the model should be created in | `--pep` |
| `--migrations-directory=databases/migrations` |  The directory of the migrations. Use this option if using the `migration` option. | `--migrations-directory=app/migrations` |
| `--seeders-directory=databases/seeds` |  The directory of the seeding classes. Use this option if using the `seed` option. | `--seeders-directory=app/seeds` |

# Model Docstrings

One of the downsides of Masonite ORM compared to other models is you don't know what columns and data types you have on your models / tables.

For example, on other ORMs, columns are class attributes on your models so you can always reference your models to know what your tables look like.

To solve this with Masonite ORM you can use the `model:docstring` command. This command will output an example docstring of all your tables columns and their data types so you can put it on your model for reference. When you make schema changes you can rerun this command to get the updated schema.

Another downside is not being able to see IDE type hints on your models. This is also solved using the `--type-hints` option you can find below.

```
$ masonite-orm model:docstring {table}
```

| Argument   |      Description      |  Example |
|----------|:-------------|------:|
| `{table}` |  The name of the table you want to create the docstring for | `User` |

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--type-hints` |  Used to also optionally output type hints you can add to your model class that your IDE can use to assist in column type hinting | `--type-hints` | 
| `--connection {name}` |  The name of the connection to use to use. | `--connection staging` |

# Make Observers

You can easily build observer files:

```
$ masonite-orm observer {name}
```

| Argument   |      Description      |  Example |
|----------|:-------------|------:|
| `{name}` |  The name of the observer to create | `UserObserver` |

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--model` |  Name of the model to build the observer for | `--model User` | 
| `--directory=app/observers` |  The location of the observers directory. | `--directory app/databases` |

# Seeding

Seeding is a great way to get test data into your database. 

## Making a Seed File

To make a seed file is simple:

```python
$ masonite-orm seed {table}
```

| Argument   |      Description      |  Example |
|----------|:-------------|------:|
| `{table}` |  The table of the seed to create | `users` |

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--directory=databases/seeds` |  The location of the seeds directory. | `--directory app/seeds` |

## Running Seeds

To run the seeds is simple:

```
$ masonite-orm seed:run
```

| Options   |      Description      |  Example |
|----------|:-------------|------:|
| `--connection {name}` |  The name of the connection to use to refresh the migrations | `--connection staging` |
| `--dry` |  If the seed should run in dry mode | `--dry` |
| `--table` |  The name of the table to seed | `--table users` |
| `--directory=databases/seeds` |  The location of the seeds directory. | `--directory app/seeds` |
