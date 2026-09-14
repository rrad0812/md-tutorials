…Django Performance Optimisations
2024-01-24 |  10 minute read |  #software , #development , #django ,

#python

In the past 4 weeks, I have been iterating on optimising a Django app, in

particular the Django admin, which is heavily by one of the apps I help

maintain at work.

When I initially started investigating the topic, many places pointed out

to caching. Caching is a great option indeed, but it can lead to a lot of
problems. However, for the Django Admin view, caching is even more compli-

cated because it is not about caching the views and URLs - instead it would

require custom caching rules and logic, which seems an unnecessary layer of

complexity. Furthermore, most of the bottlenecks we experience are related

to database queries and the load there, not issues generating the views.

Moreover, these database inefficiencies would not be solved with caching -

in fact, they would be perpetuated and just masked behind a cache. Hence,

view caching is not covered in this post - but query caching will somewhat

be.

This post details the main tips and takeaways of my experience in this ad-

venture, summarising my learnings while working on this.

# Tip 1 - django-debug-toolbar is a great companion

Whether you have access to other forms of application performance monitor

(APM) or not, this tool is a lifesaver on many ends. It will help you ana-

lyse SQL queries, understand where load times are getting stuck and much

more.

The django-debug-toolbar  provides sufficient information to get started

optimising things without many hiccups and it’s really straightforward to

setup and use. It goes without saying but, please ensure you are not pub-

lishing the configuration that enables the toolbar in production - that

could lead to a massive security incident if your app contains sensitive

data and/or is exposed on a public service…



In my endeavour to optimise some views of our django-admin , what I used

mostly was the SQL tab. However, occasionally I ticked the other ones too,

and the amount of information this tool provides is just great.

# Tip 2 - select_related for all Foreign Keys

A common thing in your models is for them to have foreign keys into other

models. This is a basic primitive to translate your data relationships be-

tween models.

class Book(models.Model):
    name = models.CharField(���)
    author = models.ForeignKey("app.Author", on_delete=models.CASCADE)

class Author(models.Model):
    name = models.TextField(���) # let's assume that for multi-author book we can add all nam

