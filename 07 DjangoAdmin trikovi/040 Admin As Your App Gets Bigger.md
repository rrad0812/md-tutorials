Things You Must Know About
Django Admin As Your App Gets
Bigger
From Zero to Hero With Django Admin

. . .

The Django admin is a very powerful tool. We use it for day to day operations,browsing
data and support. As we grew some of our projects from zero to 100K+users we started
experiencing some of Django's admin pain points - long response times and heavy load
on the database.

In this short article I am going to share some simple techniques we use in our projects
to make the Django admin behave as apps grow in size and complexity.

We use Django 1.8, Python 3.4 and PostgreSQL 9.4. The code samples are for Python
3.4 but they can be easily modified to work on 2.7 and other Django versions.

. . .

Before We Start
These are the main components in a Django Admin list view:



Example admin list view including some of the components discussed in the article

Logging
Most of Django's work is performing SQL queries so our main focus will be on
minimizing the amount of queries. To keep track of query execution you can use one of
the following:

• django-debug-toolbar - Very nice utility that adds a little panel on the side of the
screen with a list of SQL queries executed and other useful metrics.

• If you don't like dependencies (like us) you can log SQL queries to the console by
adding the following logger in settings.py:

LOGGING = {
# ...
'loggers': {

'django.db.backends': {
'level': 'DEBUG',

},
},
# ...

}

. . .

The N1 Problem
The N1 problem is a well known problem in ORMs. To illustrate the problem let's say
we have this schema:

class Category(models.Model):
name = models.CharField(max_length=50)

def__str__(self):
return self.name

class Product(models.Model):
name = models.CharField(max_length=50)
category models ForeignKey(Category)



category = models.ForeignKey(Category)

By implementing __str__ we tell Django that we want the name of the category to be
used as the default description of the object. Whenever we print a category object,
Django will fetch the name of the category.

A simple admin page for our Product model might look like this:

@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):

list_display = (
'id',
'name',
'category',

)

This seems innocent enough but the SQL log reveals the horror:

(0.000) SELECT COUNT(*) AS "__count" FROM "app_product"; args=()
(0.002) SELECT "app_product"."id", "app_product"."name", "app_product"."category_id"

FROM "app_product" ORDER BY "app_product"."id" DESC LIMIT 100; args=()
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 1; args=(1)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 2; args=(2)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 1; args=(1)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 4; args=(4)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 3; args=(3)
...
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 2; args=(2)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 99; args=(99)
(0.000) SELECT ... FROM "app_category" where "app_category"."id" = 104; args=(104)

Django first counts the objects (more on that later), then fetches the actual objects
(limiting to the default page size of 100 and then passes the data on to the template for
rendering. We used the category name as the description of the Category object, so for
each product Django has to fetch the category name. This results in 100 additional
queries.

To tell Django we want to perform a join instead of fetching the names of the categories
one by one, we can use list_select_related:

@admin.register(models.Product)



@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):

list_display = (
'id',
'name',
'category',

)
list_select_related = (

'category',
)

Now, the SQL log looks much nicer. Instead of 101 queries we have only 1

(0.004) SELECT "app_product"."id", "app_product"."name",
"app_product"."category_id", "app_category"."id", "app_category"."name"
FROM "app_product"
INNER JOIN "app_category" on ("app_product"."category_id" = "app_category"."id")
ORDER BY "app_product"."id" DESC LIMIT 100; args=()

To understand the real impact of this setting consider the following. Django default
page size is 100 objects. If you have one related fields you have ~101 queries. If you
have two related objects displayed in the list view, you have ~201 queries and so on.

Fetching related fields in a join can only work for ForeignKey relations. If you wish to
display ManyToMany relations it's a bit more complicated (and most of the time wrong, but
keep reading).

. . .

Related Fields
Sometimes it can be useful to quickly navigate between objects. After trying for a while
to teach support personnel to filter using URL parameters, we finally gave up and
created two simple decorators.

