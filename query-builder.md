# Preface

The query builder is a class which is used to build up a query for execution later. For example if you need multiple wheres for a query you can chain them together on this `QueryBuilder` class. The class is then modified until you want to execute the query. Models use the query builder under the hood to make all of those calls. Many model methods actually return an instance of `QueryBuilder` so you can continue to chain complex queries together.

Using the query builder class directly allows you to make database calls without needing to use a model.

# Getting the QueryBuilder class

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

# Models

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

# Fetching Records

## Select

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

## First

You can easily get the first record:

```python
builder.table('users').first()
# SELECT * from `users` LIMIT 1
```

## All Records

You can also simply fetch all records from a table:

```python
builder.table('users').all()
# SELECT * from `users`
```

## The Get Method

Once you start chaining methods you should call the `get()` method instead of the `all()` method to execute the query.

For example, this is correct:

```python
builder.table('users').select('username').get()
```

And this is wrong:

```python
builder.table('users').select('username').all()
```

## Wheres

You may also specify any one of these where statements:

The simplest one is a "where equals" statement. This is a query to get where `username` equals `Joe` AND `age` equals `18`:

```python
builder.table('users').where('username', 'Joe').where('age', 18).get()
```

You can also use a dictionary to build the where method:

```python
builder.table('users').where({"username": "Joe", "age": 18}).get()
```

You can also specify different comparison operators:

```python
builder.table('users').where('age', '=', 18).get()
builder.table('users').where('age', '>', 18).get()
builder.table('users').where('age', '<', 18).get()
builder.table('users').where('age', '>=', 18).get()
builder.table('users').where('age', '<=', 18).get()
builder.table('users').where('age', 'regexp', r"[0-9]").get()
builder.table('users').where('age', 'not regexp', r"[0-9]").get()
```

## Where Null

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

## Where In

In order to fetch all records within a certain list we can pass in a list:

```python
builder.table('users').where_in('age', [18,21,25]).get()
```

This will fetch all records where the age is either `18`, `21` or `25`.

## Where Like

You can do a WHERE LIKE or WHERE NOT LIKE query:

```python
builder.table('users').where_like('name', "Jo%").get()
builder.table('users').where_not_like('name', "Jo%").get()
```

## Where Subqueries

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

## Select Subqueries

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

## Conditional Queries

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

## Limits / Offsets

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

## Between

You may need to get all records where column values are between 2 values:

```python
builder.table('users').where_between('age', 18, 21).get()
```

## Group By

You may want to group by a specific column:

```python
builder.table('users').group_by('active').get()
```

You can also specify a multiple column group by:

```python
builder.table('users').group_by('active, name, is_admin').get()
```
## Group By Raw

You can also group by raw:

```python
builder.table('users').group_by_raw('COUNT(*)').get()
```

## Having

Having clauses are typically used during a group by. For example, returning all users grouped by salary where the salary is greater than 0:

```python
builder.table('users').sum('salary').group_by('salary').having('salary').get()
```

You may also specify the same query but where the sum of the salary is greater than 50,000

```python
builder.table('users').sum('salary').group_by('salary').having('salary', 50000).get()
```


## Joining

Creating join queries is very simple. 

```python
builder.join('other_table', 'column1', '=', 'column2')
```

This will build a `JoinClause` behind the scenes for you.
## Advanced Joins

Advanced joins are for use cases where you need to compile a join clause that is more than just joining on 2 distant columns. Advanced joins are where you need additional `on` or `where statements`.There are currently 2 ways to perform an advanced where clause. 

The first way is that you may create your own `JoinClause` from scratch and build up your own clause:

```python
from masoniteorm.expressions import JoinClause

clause = (
    JoinClause('other_table as ot')
    .on('column1', '=', 'column2')
    .on('column3', '=', 'column4')
    .where('column3', '>', 4)
)

builder.join(clause)
```

The second way is passing a "lambda" to the join method directly which will return you a `JoinClause` class you can build up. This way is a bit more cleaner:

