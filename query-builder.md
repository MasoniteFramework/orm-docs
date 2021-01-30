# Query builder

## Preface

The query builder is a class which is used to build up a query for execution later. For example if you need multiple wheres for a query you can chain them together on this `QueryBuilder` class. The class is then modified until you want to execute the query. Models use the query builder under the hood to make all of those calls. Many model methods actually return an instance of `QueryBuilder` so you can continue to chain complex queries together.

Using the query builder class directly allows you to make database calls without needing to use a model.

## Getting the QueryBuilder class

To get the query builder class you can simply import the query builder. Once imported you will need to pass the `connection_details` dictionary you store in your `config.database` file:

```python
from masoniteorm.query import QueryBuilder

builder = QueryBuilder().table("users")
```

You can also switch or specify connection on the fly using the `on` method:

```python
from masoniteorm.query import QueryBuilder

builder = QueryBuilder().on('staging').table("users")
```

> `from_("users")` is also a valid alias for the `table("users")` method. Feel free to use whatever you feel is more expressive.

You can then start making any number of database calls.

## Models

If you would like to use models you should reference the [Models](models.md) documentation. This is an example of using models directly with the query builder.

By default, the query builder will return dictionaries or lists depending on the result set. Here is an example of a result using only the query builder:

```python
# Without models
user = QueryBuilder().table("users").first()
# == {"id": 1, "name": "Joe" ...}

# With models
from masoniteorm.models import Model

class User(Model):
    pass

user = QueryBuilder(model=User).table("users").first()
# == <app.models.User>
```

## Fetching Records

### Select

```python
builder.table('users').select('username').get()
# SELECT `users`.`username` FROM `users`
```

You can also select a table and column:

```python
builder.table('users').select('profiles.name').get()
# SELECT `profiles`.`name` FROM `users`
```

You can also select a table and an asterisk (`*`). This is useful when doing joins:

```python
builder.table('users').select('profiles.*').get()
# SELECT `profiles`.* FROM `users`
```

Lastly you can also provide the column with an alias by adding `as` to the column select:

```python
builder.table('users').select('profiles.username as name').get()
# SELECT `profiles`.`username` AS name FROM `users`
```

### First

You can easily get the first record:

```python
builder.table('users').first()
# SELECT * from `users` LIMIT 1
```

### All Records

You can also simply fetch all records from a table:

```python
builder.table('users').all()
# SELECT * from `users`
```

### The Get Method

Once you start chaining methods you should call the `get()` method instead of the `all()` method to execute the query.

For example, this is correct:

```python
builder.table('users').select('username').get()
```

And this is wrong:

```python
builder.table('users').select('username').all()
```

### Wheres

You may also specify any one of these where statements:

The simplest one is a "where equals" statement. This is a query to get where `username` equals `Joe` AND `age` equals `18`:

```python
builder.table('users').where('username', 'Joe').where('age', 18).get()
```

You can also use a dictionary to build the where method:

```python
builder.table('users').where({"username": "Joe", "age": 18}).get()
```

You can also specify comparison operators:

```python
builder.table('users').where('age', '=', 18).get()
builder.table('users').where('age', '>', 18).get()
builder.table('users').where('age', '<', 18).get()
builder.table('users').where('age', '>=', 18).get()
builder.table('users').where('age', '<=', 18).get()
```

### Where Null

Another common where clause is checking where a value is `NULL`:

```python
builder.table('users').where_null('admin').get()
```

This will fetch all records where the admin column is `NULL`.

Or the inverse:

```python
builder.table('users').where_not_null('admin').get()
```

This selects all columns where admin is `NOT NULL`.

### Where In

In order to fetch all records within a certain list we can pass in a list:

```python
builder.table('users').where_in('age', [18,21,25]).get()
```

This will fetch all records where the age is either `18`, `21` or `25`.

### Where Like

You can do a WHERE LIKE or WHERE NOT LIKE query:

```python
builder.table('users').where_like('name', "Jo%").get()
builder.table('users').where_not_like('name', "Jo%").get()
```