admin_link

Create a link to a detail page of a related model:

def admin_change_url(obj):
app_label = obj._meta.app_label
model_name = obj._meta.model.__name__.lower()
return reverse('admin:{}_{}_change'.format(

app_label, model_name



app_label, model_name
), args=(obj.pk,))

def admin_link(attr, short_description, empty_description="-"):
"""Decorator used for rendering a link to a related model in

    the admin detail page.

    attr (str):
        Name of the related field.
    short_description (str):
        Name if the field.
    empty_description (str):
        Value to display if the related field is None.

    The wrapped method receives the related object and should
    return the link text.

    Usage:
        @admin_link('credit_card', _('Credit Card'))
        def credit_card_link(self, credit_card):
            return credit_card.name
    """

def wrap(func):
def field_func(self, obj):

related_obj = getattr(obj, attr)
if related_obj is None:

return empty_description
url = admin_change_url(related_obj)
return format_html('<a href="{}">{}</a>', url, func(self, related_obj))

field_func.short_description = short_description
field_func.allow_tags = True

return field_func
return wrap

The decorator will render a link (<a href="...">...</a>) to the related model in both the
list view and the detail view. If for example, we want to add a link from each product to
its category detail page, we use the decorator like this:

@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):

list_display = (
'id',
'name',
'category_link',

)
admin_select_related = (

'category',
)



@admin_link('category', _('Category'))
def category_link(self, category):

return category

admin_changelist_link

More complicated links such as "all the products of a category" require a different
implementation. We created a decorator that accepts a query string, and link to the list
view of a related model:

def admin_changelist_url(model):
app_label = model._meta.app_label
model_name = model.__name__.lower()
return reverse('admin:{}_{}_changelist'.format(app_label, model_name))

def admin_changelist_link(
attr,
short_description,
empty_description='-',
query_string=None

):
"""Decorator used for rendering a link to the list display of

    a related model in the admin detail page.

    attr (str):
        Name of the related field.
    short_description (str):
        Field display name.
    empty_description (str):
        Value to display if the related field is None.
    query_string (function):
        Optional callback for adding a query string to the link.
        Receives the object and should return a query string.

    The wrapped method receives the related object and
    should return the link text.

    Usage:

        @admin_changelist_link('credit_card', _('Credit Card'))
        def credit_card_link(self, credit_card):
            return credit_card.name
    """

def wrap(func):
def field_func(self, obj):

related_obj = getattr(obj, attr)
if related_obj is None:

return empty_description
url = admin_changelist_url(related_obj.model)
if query_string



if query_string:
url += '?' + query_string(obj)

return format_html('<a href="{}">{}</a>', url, func(self, related_obj))
field_func.short_description = short_description
field_func.allow_tags = True

return field_func
return wrap

To add a link from a category to of its products, we do the following in CategoryAdmin:

