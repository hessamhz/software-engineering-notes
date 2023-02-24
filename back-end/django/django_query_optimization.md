# Table of Contents
1. [Database Design](#dbdesign)
    1. [Indexing](#indexing)
2. [Queries](#queries)
    1. [ORM is Lazy](#orm)
    2. [How Django connects to the database](#connection)
    3. [Python or SQL?](#psql)
    4. [Bulk Insertion](#bulk)
    5. [Application of select_related() and prefetch_related()](#select)
    6. [Defer and Only](#defer)
    7. [Application of values() and value_list()](#values)
    8. [F Expression](#fexp)
    9. [Aggregations](#aggr)
    10. [FKs optimization](#fk)
    10. [Application of count() and exits()](#count)
3. [Final Note](#final)

## Datebase Design <a id="dbdesign"></a>
### Indexing <a id="indexing"></a>
Use Meta.indexes or Field.db_index to fields that you frequently query using filter(), exclude(), order_by(), etc. as indexes may help to speed up lookups. Note that determining the best indexes is a complex database-dependent topic that will depend on your particular application. The overhead of maintaining an index may outweigh any gains in query speed.

When specifying db_index=True on your model fields, Django typically outputs a single CREATE INDEX statement. However, if the database type for the field is either varchar or text (e.g., used by CharField, FileField, and TextField), then Django will create an additional index that uses an appropriate PostgreSQL operator class for the column. The extra index is necessary to correctly perform lookups that use the LIKE operator in their SQL, as is done with the contains and startswith lookup types.
## Queries <a id="queries"></a>
### ORM is Lazy <a id="orm"></a>
Before really jumping into the whole doc, you should know that django querysets are lazy loaded and cached. Lazy loading means that until you perform certain actions on the queryset, such as iterating over it, the corresponding DB query won't be made. Caching means that if you re-use the same queryset, multiple DB queries won't be made. All of .all(), .filter() and .exclude() are lazy loaded. Take note that FKs and reverse relationships and M2M fields are NOT cached.
So when actually querysets are evaluated?
``` python
# Iteration
for person in Person.objects.all():
    # Some logic

# Slicing/Indexing
Person.objects.all()[0]

# Pickling (i.e. serialization)
pickle.dumps(Person.objects.all())

# Evaluation functions
repr(Person.objects.all())
len(Person.objects.all())
list(Person.objects.all())
bool(Person.objects.all())

# Other
[person for person in Person.objects.all()]  # List comprehensions
person in Person.objects.all()  # `in` checks
```
When queries Cached?
``` python
### Not Cached

# Not reusing evaluated QuerySets
print([p.name for p in Person.objects.all()])  # QuerySet evaluated and cached
print([p.name for p in Person.objects.all()])  # New QuerySet is evaluated and cached

# Slicing/indexing unevaluated QuerySets
queryset = Person.objects.all()
print(queryset[0])  # Queries the database
print(queryset[0])  # Queries the database again

# Printing
print(Person.objects.all())

### Cached

# Reusing an evaluated QuerySet
queryset = Person.objects.all()
print([p.name for p in queryset])  # QuerySet evaluated and cached
print([p.name for p in queryset])  # Cached results are used

# Slicing/indexing evaluated QuerySets
queryset = Person.objects.all()
list(queryset)  # Queryset evaluated and cached
print(queryset[0])  # Cache used
print(queryset[0])  # Cache used


## Not initially retrieved/cached

# Foreign-key related objects
person = Person.objects.get(id=1)
person.father  # foreign object is retrieved and cached
person.father  # cached version is used

## Never cached

# Callable attributes
person = Person.objects.get(id=1)
person.children.all()  # Database hit
person.children.all()  # Another database hit
```
Here is another sequential example.
``` python
qs = Person.objects.filter(id=3)
# The DB query has not been executed at this point
x = qs
# Just assigning variables doesn't do anything
for x in qs:
    print(x) 
# The query is executed at this point, on iteration
for x in qs:
    print(x.id)
# The query is not executed this time, due to caching
```
<b> Important Note </b>
As you might think this is a double-edged sword. Since we are catching if we use a batch of the queryset the garbage collecter doesn't work!. Django solution for this is using the .iterator() but it also has some flaws. I recommend reading <a href="https://www.codementor.io/@pritishc/django-optimization-or-how-we-avoided-memory-mishaps-yh6fwfxet"> this </a> article which handles the problem.
``` python
for post in Blog.objects.iterator():
    # Do sth
```
### How Django connects to the DB? <a id="connection"></a>
Django opens a connection to the database when it first makes a database query. It keeps this connection open and reuses it in subsequent requests. Django closes the connection once it exceeds the maximum age defined by CONN_MAX_AGE or when it isn’t usable any longer.

In detail, Django automatically opens a connection to the database whenever it needs one and doesn’t have one already — either because this is the first connection, or because the previous connection was closed.
At the beginning of each request, Django closes the connection if it has reached its maximum age. If your database terminates idle connections after some time, you should set CONN_MAX_AGE to a lower value, so that Django doesn’t attempt to use a connection that has been terminated by the database server. (This problem may only affect very low traffic sites.)

At the end of each request, Django closes the connection if it has reached its maximum age or if it is in an unrecoverable error state. If any database errors have occurred while processing the requests, Django checks whether the connection still works, and closes it if it doesn’t. Thus, database errors affect at most one request; if the connection becomes unusable, the next request gets a fresh connection.
### Python or SQL? <a id="psql"></a>
Almost everything will be faster with the SQL query so be kind and use it. For example most of the loops on data in Python can be easily done with query. look at the aggeration example. Another example is creating, updating or deleting.
``` python
# DON'T
for person in Person.objects.all():
    person.age = 0
    person.save()

# DO
Person.objects.update(age=0)

# DON'T
for person in Person.objects.all():
    person.delete()

# DO
Person.objects.all().delete()
```
### Bulk Insertion (Create and Update) <a id="bulk"></a>
Always use bulk queries in order to create or update a bulk number of objects.
``` python
# Foo is a django model

for i in range(100):
    Foo(bar="sth").save()
```
The above query runs 100 inserts. instead we should use bulk_create

``` python
foos = []
batch_size = 100
for i in range(100):
    foos.append(Foo(bar="sth))
Foo.objects.bulk_create(foos, 100)

# Foo.objects.bulk_update(foos, ['bar'], batch_size)
```
create and update doesn't affect signal.

Also take note that you can bulk insert into ManyToMany fields as well and you should do it!
``` python
author1 = Author(name='author1')
author2 = Author(name='author2')
author3 = Author(name='author3')
entry = Entry.objects.get(id=1)

# Don't
entry.authors.add(author1)
entry.authors.add(author2)
entry.authors.add(author3)

# Do
entry.authors.add(author1, author2, author3)
```

### select_related and prefetch_related <a id="select"></a>
In order to not use extra queries we can use select_related() this function will do a SQL Inner join on two tables and append them together so we do not do an extra query on the joint table for example:
``` python
class Media(models.Model):
    image = models.FileField()

class User(models.Model):
    name = models.CharField(max_length=200)
    avatar = models.ForeignKey(
        Media,
        on_delete=models.SET_NULL,
        related_id="user_avatar",
    )
``` 
In this case if want to get the Image we have to do an extra query:
``` python
user = User.objects.get(id=1)
# This runs a query which gets the media model
image = user.avatar.image
```
so instead we will use select_related, this will run a SQL query and joins the two tables. Just be careful that select_related <i> only </i> works on <b> OneToOne </b> and <b> ForeignKey </b> for ManytoMany and ReverseForeignKey you have to use prefetch_related this will instead run multiple SQL queries and does the join on Python Level.
 
I Recommend reading <a href="https://betterprogramming.pub/django-select-related-and-prefetch-related-f23043fd635d"> this </a> article as well, which analyzes the query perfomance. 

### Defer and Only <a id="defer"></a>
We can use defer to avoid fetching fields from DB, Imagine a queryset that doesn't need a big or hard to convert object like HTML Field. u can use defer('field') to avoid fetching it from db. 
``` python
Post.objects.defer("content").filter(author="mammad")
```
You can also clear the set of deffered fields by passing None:
``` python
queryset.defer(None)
```
#### <b>Important Note</b>
The defer() and only() are only for advanced use-cases. They provide an optimization for when <b> you have analyzed your queries closely </b> and understand exactly what information you need and have measured that the difference between returning the fields you need and the full set of fields for the model will be significant. <br/>
If you are frequently loading and using a particular subset of your data, the best choice you can make is to normalize your models and put the non-loaded data into a separate model.
``` python
class CommonlyUsedModel(models.Model):
    f1 = models.CharField(max_length=10)

    class Meta:
        managed = False
        db_table = 'app_largetable'

class ManagedModel(models.Model):
    f1 = models.CharField(max_length=10)
    f2 = models.CharField(max_length=10)

    class Meta:
        db_table = 'app_largetable'

# Two equivalent QuerySets:
CommonlyUsedModel.objects.all()
ManagedModel.objects.all().defer('f2')
``` 
On the other hand only() method is exactly opposite of defer() for example for the above problem it will be:
``` python
ManagedModel.objects.all().only('f1')
```
### values() and value_list() <a id="values"></a>
Similar to only() and defer() you can make lists, dicts or tuples with the fields you want. 
``` python
# DON'T
age_lookup = {
    person.name: person.age
    for person in Person.objects.all()
}
# DO
age_lookup = {
    person['name']: person['age']
    for person in Person.objects.values('name', 'age')
}

# DON'T
person_ids = [person.id for person in Person.objects.all()]
# DO
person_ids = Person.objects.values_list('id', flat=True)
```
### F Expression <a id="fexp"></a>
We can use F expression for updating likes, balance and etc.
``` python
for post in Post.objects.all():
    entry.likes += 1
    entry.save()
    
Post.objects.update(likes=F('likes') + 1)
```
### Aggregations  <a id="aggr"></a>
Aggregations can easily convert a python code to a SQL query which runs faster.
``` python
# DON'T
max_age = 0
for person in Person.objects.all():
    if person.age > max_age:
        max_age = person.age
# DO
max_age = Person.objects.all().aggregate(Max('age'))['age__max']
```
### FK Value - Very Important <a id="fk"></a>
ORM actually retrieves and caches the FKs, don't use the .id which hits the db for no specific reason.
``` python
# Don't. Needs database hit
blog_id = Entry.objects.get(id=200).blog.id

# Do. The foreign key is already cached, so no database hit
blog_id = Entry.objects.get(id=200).blog_id

# Do. No database hit
blog_id = Entry.objects.select_related('blog').get(id=200).blog.id
```
### count() and exists() <a id="count"></a>
If you don't need the the content just use this. I have seen it many times that people use len(queryset) to determine the count but it's not as efficient as using .count() which runs differently and uses the SQL.
``` python
# Don't
count = len(Entry.objects.all()) 

# Do
count = Entry.objects.count()  

# Don't
qs = Entry.objects.all()
if qs:
   pass
  
# Do
qs = Entry.objects.exists()
if qs:
   pass
```
## Final Note <a id="final"></a>

This doc deffinitely can be extended. Feel free to do a PR on it.
I also found <a href="https://gist.github.com/levidyrek/6db1cf88b953f3f006bf678a0f09da8e"> this cheat sheet </a> which is absolutely brilliant, I used some of it's example make sure to star it. 

Another mention is <a href="https://github.com/jazzband/django-debug-toolbar"> django-debug-toolbar </a> which really helps analyzing the queryset. queryset.explain() is also a built-in feature to analyze your query.
