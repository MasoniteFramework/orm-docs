# Schema & Migrations

Migrations are used to build and modify your database tables. This is done through use of migration files and the `Schema` class. Migration files are really just wrappers around the `Schema` class as well as a way for Masonite to manage which migrations have run and which ones have not.

## Creating Migrations

Creating migrations are easy with the migration commands. To create one simply run:

```text
$ masonite-orm migration migration_for_users_table
```

This will create a migration file for you and put it in the `databases/migrations` directory.

If you want to create a starter migration, that is a migration with some boilerplate of what you are planning to do, you can use the `--table` and `--create` flag:

```text
$ masonite-orm migration migration_for_users_table --create users
```

This will setup a migration for you with some boiler plate on creating a new table

```text
$ masonite-orm migration migration_for_users_table --table users
```

This will setup a migration for you for boiler plate on modifying an existing table.

## Building Migrations

To start building up your migration, simply modify the `up` method and start adding any of the available methods below to your migration.

A simple example would look like this for a new table:

```python
class MigrationForUsersTable(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.create("users") as table:
            table.increments('id')
            table.string('username')
            table.string('email').unique()
            table.string('password')
            table.boolean('is_admin')
            table.integer('age')

            table.timestamps()

    def down(self):
        """
        Revert the migrations.
        """
        self.schema.drop("users")
```

### Available Methods