@admin.register(models.Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = (
        'id',
        'name',
        'products_link',
    )

    @admin_changelist_link('products', _('Products'),
                            query_string=lambda c: 'category_id={}'.format(c.pk))
    def products_link(self, products):
        return _('Products')

Be careful with the products argument. It is very tempting to do something like this:

# Bad example

@admin_changelist_link('products', _('Products'),
query_string=lambda c: 'category_id={}'.format(c.pk))

def products_link(self, products):
# Dont do that!
return 'see {} products'.format(products.count())

The example above will result in additional queries.

readonly_fields

In the detail page, Django creates an editable element for each field. Text and numeric
fields will be rendered as regular input field. Choice fields and foreign key fields will be
rendered as a <select> element. To render a select box Django has to do the following:

 Fetch the options - the entire related model and their descriptions (remember the N1
problem?).

 Render the option list - one option for each related model instance.



 Render the option list - one option for each related model instance.

A common scenario that is often overlooked, is foreign key to the User model. When you
have 100 users you might not notice the load, but what happens when you suddenly
have 100K users? The detail page will fetch the entire users table, and the option list
will make the resulting HTML huge. We pay twice, first for the full table scan, and then
for downloading the html file. Not to mention the memory required to generate the html
file in the first place.

Having a select element with 100K options is not really usable. The easiest way to
prevent Django from rendering a field as a <select> element is to mark it as
readonly_fields:

@admin.register(SomeModel)
def SomeModelAdmin(admin.ModelAdmin):

readonly_fields = (
'user',

)

This will render the description of the related model, without being able to change it in
the admin.

Another option to prevent Django from rendering a select box, is to mark the field as
raw_id fields.

@admin.register(SomeModel)
def SomeModelAdmin(admin.ModelAdmin):

raw_id_fields = (
'user',

)

Using raw_id_fields, Django will render a special widget that shows the id of the value,
and an option to open a list of all values in a popup window. This option is very useful
when you want to edit a foreign key value.

. . .

Filters
We often use the admin interface as a day to day tool for general support. We found
that most of the times we use the same filters: only active users, users registered in the



last month, successful transactions ans so on. Once we realized that, we asked
ourselves, why fetch the entire dataset if we are most likely to immediately apply a filter
to it?. We started to look for a way to apply a default filter when entering the model list
view.

DefaultFilterMixin

There are many approaches to apply default filters. Some approaches involve custom
filters or injecting special query parameters to the request. We wanted to avoid those.

We found that the following approach to be simple and straightforward:

from urllib.parse import urlencode
from django.shortcuts import redirect

class DefaultFilterMixin:
def get_default_filters(self, request):

"""Set default filters to the page.

        request (Request)

        Returns (dict):
            Default filter to encode.
        """

raise NotImplementedError()

def changelist_view(self, request, extra_context=None):
ref = request.META.get('HTTP_REFERER', '')
path = request.META.get('PATH_INFO', '')

# If already have query parameters or if the page
# was referred from it self (by drilldown or redirect)
# don't apply default filter.
if request.GET or ref.endswith(path):

return super().changelist_view(request, extra_context=extra_context)

query = urlencode(self.get_default_filters(request))
return redirect('{}?{}'.format(path, query))

If the list view was accessed from a different view, and no query params were specified,
we generate a default query and redirect.

Let's apply a default filter to our product page to show only products created in the last
month:



from django.utils import timezone

@admin.register(models.Product)
class ProductAdmin(DefaultFilterMixin, admin.ModelAdmin):

date_hierarchy = 'created'

def get_default_filters(self, request):
now = timezone.now()
return {

'created__year': now.year,
'created__month': now.month,

}

If we drill down from within the page, or if we get to the page with query parameters,
the default filter will not be applied.

. . .

Quick Bits
Some neat tricks we gathered over time.

show_full_result_count

Prevent Django from showing the total number of rows in the list view. Setting
show_full_result_count=False saves a count(*) query on the queryset on every page load.

defer

When performing a query the entire resultset is put into memory for processing. If you
have large columns in your model such as JSON or Text fields, it might be a good idea
to defer them until you really need to use them. To defer fields override get_queryset.

Change the Admin Default URL Route
This is definitely not the only precaution you should take to protect your admin page,
but it can make it harder for "curious" users to reach the login page.

In your main urls.py override the default admin route:

# urls.py

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
url(r'^foo/' include(admin site urls)),



url(r'^foo/', include(admin.site.urls)),
]

SEE ALSO

I wrote a bunch of tips on how to make Django admin safer.

date_hierarchy

We found that this index can be used to improve queries generate with date hierarchy
predicate in PostgresSQL 9.4

CREATE INDEX yourmodel_date_hierarchy_ix ON yourmodel_table (
extract('day' from created at time zone 'America/New_York'),
extract('month' from created at time zone 'America/New_York'),
extract('year' from created at time zone 'America/New_York')

);

Make sure to change table name, index name, the date hierarchy column and the time
zone.