# In your admin.py
@admin.register(models.Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ("name", "author��name")
    readonly_fields = ("name", "author��name")
    search_fields = ("name", "author��name")

In that very simple, contrived example what will happen is that for each

Book, Django will write another query to fetch the author of that book. As

things scale and you start having a few hundred thousand books, loading

this page will take 2, 3 or even more seconds because it will perform

O(n^2)  queries. Not only will the platform become slower, but it will also

drain your application.

To solve this, there are two very quick ways really. One is just hackier/

simpler, the other one is way more refined/controlled:

1. Specify list_select_related=True  in the Admin  model definition, and

Django will automatically perform this for all foreign keys the model

contains.

2. Specify which foreign keys you want to be preloaded with

list_select_related=("author") .



In both cases, Django will essentially perform an inner join query in both

tables, thus no longer performing O(n^2)  operations but O(n) . In some

cases, this has been a golden hack really!

Relevant references:

• https://docs.djangoproject.com/en/5.0/ref/contrib/admin/

#django.contrib.admin.ModelAdmin.list_select_related

• https://docs.djangoproject.com/en/5.0/ref/models/querysets/#select-

related

# Tip 3 - prefetch_related all those ManyToMany

Similarly, models regularly use many-to-many relationships to better spec-

ify and explain the relationships between our data models in our programs.

Let’s review the example from the previous section and make it even more

accurate:

class Book(models.Model):
    name = models.CharField(���)
    author = models.ManyToMany("app.Author")

class Author(models.Model):
    name = models.TextField(���)

We’re now ensuring books can have 1 or more authors, properly represented

by the ManyToMany  relationship and this is particularly relevant/visibile

for multi-author books; Now, instead of authors which would have the many

authors in the name  field, we can create dedicated entries for each author

and get them together through this relationship instead! This also facili-

tates other types of visualisations you would like to do, namely how many

books an Author has, regardless of publishing alone or not.

The admin view would look very similar to before, however:

# In your admin.py
@admin.register(models.Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ("name", "author��name")



    readonly_fields = ("name", "author��name")
    search_fields = ("name", "author��name")

In case of many-to-many relationships, if we do not optimise we will end up

performing also n^3  queries for each book - one for the intermediate table

that contains the ids for the many-to-many relatiionships, and another for

the destination table (author, in this case), to fetch the needed data.

However, list_select_related  will not work this time. For these types of

relationships, we should instead use prefetch_related  to prefetch the data

of the predefined models and do the joining in Python instead and there is

no ModelAdmin  attribute we can leverage this time. Instead, we have to get

hacky and override the get_queryset  method for the object instance and

since this returns a queryset, we can add the prefetch_related  we need:

# In your admin.py
@admin.register(models.Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ("name", "author��name")
    readonly_fields = ("name", "author��name")
    search_fields = ("name", "author��name")

def get_queryset(self, request):
return super().get_queryset(request).prefetch_related("author")

Basically, we are tweaking the queryset the ModelAdmin prepares when this

view is loadding and ensuring the author’s table is prefetched. Overriding

this method to tweak the queryset will come back again, so if you’re strug-

gling to understand how this works, I’d advise to read more about object-

oriented programming and class inheritance in Python.

With this minimal change, our Django Admin becomes n  again as it performs

one additional query only, to cache the authors table then performing some

arrangements in Python.

Relevant references:

• https://docs.djangoproject.com/en/5.0/ref/models/querysets/#prefetch-

related

• https://docs.djangoproject.com/en/4.2/ref/contrib/admin/

#django.contrib.admin.ModelAdmin.get_queryset



# Tip 4 - Use count annotations instead of model attributes

Often times, we would like our models to have counts in them, such that we

can know, for example, how many books an Author has, right? What could be

some of the approaches to this problem?

1. Create a field in the model that is updated when the its save  method

is called to increase this counter

2. Create a method to the model that will get us the count when called

and call this method on the admin view

3. User annotates and counts, making the database work for us and perform

these counts we need for ourselves

class Author(models.Model):
    name = models.TextField(���)
    book_count = models.IntegerField()

def save(self):
# Pretty tricky logic goes here to increment book_count, ensuring no duplicate books, 
return super().save(���)

Option 1 is insanely complicated and somewhat impossible to make it right,

hence I’m stopping evolving this hypothesis because of that. Let’s move on

to option 2:

@admin.register(models.Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ("name", "book_count")
    readonly_fields = ("name", "book_count")
    search_fields = ("name")

def book_count(self, obj):
return models.Author.objects.filter(author=obj).count()

    book_count.short_description = "Published of books"

I saw many cases where something like this was implemented. And although at

first sight it seems a good idea, we have to account for the fact this

means, for each entry in the Author’s table, this query will be performed.



This can put your database under tremendous load, and there is a much eas-

ier way to achieve this - option 3:

@admin.register(models.Author)
class AuthorAdmin(admin.ModelAdmin):
    list_display = ("name", "book_count")
    readonly_fields = ("name", "book_count")
    search_fields = ("name")

def get_queryset(self, request):
return super().get_queryset(request).annotate(num_books=Count("books"

def book_count(self, obj):
return obj.num_books

    book_count.short_description = "Published of books"

A few things are happening there. If you were not aware, the fields you

specify in list_display  can be the field’s names or a callable. This means

we can define functions to do custom things with the object data returned

by the queryset. Typically, this is used for rearranging data, like speci-

fying a function that merges first and last names.

When we override the get_queryset  method (I said earlier it was going to

show up again), we can use a nice feature from the Django ORM to annotate

the response to add new columns to the response - in this case, to have the

database do the counts for us in a single query, instead of finding hacky

ways to do them, reducing processing impact or database load for a whole

lot.

A nice thing I learnt about this was Django is smart enough to do the ID

checks in other models. This means that even though you are querying an-

other table, it will do the proper ID filtering for you without you having

to worry about that. In the end, I feel this approach even makes the code

more readable and maintainable, not just more performant.

Again, this improves Django Admin performance from O(n^2)  to O(n) .

Relevant references:

• https://docs.djangoproject.com/en/5.0/ref/contrib/admin/



#django.contrib.admin.ModelAdmin.list_display

• https://docs.djangoproject.com/en/5.0/ref/models/querysets/#count

# Tip 5 - Proper use of indexes field attribute

These tips are not in order, for sure, as this might be a great starting

point if you are aware and comfortable about your models’ relationships and

querying needs.

The  last  tip  I  have  for  this  post,  is  about  using  indexes  correctly.

Previously this was known as adding db_index=True  to your field defini-

tion, but Django now is migrating to a more verbose/explicit index declara-

tion with Meta  options.

Using db_index  or indexes  meta property essentially creates an index for

the specified fields, making queries on them significantly faster and more

performant.  This  is  recommended  to  use  on  fields  that  are  frequently

queried.  However,  ForeignKey ,  ManyToMany  and  fields  defined  in

UniqueConstraint s do not require this definition as it will be taken care

of automatically.

This change will not imply improving the algorithm’s performance in terms

of cycles but by being faster on the actual lookups.

Relevant references:

• https://stackoverflow.com/questions/59596176/when-we-should-use-db-

index-true-in-django

• https://docs.djangoproject.com/en/5.0/ref/models/options/

#django.db.models.Options.indexes

• https://docs.djangoproject.com/en/5.0/ref/models/fields/#db-index

# Tip 6 - Query caching

Although I did mention caching was not covered, I was referring to view

caching. One type of cache that can be worked on are query caches, which

for functions/calls that perform the same query over and over with some-

thing variable does wonders.



This tip is perhaps for a niche of Django applications that have custom

commands. Regardless, seems worth to mention this trick.

Imagine you implemented a custom command for housekeeping needs and that

somewhere in the code you are running the same query over and over:

def handle(self):
for entry in MyModel.objects.all():

        another_entry = AnotherModel.objects.filter(entry=entry).first()
pass

The above code does nothing but the point I want to highlight is that we

can  cache  AnotherModel.objects.filter(entry=entry).first()  query,  using

functools ’ cache  function - Python built-ins, no extra package, no extra

setup. How can we do that?

@functools.cache
def getAnotherModel(entry):

return AnotherModel.objects.filter(entry=entry).first()

def handle(self):
for entry in MyModel.objects.all():

        another_entry = getAnotherModel(entry)
pass

For the duration of your custom command, the queries’ results will be

cached in-memory and each time you repeat the function call, you will get

the cached result instead of performing another query.

The contrived example might be the most perfect one to demonstrate the dif-

ference, but often times our housekeeping commands do repeat the queries a

lot. This means the filter parameters would be the same somehow, and this

increases the load the on the database servers again by performing the same

query over and over again.

This idea can however be widespread in other use cases anywhere that per-

forms the same type of queries with variable arguments, like view’s func-

tions for instance.



Relevant references:

• https://docs.djangoproject.com/en/4.2/ref/django-admin/#running-

management-commands-from-your-code

• https://docs.python.org/3/library/functools.html#functools.cache

# Rant - Inlines are painful

Inlines have been a problem for me and I haven’t quite well figure out the

next steps for them. If you search for this topic, you will find a lot of

StackOverflow discussions on this, most with a bunch of hacky solutions to

modify the querysets and such. In my particular case, the added complexity

never justified the performance gains we got, but it still might be life-

saver in your case.

But this leads to the point on this section: Custom SQL and queries.

When neither of the above is sufficient, you might need custom SQL queries

somewhere or revise your own model definition and setup entirely as some-

thing is likely wrong - I’m working on a gigantic database here and these

have done wonders. We still need to do some optimisations at templating

level and the only tables of our admin views that still take some time to

load are usually the tables with more than 10GBs of data on them.

Custom SQLs is a whole different beast to discuss, hence I will leave it

out of this post. Regardless, wanted to highlight inlines can be a perfor-

mance bottleneck, specially if you use them often.

Relevant References:

• https://stackoverflow.com/questions/29647418/django-admin-inline-

select-related

• https://stackoverflow.com/questions/49411204/django-excessive-queries-

with-inlines

• https://stackoverflow.com/questions/16305908/slow-performance-for-

django-admin-inline

• https://stackoverflow.com/questions/559701/django-queries-made-repeat-

inefficient?rq=4