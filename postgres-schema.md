Masonite ORM supports setting the schema on different classes to change which Postgres schema is called on different actions such as queries and migrations. 

> Setting schemas currently only works for the Postgres driver

# Models

On model calls you can set the schema which will pass to the query builder:

```python
User.set_schema("schema2").where(..).get()
```

# Migrations

The migration command will take a `--schema` option to change the schema to run the migrations for. This option is available on all the migration commands.

> If using Masonite you will use the `python craft` command instead of the `masonite-orm` command.

```
$ masonite-orm migrate --schema schema2
```

# Connection Settings

On the postgres connection settings you can set the schema on the `schema` key:

```python
"postgres": {
    "driver": "postgres",
    "host": "...",
    "user": "..",
    "password": "...",
    "schema": "schema2"
},
```

This will set the default schema for the postgres connection
