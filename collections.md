# Collections

Anytime your results return multiple values then an instance of `Collection` is returned. This allows you to iterate over your values and has a lot of shorthand methods.

When using collections as a query result you can iterate over it as if the collection with a normal list:

```python
users = User.get() #== <masoniteorm.collections.Collection>
users.count() #== 50
users.pluck('email') #== <masoniteorm.collections.Collection> of emails

for user in users:
  user.email #== 'joe@masoniteproject.com'
```

## Available Methods

|  |  |  |
| :--- | :--- | :--- |
| all | avg | chunk |
| collapse | contains | count |
| diff | each | every |
| filter | first | flatten |
| for\_page | forget | get |
| group\_by | implode | is\_empty |
| last | map\_into | map |
| max | merge | pluck |
| pop | prepend | pull |
| push | put | [random](/#random) |
| reduce | reject | reverse |
| serialize | shift | sort |
| sum | take | to\_json |
| transform | unique | where |
| zip |


## random
The `random` method returns a random item from the collection
```python
user = User.all().random() #== returns a random User instance
```

An integer count can be given to `random` method to specify how many items you would like to randomly retrieve from the collection. A collection will always be returned when the items count is specified

```python
users = User.all().random(3) #== returns a Collection of 3 users
users.count() #== 3
users.all() #== returns a list of 3 users
```

If the collection length is smaller than specified count a `ValueError` will be raised.