### Subqueries

You can make subqueries easily by passing a callable into the where method:

```python
builder.table("users").where(lambda q: q.where("active", 1).where_null("activated_at")).get()
# SELECT * FROM "users" WHERE ("users"."active" = '1' AND "users"."activated_at" IS NULL)
```

You can also so a subquery for a `where_in` statement:

```python
builder.table("users").where_in("id", lambda q: q.select("profile_id").table("profiles")).get()
# SELECT * FROM "users" WHERE "id" IN (SELECT "profiles"."profile_id" FROM "profiles")
```

### Select Subqueries

You can make a subquery in the select clause. This takes 2 parameters. The first is the alias for the subquery and the second is a callable that takes a query builder.

```python
builder.table("stores").add_select("sales", lambda query: (
    query.count("*").from_("sales").where_column("sales.store_id", "stores.id")
)).order_by("sales", "desc")
```

This will add a subquery in the select part of the query. You can then order by or perform wheres on this alias.

Here is an example of all stores that make more than 1000 in sales:

```python
builder.table("stores").add_select("sales", lambda query: (
    query.count("*").from_("sales").where_column("sales.store_id", "stores.id")
)).where("sales", ">", "1000")
```

### Conditional Queries

Sometimes you need to specify conditional statements and run queries based on the conditional values.

For example you may have code that looks like this:

```python
def show(self, request: Request):
    age = request.input('age')
    article = Article.where('active', 1)
    if age >= 21:
        article.where('age_restricted', 1)
```

Instead of writing the code above you can use the `when` method. This method accepts a conditional as the first parameter and a callable as the second parameter. The code above would look like this:

```python
def show(self, request: Request):
    age = request.input('age')
    article = Article.where('active', 1).when(age >= 21, lambda q: q.where('age_restricted', 1))
```

If the conditional passed in the first parameter is not truthy then the second parameter will be ignored.

### Limits / Offsets

It's also very simple to use both limit and/or offset a query.

Here is an example of a limit:

```python
builder.table('users').limit(10).get()
```

Here is an example of an offset:

```python
builder.table('users').offset(10).get()
```

Or here is an example of using both:

```python
builder.table('users').limit(10).offset(10).get()
```

### Between

You may need to get all records where column values are between 2 values:

```python
builder.table('users').where_between('age', 18, 21).get()
```

### Group By

You may want to group by a specific column:

```python
builder.table('users').group_by('active').get()
```

You can also specify a multiple column group by:

```python
builder.table('users').group_by('active, name, is_admin').get()
```
### Group By Raw

You can also group by raw:

```python
builder.table('users').group_by_raw('COUNT(*)').get()
```

### Having

Having clauses are typically used during a group by. For example, returning all users grouped by salary where the salary is greater than 0:

```python
builder.table('users').sum('salary').group_by('salary').having('salary').get()
```

You may also specify the same query but where the sum of the salary is greater than 50,000

```python
builder.table('users').sum('salary').group_by('salary').having('salary', 50000).get()
```

### Inner Joining

Joining is a way to take data from related tables and return it in 1 result set as well as filter anything out that doesn't have a relationship on the joining tables.

```python
builder.table('users').join('table1', 'table2.id', '=', 'table1.table_id')
```

This join will create an inner join.

You can also choose a left join:

### Left Join

```python
builder.table('users').left_join('table1', 'table2.id', '=', 'table1.table_id')
```

and a right join:

### Right Join

```python
builder.table('users').right_join('table1', 'table2.id', '=', 'table1.table_id')
```

### Increment

There are times where you really just need to increment a column and don't need to pull any additional information. A lot of the incrementing logic is hidden away:

```python
builder.table('users').increment('status')
```

Decrementing is also similiar:

### Decrement

```python
builder.table('users').decrement('status')
```

You also pass a second parameter for the number to increment the column by.

```python
builder.table('users').increment('status', 10)
builder.table('users').decrement('status', 10)
```

## Aggregates

There are several aggregating methods you can use to aggregate columns:

### Sum

