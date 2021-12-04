# Installation

Setting up Masonite is extremely simple.

If you are using the Masonite web framework than all the installation is setup for you. If you are using anything other than Masonite or building your own Python application then be sure to follow the install steps below:

## Pip install

First install via pip:

```text
$ pip install masonite-orm
```

## Configuration File

To start configuring your project you'll need a `config/database.py` file. In this file we will be able to put all our connection information.

One we have our `config/database.py` file we can put a `DATABASES` variable with a dictionary of connection details. Each key will be the name of our connection. The connection name can be whatever you like and does not need to relate to a database name. Common connection names could be something like `dev`, `prod` and `staging`. Feel free to name these connections whatever you like.

The connection variable will look something like this

```python
# config/database.py
DATABASES = {
  "default": "mysql",
  "mysql": {
    "host": "127.0.0.1",
    "driver": "mysql",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 3306,
    "log_queries": False,
    "options": {
      #  
    }
  },  
  "postgres": {
    "host": "127.0.0.1",
    "driver": "postgres",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 5432,
    "log_queries": False,
    "options": {
      #  
    }
  },
  "sqlite": {
    "driver": "sqlite",
    "database": "masonite.sqlite3",
  }
}
```

Lastly you will need to import the `ConnectionResolver` class and and register the connection details. Normal convention is to set this to a variable called `DB`:

```python
# config/database.py
from masoniteorm.connections import ConnectionResolver

DATABASES = {
  # ...
}

DB = ConnectionResolver().set_connection_details(DATABASES)
```

After this you have successfully setup Masonite ORM in your project!

## MSSQL

Masonite ORM supports Microsoft SQL Server and several options to modify the connection string. All available options are:

```python
"mssql": {
    "host": "127.0.0.1",
    "driver": "mssql",
    "database": "masonite",
    "user": "root",
    "password": "",
    "port": 1433,
    "log_queries": False,
    "options": {
      "trusted_connection": "Yes",
      "integrated_security": "sspi",
      "instance": "SQLExpress",
      "authentication": "ActiveDirectoryPassword",
      "driver": "ODBC Driver 17 for SQL Server",
      "connection_timeout": 15,
    }
  },

```

## Transactions

You can use global level database transactions easily by importing the connection resolver class:

```python
from config.database import DB

DB.begin_transaction()
User.create({..})
```

You can then either rollback or commit the transactions:

```python
DB.commit()
DB.rollback()
```

You may also optionally pass the connection you'd like to use:

```python
DB.begin_transaction("staging")
DB.commit("staging")
DB.rollback("staging")
```

You can also use the transaction as a context manager:

```python
with DB.transaction():
  User.create({..})
```

If there are any exceptions in inside the context then the transaction will be rolled back. Else it will commit the transaction.

## Logging

If you would like, you can log any queries Masonite ORM generates to any supported Python logging handler. First you need to enable logging
in `config/database.py` file through the `log_queries` boolean parameter.

Inside your `config/database.py` file you can put on the bottom here. The StreamHandler will output the queries to the terminal.

```python
logger = logging.getLogger('masoniteorm.connection.queries')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()

logger.addHandler(handler)
```

You can specify as many handlers as you would like. Here's an example of logging to both the terminal and a file:

```python
logger = logging.getLogger('masoniteorm.connection.queries')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
file_handler = logging.FileHandler('queries.log')

logger.addHandler(handler)
logger.addHandler(file_handler)
```

## Raw Queries

You can query the database directly using the connection resolver class. If you set the connection resolver to the variable `DB` you can import it like:

```python
from config.database import DB

result = DB.statement("select * from users where users.active = 1")
```

You may also pass query bindings as well to protect against SQL injection by passing a list of bindings:

```python
from config.database import DB

result = DB.statement("select * from users where users.active = '?'", [1])
```

This will use the default connection but you may also optionally pass a connection to use:

```python
from config.database import DB

result = DB.statement("select * from users where users.active = '?'", [1], connection="production")
```
