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

|                          |                        |                        |
| :----------------------- | :--------------------- | :--------------------- |
| [all](/#all)             | [avg](/#avg)           | [chunk](/#chunk)       |
| [collapse](/#collapse)   | [contains](/#contains) | [count](/#count)       |
| [diff](/#diff)           | [each](/#each)         | [every](/#every)       |
| [filter](/#filter)       | [first](/#first)       | [flatten](/#flatten)   |
| [for_page](/#for_page)   | [forget](/#forget)     | [get](/#get)           |
| [group_by](/#group_by)   | [implode](/#implode)   | [is_empty](/#is_empty) |
| [last](/#last)           | [map_into](/#map_into) | [map](/#map)           |
| [max](/#max)             | [merge](/#merge)       | [pluck](/#pluck)       |
| [pop](/#pop)             | [prepend](/#prepend)   | [pull](/#pull)         |
| [push](/#push)           | [put](/#put)           | [random](/#random)     |
| [reduce](/#reduce)       | [reject](/#reject)     | [reverse](/#reverse)   |
| [serialize](/#serialize) | [shift](/#shift)       | [sort](/#sort)         |
| [sum](/#sum)             | [take](/#take)         | [to_json](/#to_json)   |
| [transform](/#transform) | [unique](/#unique)     | [where](/#where)       |
| [zip](/#zip)             |

### all

Returns the underlying list or dict represented by the collection:

```python
users = User.get().all() #== [<app.User.User>, <app.User.User>]

Collection([1, 2, 3]).all() #== [1, 2, 3]
```

### avg

Returns the average of all items in the collection:

```python
Collection([1, 2, 3, 4, 5]).avg() #== 3
```

If the collection contains nested objects or dictionaries (e.g. for a collection of models), you must pass a key to use for determining which values to calculate the average:

```python
average_price = Product.get().avg('price')
```

### chunk

Chunks a collection into multiple, smaller collections of a given size. Uses a generator to keep each chunk small. Useful for chunking large data sets where pulling too many results in memory will overload the application.

```python
collection = Collection([1, 2, 3, 4, 5, 6, 7])
chunks = collection.chunk(2).serialize() #== [[1, 2], [3, 4], [5, 6], [7]]
```

### collapse

Collapses a collection of lists into a flat collection:

```python
collection = Collection([[1, 2, 3], [4, 5, 6])
collection.collapse().serialize() #== [1, 2, 3, 4, 5, 6]
```

### contains

Determines whether the collection contains a given item:

```python
collection = Collection(['foo', 'bar'])
collection.contains('foo') #== True
```

You can also pass a key / value pair to the contains method, which will determine if the given pair exists in the collection.

Finally, you may also pass a callback to the contains method to perform your own truth test:

```python
collection = Collection([1, 2, 3, 4, 5])
collection.contains(lambda item: item > 5) #== False
```

### count

Returns the total number of items in the collection. `len()` standard python method can also be used.

### diff

Returns the difference as a collection against another collection

```python
collection = Collection([1, 2, 3, 4, 5])
diff = collection.diff([2, 4, 6, 8])
diff.all() #== [1, 3, 5]
```

### each

Iterates over the items in the collection and passes each item to a given callback:

```python
posts.each(lambda post: post.author().save(author))
```

### every

Creates a new collection by applying a given callback on every element:

```python
collection = Collection([1, 2, 3])
collection.every(lambda x: x*2 ).all() #== [2, 4, 6]
```

### filter

Filters the collection by a given callback, keeping only those items that pass a given truth test:

```python
collection = Collection([1, 2, 3, 4])
filtered = collection.filter(lambda item: item > 2)
filtered.all() #== [3, 4]
```

### first

Returns the first item of the collection, if no arguments are given.

When given a truth test as callback, it returns the first element in the collection that passes the test:

```python
collection = Collection([1, 2, 3, 4])
collection.first(lambda item: item > 2)
```

### flatten

Flattens a multi-dimensional collection into a single dimension:

```python
collection = Collection([1, 2, [3, 4, 5, {'foo': 'bar'}]])
flattened = collection.flatten().all() #== [1, 2, 3, 4, 5, 'bar']
```

### forget

Removes an item from the collection by its key:

```python
collection = Collection([1, 2, 3, 4, 5])
collection.forget(1).all() #== [1,3,4,5]
collection.forget(0,2).all() #== [3,5]
```

Unlike most other collection methods, `forget` does not return a new modified collection; it modifies the collection it is called on.

### for_page

Paginates the collection by returning a new collection containing the items that would be present on a given page number:

```python
collection = Collection([1, 2, 3, 4, 5, 6, 7, 8, 9])
chunk = collection.for_page(2, 4).all() #== 4, 5, 6, 7
```

`for_page(page, count)` takes the page number and the number of items to show per page.

### get

Returns the item at a given key or index. If the key does not exist, None is returned. An optional default value can be passed as the second argument:

```python
collection = Collection([1, 2, 3])
collection.get(0) #== 1
collection.get(4) #== None
collection.get(4, 'default') #== 'default'

collection = Collection({"apples": 1, "cherries": 2})
collection.get("apples") #== 1
```

### group_by

Returns a collection where items are grouped by the given key:

```python
collection = Collection([
  {"id": 1, "type": "a"},
  {"id": 2, "type": "b"},
  {"id": 3, "type": "a"}
])
collection.implode("type").all()
#== {'a': [{'id': 1, 'type': 'a'}, {'id': 4, 'type': 'a'}],
#    'b': [{'id': 2, 'type': 'b'}]}
```

### implode

Joins the items in a collection with `,` or the given _glue_ string.

```python
collection = Collection(['foo', 'bar', 'baz'])
collection.implode() #== foo,bar,baz
collection.implode('-') #== foo-bar-baz
```

If the collection contains dictionaries or objects, you must pass the key of the attributes you wish to join:

```python
collection = Collection([
    {'account_id': 1, 'product': 'Desk'},
    {'account_id': 2, 'product': 'Chair'}
])
collection.implode(key='product') #== Desk,Chair
collection.implode(" - ", key='product') #== Desk - Chair
```

### is_empty

Returns `True` if the collection is empty; otherwise, `False` is returned:

```python
Collection([]).is_empty() #== True
```

### last

Returns the last element in the collection if no arguments are given.

Returns the last element in the collection that passes the given truth test:

```python
collection = Collection([1, 2, 3, 4])
last = collection.last(lambda item: item < 3) #== 2
```

### map

Iterates through the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a **new** collection of modified items:

```python
collection = Collection([1, 2, 3, 4])
multiplied = collection.map(lambda item: item * 2).all() #== [2, 4, 6, 8]
```

If you want to transform the original collection, use the [transform](/#transform) method.

### map_into

Iterates through the collection and cast each value into the given class:

```python
collection = Collection([1,2])
collection.map_into(str).all() #== ["1", "2"]
```

A class method can also be specified. Some additional keywords arguments can be passed to this method:

```python
class Point:
    @classmethod
    def as_dict(cls, coords, one_dim=False):
        if one_dim:
            return {"X": coords[0]}
        return {"X": coords[0], "Y": coords[1]}

collection = Collection([(1,2), (3,4)])
collection.map_into(Point, "as_dict") #== [{'X': 1, 'Y': 2}, {'X': 3, 'Y': 4}]
collection.map_into(Point, "as_dict", one_dim=True) #== [{'X': 1}, {'X': 3}]
```

### max

Retrieves max value of the collection:

```python
collection = Collection([1,2,3])
collection.max() #== 3
```

If the collection contains dictionaries or objects, you must pass the key on which to compute max value:

```python
collection = Collection([
    {'product_id': 1, 'product': 'Desk'},
    {'product_id': 2, 'product': 'Chair'}
    {'product_id': 3, 'product': 'Table'}
])
collection.max("product_id") #== 3
```

### merge

Merges the given list into the collection:

```python
collection = Collection(['Desk', 'Chair'])
collection.merge(['Bookcase', 'Door'])
collection.all() #== ['Desk', 'Chair', 'Bookcase', 'Door']
```

Unlike most other collection methods, `merge` does not return a new modified collection; it modifies the collection it is called on.

### pluck

Retrieves all of the collection values for a given key:

```python
collection = Collection([
    {'product_id': 1, 'product': 'Desk'},
    {'product_id': 2, 'product': 'Chair'}
    {'product_id': 3, 'product': None}
])

plucked = collection.pluck('product').all() #== ['Desk', 'Chair', None]
```

A key can be given to pluck the collection into a dictionary with the given key

```python
collection.pluck("product", "product_id") #== {1: 'Desk', 2: 'Chair', 3: None}
```

You can pass `keep_nulls=False` to remove `None` value in the collection.

```python
collection.pluck("product", keep_nulls=False) #== ['Desk', 'Chair']
```

### pop

Removes and returns the last item from the collection:

```python
collection = Collection([1, 2, 3, 4, 5])
collection.pop() #== 5
collection.all() #== [1, 2, 3, 4]
```

### prepend

Adds an item to the beginning of the collection:

```python
collection = Collection([1, 2, 3, 4])
collection.prepend(0)
collection.all() #== [0, 1, 2, 3, 4]
```

### pull

Removes and returns an item from the collection by its key:

```python
collection = Collection([1, 2, 3, 4])
collection.pull(1) #== 2
collection.all() #== [1, 3, 4]

collection = Collection({'apple': 1, 'cherry': 3, 'lemon': 2})
collection.pull('cherry') #== 3
collection.all() #== {'apple': 1, 'lemon': 2}
```

### push

Appends an item to the end of the collection:

```python
collection = Collection([1, 2, 3, 4])
collection.push(5)
collection.all() #== [1, 2, 3, 4, 5]
```

### put

Sets the given key and value in the collection:

```python
collection = Collection([1, 2, 3, 4])
collection.put(1, 5)
collection.all() #== [1, 5, 3, 4]

collection = Collection({'apple': 1, 'cherry': 3, 'lemon': 2})
collection.put('cherry', 0)
collection.all() #== {'apple': 1, 'cherry': 0, 'lemon': 2}
```

### random

Returns a random item from the collection

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

### reduce

Reduces the collection to a single value, passing the result of each iteration into the subsequent iteration.

```python
collection = Collection([1, 2, 3])
collection.reduce(lambda result, item: (result or 0) + item) #== 6
```

Initial value is `0` by default but can be overridden:

```python
collection.reduce(lambda result, item: (result or 0) + item, 4) #== 10
```

### reject

It's the inverse of [filter](/#filter) method. It filters the collection using the given callback. The callback should return `True` for any items to remove from the resulting collection:

```python
collection = Collection([1, 2, 3, 4])
filtered = collection.reject(lambda item: item > 2)
filtered.all() #== [1, 2]
```

Unlike most other collection methods, `reject` does not return a new modified collection; it modifies the collection it is called on.

### reverse

Reverses the order of the items in the collection:

```python
collection = Collection([1, 2, 3])
collection.reverse().all() #== [3, 2, 1]
```

Unlike most other collection methods, `reverse` does not return a new modified collection; it modifies the collection it is called on.

### serialize

Converts the collection into a list. If the collection’s values are [ORM models](/models.md), the models will also be converted to dictionaries:

```python
collection = Collection([1, 2, 3])
collection.serialize() #== [1, 2, 3]

collection = Collection([User.find(1)])
collection.serialize() #== [{'id': 1, 'name': 'John', 'email': 'john.doe@masonite.com'}]
```

Be careful, `serialize` also converts all of its nested objects. If you want to get the underlying items as is, use the [all](/#all) method instead.

### shift

Removes and returns the first item from the collection:

```python
collection = Collection([1, 2, 3, 4, 5])
collection.shift() #== 1
collection.all() #== [2, 3, 4, 5]
```

### sort

Sorts the collection:

```python
collection = Collection([5, 3, 1, 2, 4])
sorted = collection.sort()
sorted.all() #== [1, 2, 3, 4, 5]
```

### sum

Returns the sum of all items in the collection:

```python
Collection([1, 2, 3, 4, 5]).sum() #== 15
```

If the collection contains dictionaries or objects, you must pass a key to use for determining which values to sum:

```python
collection = Collection([
    {'name': 'JavaScript: The Good Parts', 'pages': 176},
    {'name': 'JavaScript: The Defnitive Guide', 'pages': 1096}
])
collection.sum('pages') #== 1272
```

### take

Returns a new collection with the specified number of items:

```python
collection = Collection([0, 1, 2, 3, 4, 5])
chunk = collection.take(3)
chunk.all() #== [0, 1, 2]
```

You can also pass a negative integer to take the specified amount of items from the end of the collection:

```python
chunk = collection.chunk(-2)
chunk.all() #== [4, 5]
```

### to_json

Converts the collection into JSON:

```python
collection = Collection([{'name': 'Desk', 'price': 200}])
collection.to_json() #== '[{"name": "Desk", "price": 200}]'
```

### transform

Iterates over the collection and calls the given callback with each item in the collection. The items in the collection will be replaced by the values returned by the callback:

```python
collection = Collection([1, 2, 3, 4, 5])
collection.transform(lambda item: item * 2)
collection.all() #== [2, 4, 6, 8, 10]
```

If you wish to create a new collection instead, use the [map](/#map) method.

### unique

Returns all of the unique items in the collection:

```python
collection = Collection([1, 1, 2, 2, 3, 4, 2])
unique = collection.unique()
unique.all() #== [1, 2, 3, 4]
```

When dealing with dictionaries or objects, you can specify the key used to determine uniqueness:

```python
collection = Collection([
    {'name': 'Sam', 'role': 'admin'},
    {'name': 'Joe', 'role': 'basic'},
    {'name': 'Joe', 'role': 'admin'},
])
unique = collection.unique('name')
unique.all()
# [
#     {'name': 'Sam', 'role': 'admin'},
#     {'name': 'Joe', 'role': 'basic'}
# ]
```

### where

Filters the collection by a given key / value pair:

```python
collection = Collection([
    {'name': 'Desk', 'price': 200},
    {'name': 'Chair', 'price': 100},
    {'name': 'Bookcase', 'price': 150},
    {'name': 'Door', 'price': 100},
])
filtered = collection.where('price', 100)
filtered.all()
# [
#     {'name': 'Chair', 'price': 100},
#     {'name': 'Door', 'price': 100}
# ]

```

### zip

Merges together the values of the given list with the values of the collection at the corresponding index:

```python
collection = Collection(['Chair', 'Desk'])
zipped = collection.zip([100, 200])
zipped.all() #== [('Chair', 100), ('Desk', 200)]
```