```python
builder.join('other_table as ot', lambda join: (
    (
        join.on('column1', '=', 'column2')
        .on('column3', '=', 'column4')
        .where('column3', '>', 4)
    )
))
```

## Left Join

```python
builder.table('users').left_join('table1', 'table2.id', '=', 'table1.table_id')
```

and a right join:

## Right Join

```python
builder.table('users').right_join('table1', 'table2.id', '=', 'table1.table_id')
```

## Increment

There are times where you really just need to increment a column and don't need to pull any additional information. A lot of the incrementing logic is hidden away:

```python
builder.table('users').increment('status')
```

Decrementing is also similiar:

## Decrement

```python
builder.table('users').decrement('status')
```

You also pass a second parameter for the number to increment the column by.

```python
builder.table('users').increment('status', 10)
builder.table('users').decrement('status', 10)
```

# Pagination

Sometimes you'll want to paginate through a result set. There are 2 ways to pagainate records.

The first is a "length aware" pagination. This means that there will be additional results on the pagination like the total records. This will do 2 queries. The initial query to get the records and a COUNT query to get the total. For large or complex result sets this may not be the best choice as 2 queries will need to be made.

```python
builder.table("users").where("active", 1).paginate(number_of_results, page)
```

You may also do "simple pagination". This will not give you back a query total and will not make the second COUNT query.

```python
builder.table("users").where("active", 1).simple_paginate(number_of_results, page)
```

# Aggregates

There are several aggregating methods you can use to aggregate columns:

## Sum

```python
salary = builder.table('users').sum('salary').first().salary
```

Notice the alias for the aggregate is the name of the column.

## Average

```python
salary = builder.table('users').avg('salary').first().salary
```

Notice the alias for the aggregate is the name of the column.

## Count

```python
salary = builder.table('users').count('salary').first().salary
```

You can also count all:

```python
salary = builder.table('users').count('salary').first().salary
```

## Max

```python
salary = builder.table('users').max('salary').first().salary
```

## Min

```python
salary = builder.table('users').min('salary').first().salary
```

## Aliases

You may also specify an alias for your aggregate expressions. You can do this by adding "as {alias}" to your aggregate expression:

```python
builder.table('users').sum('salary as payments').get()
#== SELECT SUM(`users`.`salary`) as payments FROM `users`
```

# Order By

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

# Order By Raw

You can also order by raw. This will pass your raw query directly to the query:

```python
builder.order_by_raw("name asc")
```

# Creating Records

You can create records by passing a dictionary to the `create` method. This will perform an INSERT query:

```python
builder.create({"name": "Joe", "active": 1})
```

# Bulk Creating

You can also bulk create records by passing a list of dictionaries:

```python
builder.bulk_create([
    {"name": "Joe", "active": 1},
    {"name": "John", "active": 0},
    {"name": "Bill", "active": 1},
])
```

# Raw Queries

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
builder.statement("select count(*) from users where active = '?'", [1])
```

You can also use the `Raw` expression class to specify a raw expression. This can be used with the update query:

```python
from masoniteorm.expressions import Raw

builder.update({
    "name": Raw('"alias"')
})
# == UPDATE "users" SET "name" = "alias"
```

You can also query using having raw. This will pass your raw query directly to the query:

```python
builder.having_raw("age > 18")
```

# Chunking

If you need to loop over a lot of results then consider chunking. A chunk will only pull in the specified number of records into a generator:

```python
for users in builder.table('users').chunk(100):
    for user in users:
        user #== <User object>