```python
builder.table('users').sum('salary').get()
```

### Average

```python
builder.table('users').avg('salary').get()
```

### Count

```python
builder.table('users').count('salary').get()
```

### Max

```python
builder.table('users').max('salary').get()
```

### Min

```python
builder.table('users').min('salary').get()
```

### Aliases

You may also specify an alias for your aggregate expressions. You can do this by adding "as {alias}" to your aggregate expression:

```python
builder.table('users').sum('salary as payments').get()
#== SELECT SUM(`users`.`salary`) as payments FROM `users`
```

## Order By

You can easily order by:

```python
builder.order_by("column")
```

The default is ascending order but you can change directions:

```python
builder.order_by("column", "desc")
```

You can also specify a comma separated list of columns to order by all 3 columns:

```python
builder.order_by("name, email, active")
```

You may also specify the sort direction on each one individually:

```python
builder.order_by("name, email desc, active")
```

This will sort `name` and `active` in ascending order because it is the default but will sort email in descending order.

These 2 peices of code are the same:

```python
builder.order_by("name, active").order_by("name", "desc")
builder.order_by("name, email desc, active")
```

## Order By Raw

You can also order by raw. This will pass your raw query directly to the query:

```python
builder.order_by_raw("name asc")
```

## Creating Records

You can create records by passing a dictionary to the `create` method. This will perform an INSERT query:

```python
builder.create({"name": "Joe", "active": 1})
```

## Bulk Creating

You can also bulk create records by passing a list of dictionaries:

```python
builder.bulk_create([
    {"name": "Joe", "active": 1},
    {"name": "John", "active": 0},
    {"name": "Bill", "active": 1},
])
```

## Raw Queries

If some queries would be easier written raw you can easily do so for both selects and wheres:

```python
builder.table('users').select_raw("COUNT(`username`) as username").where_raw("`username` = 'Joe'").get()
```

You can also specify a fully raw query using the `statement` method. This will simply execute a query directly and return the result rather than building up a query:

```python
builder.statement("select count(*) from users where active = 1")
```

You can also pass query bindings as well:

```python
builder.statement("select count(*) from users where active = ?", [1])
```

## Chunking

If you need to loop over a lot of results then consider chunking. A chunk will only pull in the specified number of records into a generator:

```python
for users in builder.table('users').chunk(100):
    for user in users:
        user #== <User object>
```

## Getting SQL

If you want to find out the SQL that will run when the command is executed. You can use `to_sql()`. This method returns the full query and is not the query that gets sent to the database. The query sent to the database is a "qmark query". This `to_sql()` method is mainly for debugging purposes.

See the section below for more information on qmark queries.

```python
builder.table('users').count('salary').to_sql()
#== SELECT COUNT(`users`.`salary`) FROM `users`
```

## Getting Qmark

Qmark is essentially just a normal SQL statement except the query is replaced with question marks. The values that should have been in the position of the question marks are stored in a tuple and sent along with the qmark query to help in sql injection. The qmark query is the actual query sent using the connection class.

```python
builder.table('users').count('salary').where('age', 18).to_qmark()
#== SELECT COUNT(`users`.`salary`) FROM `users` WHERE `users`.`age` = '?'
```

## Updates

### Updating Records

You can update many records.

```python
builder.where('active', 0).update({
    'active': 1
})
# UPDATE `users` SET `users`.`active` = 1 where `users`.`active` = 0
```

## Deletes

### Deleting Records

You can delete many records as well. For example, deleting all records where active is set to 0.

```python
builder.where('active', 0).delete()
```

## Available Methods

|  |  |  |
| :--- | :--- | :--- |
| aggregate | all | between |
| count | create | decrement |
| delete | first | get |
| group\_by | having | increment |
| join | left\_join | limit |
| max | not\_between | offset |
| order\_by | right\_join | select |
| select\_raw | sum | to\_qmark |
| to\_sql | update | where |
| where\_column | where\_exists | where\_has |
| where\_in | where\_not\_in | where\_not\_null |
| where\_null | where\_raw |  |