| Command | Description |
| :--- | :--- |
| `table.string()` | The varchar version of the table. Can optional pass in a length `table.string('name', length=181)` |
| `table.char()` | CHAR equivalent column. |
| `table.text()` | TEXT equivalent column. |
| `table.longtext()` | LONGTEXT equivalent column. |
| `table.integer()` | The INT version of the database. Can also specify a length `table.integer('age', length=5)` |
| `table.unsigned_integer()` | UNSIGNED INT equivalent column. |
| `table.unsigned()` | Alias for `unsigned_integer` |
| `table.tiny_integer()` | TINY INT equivalent column. |
| `table.small_integer()` | SMALL INT equivalent column. |
| `table.medium_integer()` | MEDIUM INT equivalent column. |
| `table.big_integer()` | BIG INT equivalent column. |
| `table.increments()` | The auto incrementing version of the table. An unsigned non nullable auto incrementing integer. |
| `table.tiny_increments()` | TINY auto incrementing equivalent column. |
| `table.big_increments()` | An unsigned non nullable auto incrementing big integer. Use this if you expect the rows in a table to be very large |
| `table.binary()` | BINARY equivalent column. Sometimes is text field on unsupported databases. |
| `table.boolean()` | BOOLEAN equivalent column. |
| `table.json()` | JSON equivalent column. |
| `table.jsonb()` | LONGBLOB equivalent column. JSONB equivalent column for Postgres. |
| `table.date()` | DATE equivalent column. |
| `table.year()` | YEAR equivalent column. |
| `table.datetime()` | DATETIME equivalent column. |
| `table.timestamp()` | TIMESTAMP equivalent column. |
| `table.time()` | TIME equivalent column. |
| `table.timestamps()` | Creates `created_at` and `updated_at` columns on the table with the `timestamp` column and defaults to the current time. |
| `table.decimal()` | DECIMAL equivalent column. Can also specify the length and decimal position. `table.decimal('salary', 17, 6)` |
| `table.double()` | DOUBLE equivalent column. Can also specify a float length `table.double('salary', 17,6)` |
| `table.float()` | FLOAT equivalent column. |
| `table.enum()` | ENUM equivalent column. You can also specify available options as a list. `table.enum('flavor', ['chocolate', 'vanilla'])`. Sometimes defaults to a TEXT field with a constraint on unsupported databases. |
| `table.geometry()` | GEOMETRY equivalent column. |
| `table.point()` | POINT equivalent column. |
| `table.uuid()` | A CHAR column used to store UUIDs `table.uuid('id')`. Default length is 36. |
| `table.soft_deletes()` | A nullable DATETIME column named `deleted_at`. This is used by the [SoftDeletes](models.md#soft-deleting) scope. |


## Changes & Rolling Back Migrations

In addition to building up the migration, you should also build onto the `down` method which should reverse whatever was done in the `up` method. If you create a table in the up method, you should drop the table in the down method.

| Command                        | Description                                                                                                                                            |
| :----------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `table.drop_table()`           | DROP TABLE equivalent statement.                                                                                                                       |
| `table.drop_table_if_exists()` | DROP TABLE IF EXISTS equivalent statement.                                                                                                             |
| `table.drop_column()`          | DROP COLUMN equivalent statement. Can take one or multiple column names. `drop_column('column1', 'column2')`                                           |
| `table.drop_index()`           | Drops the constraint. Must pass in the name of the constraint. `drop_index('email_index')`                                                             |
| `table.drop_unique()`          | Drops the uniqueness constraint. Must pass in the name of the constraint. `table.drop_unique('users_email_unique')`                                    |
| `table.drop_foreign()`         | Drops the foreign key. Must specify the index name. `table.drop_foreign('users_article_id_foreign')`                                                   |
| `table.rename()`               | Renames a column to a new column. Must take the old column name, new column and data type. `table.rename("user_id", "profile_id", "unsigned_integer")` |
| `table.drop_primary()`         | Drops the primary key constraint. Must pass in the constraint name `table.drop_primary('users_id_primary')`                                            |

## Getting Migration Status

At any time you can get the migrations that have run or need to be ran:

```text
$ masonite-orm migrate:status
```

## Seeing Migration SQL Dumps

If you would like to see just the SQL that would run instead of running the actual migrations, you can specify the `-s` flag \(short for `--show`\). This works on the migrate and migrate:rollback commands.

```text
python craft migrate -s
```

## Refreshing Migrations

Refreshing a database is simply rolling back all migrations and then migrating again. This "refreshes" your database.

You can refresh by running the command:

```text
$ masonite-orm migrate:refresh
```

## Modifiers

In addition to the available columns you can use, you can also specify some modifers which will change the behavior of the column:

| Command           | Description                                                                                                    |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .nullable\(\)     | Allows NULL values to be inserted into the column.                                                             |
| .unique\(\)       | Forces all values in the column to be unique.                                                                  |
| .after\(other_column\)        | Adds the column after another column in the table. Can be used like `table.string('is_admin').after('email')`. |
| .unsigned\(\)     | Makes the column unsigned. Used with the `table.integer('age').unsigned()` column.                             |
| .use_current\(\)  | Makes the column use the `CURRENT_TIMESTAMP` modifer.                                                          |
| .default\(value\) | Specify a default value for the column. Can be used like table.boolean("is_admin").default(False)              |
| .primary\() | Specify that the column should be used for the primary key constraint. Used like `table.string('role_id').primary()`              |

## Indexes

In addition to columns, you can also create indexes. Below are the available indexes you can create:

| Command | Description |
| :--- | :--- |
| `table.primary(column)` | Creates a primary table constraint. Can pass multiple columns to create a composite key like `table.primary(['id', 'email'])`. Also supports a `name` parameter to specify the name of the index. |
| `table.unique(column)` | Makes a unique index. Can also pass multiple columns `table.unique(['email', 'phone_number'])`. Also supports a `name` parameter to specify the name of the index. |
| `table.index(column)` | Creates an index on the column. `table.index('email')`. Also supports a `name` parameter to specify the name of the index. |
| `table.fulltext(column)` | Creates an fulltext index on the column or columns. `table.fulltext('email')`. Note this only works for MySQL databases and will be ignored on other databases. Also supports a `name` parameter to specify the name of the index. |

> The default primary key is often set to an auto-incrementing integer, but you can [use a UUID instead](models.md#changing-primary-key-to-use-uuid).

## Foreign Keys

If you want to create a foreign key you can do so simply as well:

```python
table.foreign('local_column').references('other_column').on('other_table')
```

And optionally specify an `on_delete` or `on_update` method:

```python
table.foreign('local_column').references('other_column').on('other_table').on_update('set null')
```

You can use these options:

| Command                  | Description                                             |
| :----------------------- | :------------------------------------------------------ |
| .on_update\('set null'\) | Sets the ON UPDATE SET NULL property on the constraint. |
| .on_update\('cascade'\)  | Sets the ON UPDATE CASCADE property on the constraint.  |
| .on_delete\('set null'\) | Sets the ON DELETE SET NULL property on the constraint. |
| .on_delete\('cascade'\)  | Sets the ON DELETE CASCADE property on the constraint.  |


You can also pass a `name` parameter to change the name of the constraint:

```python
table.foreign('local_column', name="foreign_constraint").references('other_column').on('other_table')
```

You may also use a shorthand method:

```python
table.add_foreign('local_column.other_column.other_table', name="foreign_constraint")
```

## Changing Columns

If you would like to change a column you should simply specify the new column and then specify a `.change()` method on it.

Here is an example of changing an email field to a nullable field:

```python
class MigrationForUsersTable(Migration):
    def up(self):
        """
        Run the migrations.
        """
        with self.schema.table("users") as table:
            table.string('email').nullable().change()

        with self.schema.table("users") as table:
            table.string('email').unique()


    def down(self):
        """
        Revert the migrations.
        """
        pass
```

## Truncating

You can truncate a table:

```python
schema.truncate("users")
```

You can also temporarily disable foreign key checks and truncate a table:

```python
schema.truncate("users", foreign_keys=False)
```

## Dropping a Table

You can drop a table:

```python
schema.drop_table("users")
```

## Dropping a Table If It Exists

You can drop a table if it exists:

```python
schema.drop_table_if_exists("users")
```