```

# Getting SQL

If you want to find out the SQL that will run when the command is executed. You can use `to_sql()`. This method returns the full query without bindings. The actual query sent to the database is a "qmark query" (see below). This `to_sql()` method is mainly for debugging purposes and should not be sent directly to a database as the result with have no query bindings and will be subject to SQL injection attacks. **Use this method for debugging purposes only.**

```python
builder.table('users').count('salary').where('age', 18).to_sql()
#== SELECT COUNT(`users`.`salary`) AS salary FROM `users` WHERE `users`.`age` = '18'
```

# Getting Qmark

Qmark is essentially just a normal SQL statement except that the query is replaced with quoted question marks (`'?'`). The values that should have been in the position of the question marks are stored in a tuple and sent along with the qmark query to help in sql injection. The qmark query is the actual query sent using the connection class.

```python
builder.table('users').count('salary').where('age', 18).to_qmark()
#== SELECT COUNT(`users`.`salary`) AS salary FROM `users` WHERE `users`.`age` = '?'
```

> Note: qmark queries will reset the query builder and remove things like aggregates and wheres from the builder class. Because of this, writing `get()` after `to_qmark` will result in incorrect queries (because things like wheres and aggregates will be missing from the final query). If you need to debug a query, please use the `to_sql()` method which does not have this kind of resetting behavior.

# Updates

## Updating Records

You can update many records.

```python
builder.where('active', 0).update({
    'active': 1
})
# UPDATE `users` SET `users`.`active` = 1 where `users`.`active` = 0
```

# Deletes

## Deleting Records

You can delete many records as well. For example, deleting all records where active is set to 0.

```python
builder.where('active', 0).delete()
```

# Truncating

You can also truncate directly from the query builder:

```python
builder.truncate('users')
```

You may also temporarily disable and re-enable foreign keys to avoid foreign key checks.

```python
builder.truncate('users', foreign_keys=True)
```

# Available Methods

# Aggregates

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .avg\('column') | Gets the average of a column. Can also use an `as` modifier to alias the `.avg('column as alias')`. |
| .sum\('column') | Gets the sum of a column. Can also use an `as` modifier to alias the `.sum('column as alias')`. |
| .count\('column') | Gets the count of a column. Can also use an `as` modifier to alias the `.count('column as alias')`. |
| .max\('column') | Gets the max value of a column. Can also use an `as` modifier to alias the `.max('column as alias')`. |
| .min\('column') | Gets the min value of a column. Can also use an `as` modifier to alias the `.min('column as alias')`. |

# Joins

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .join('table1', 'table2.id', '=', 'table1.table_id') | Joins 2 tables together. This will do an INNER join. Can control which join is performed using the `clause` parmeter. Can choose `inner`, `left` or `right`. |
| .left_join('table1', 'table2.id', '=', 'table1.table_id') | Joins 2 tables together. This will do an LEFT join. |
| .right_join\('table1', 'table2.id', '=', 'table1.table_id') | Joins 2 tables together. This will do an RIGHT join. |

# Where Clauses

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .between('column', 'value') | Peforms a BETWEEN clause. |
| .between('column', 'value') | Peforms a BETWEEN clause. |
| .not_between('column', 'value') | Peforms a NOT BETWEEN clause. |
| .where('column', 'value') | Peforms a WHERE clause. Can optionally choose a logical operator to use `.where('column', '=', 'value')`. Logical operators available include: `<`, `>`, `>=`, `<=`, `!=`, `=`, `like`, `not like` |
| .or_where('column', 'value') | Peforms a OR WHERE clause. Can optionally choose a logical operator to use `.where('column', '=', 'value')`. Logical operators available include: `<`, `>`, `>=`, `<=`, `!=`, `=`, `like`, `not like` |
| .where_like('column', 'value') | Peforms a WHERE LIKE clause. |
| .where_not_like('column', 'value') | Peforms a WHERE NOT LIKE clause. |
| .where_exists(lambda q: q.where(..)) | Peforms an EXISTS clause. Takes a lambda expression to indicate which subquery should generate. |
| .where_not_exists(lambda q: q.where(..)) | Peforms a NOT EXISTS clause. Takes a lambda expression to indicate which subquery should generate. |
| .where_column('column1', 'column2') | Peforms a comparison between 2 columns. Logical operators available include: `<`, `>`, `>=`, `<=`, `!=`, `=` |
| .where_in('column1', [1,2,3]) | Peforms a WHERE IN clause. Second parameter needs to be a list or collection of values. |
| .where_not_in('column1', [1,2,3]) | Peforms a WHERE NOT IN clause. Second parameter needs to be a list or collection of values. |
| .where_null('column1') | Peforms a WHERE NULL clause.  |
| .where_not_null('column1') | Peforms a WHERE NOT NULL clause. |

# Pessimistic Locking

The query builder includes a few functions to help you do “pessimistic locking” on your SELECT statements.

To run the SELECT statement with a “shared lock”, you may use the shared_lock method on a query:

```python
builder.where('votes', '>', 100).shared_lock().get()
```

To “lock for update” on a SELECT statement, you may use the lock_for_update method on a query:

```python
builder.where('votes', '>', 100).lock_for_update().get()
```

# Raw Queries

| Method | Description  |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .select_raw('SUM("column")') | specifies a raw string where the select expression would go. |
| .where_raw('SUM("column")') | specifies a raw string where the WHERE expression would go. |
| .having_raw('SUM("column") > 10') | specifies a raw string where the HAVING expression would go. |
| .order_by_raw('column1, column2') | specifies a raw string where the ORDER BY expression would go. |
| .group_by_raw('column1, column2') | specifies a raw string where the GROUP BY expression would go. |

# Modifiers

| Method | Description  |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .limit('10') | Limits the results to 10 rows |
| .offset(10) | Offsets the results by 10 rows |
| .take(10) | Alias for the `limit` method |
| .skip(10) | Alias for the `offset` method |
| .group_by('column') | Adds a GROUP BY clause. |
| .having('column') | Adds a HAVING clause. |
| .increment('column') | Increments the column by 1. Can pass in a second parameter for the number to increment by. `.increment('column', 100)`. |
| .decrement('column') | Decrements the column by 1. Can pass in a second parameter for the number to increment by. `.decrement('column', 100)`. |


# DML

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .add_select("alias", lambda q: q.where(..)) | Performs a SELECT subquery expession.  |
| .all() | Gets all records. |
| .chunk(100) | Chunks a result set. Uses a generator to keep each chunk small. Useful for chunking large data sets where pulling too many results in memory will overload the application  |
| .create({}) | Limits the results to 10 rows. Must take a dictionary of values. |
| .delete() | Performs a DELETE query based on the current clauses already chained onto the query builder. |
| .first() | Gets the first record |
| .from_('users') | Sets the table. |
| .get() | Gets all records. Used in combination with other builder methods to finally execute the query. |
| .last() | Gets the last record |
| .paginate(limit, page) | Paginates a result set. Pass in different pages to get different results. This a length aware pagination. This will perform a COUNT query in addition to the original query. Could be slower on larger data sets. |
| .select('column') | Offsets the results by 10 rows. Can use the `as` keyword to alias the column. `.select('column as alias')` |
| .simple_paginate(limit, page) | Paginates a result set. Pass in different pages to get different results. This not a length aware pagination. The result will not contain the total result counts  |
| .statement("select * from users") | Performs a raw query.  |
| .table('users') | Alias for the `from_` method. |
| .truncate('table') | Truncates a table. Can pass a second parameter to disable and enable foreign key constraints. `truncate('table', foreign_keys=True)` |
| .update({}) | dictionary values to update the record with.  |

# Testing

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .to_sql() | Returns a string of the fully compiled SQL to be generated.  |
| .to_qmark('') | Returns a string of the  SQL to generated but with `?` values where the sql bindings are placed. Also resets the query builder instance. |

# Low Level Methods

These are lower level methods that may be useful:

| Method | Description |
| :---------------- | :------------------------------------------------------------------------------------------------------------- |
| .new() | Creates a new clean builder instance. This instance does not have any clauses, selects, limits, etc from the original builder instance. Great for performing subqueries |
| .where_from_builder() | Creates a WHERE clause from a builder instance. |
| .get_table_name() | Gets the tables name. |


