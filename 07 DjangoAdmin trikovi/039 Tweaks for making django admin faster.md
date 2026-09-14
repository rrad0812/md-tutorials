Tweaks for making django admin
faster
19 January 2012 (updated 04 March 2015) , blog.ionelmc.ro

Here follow a number of tricks I've employed in the past to make django admin

faster.



Editable foreign keys in the changelist

If you have foreign keys in list_editable  django will make 1 database query

for each item in the changelist. Quite a lot for a changelist of 100 items. The trick is

to cache the choices for that formfield:

class MyAdmin(admin.ModelAdmin):
list_editable = 'myfield',
def formfield_for_dbfield(self, db_field, **kwargs):

request = kwargs['request']
formfield = super(MyAdmin, self).formfield_for_dbfield(db_field,

**kwargs)

if db_field.name == 'myfield':
choices = getattr(request, '_myfield_choices_cache', None)
if choices is None:

request._myfield_choices_cache = choices =
list(formfield.choices)

formfield.choices = choices

return formfield



Foreign keys or many to many fields in
admin inlines

If you have fk of m2m fields on InlineModelAdmin for every object in the formset

you'll get a database hit. You can avoid this by having something like:

class MyAdmin(admin.TabularInline):
fields = 'myfield',
def formfield_for_dbfield(self, db_field, **kwargs):

formfield = super(MyAdmin, self).formfield_for_dbfield(db_field,
**kwargs)

if db_field.name == 'myfield':
# dirty trick so queryset is evaluated and cached in .choices
formfield.choices = formfield.choices

return formfield



Enable template caching

It's amazing how easy it is to forget to add this in your settings:

TEMPLATE_LOADERS = (
('django.template.loaders.cached.Loader', (

'django.template.loaders.filesystem.Loader',
'django.template.loaders.app_directories.Loader',

)),
)



Use select_related for the edit forms too

In case you have some readonly fields on the edit form and they need related data

to display list_select_related doesn't help. Eg:

class MyAdmin(admin.ModelAdmin):
readonly_fields = 'myfield',
def queryset(self, request):

return super(MyAdmin, self).queryset(request).select_related('myfield')



Use annotations if possible for function
entries list_display instead of making
additional queries

Check the aaggggrrrrreeeggaatttttiiiiioonn       aappiiiii to see if you can use this. Here's the typical example:

class AuthorAdmin(admin.ModelAdmin):
list_display = 'books_count',

def books_count(self, obj):
return obj.books_count

def queryset(self, request):
return super(AuthorAdmin, self).queryset(

request).annotate(books_count=Count('books'))

The models would look like this:

class Author(models.Model):
name = models.CharField(max_length=100)

class Book(models.Model):
name = models.CharField(max_length=100)
author = models.ForeignKey(Author, related_name="books")



Cache the filters from the admin changelist

This has the obvious tradeoff that you'll have stale data in the list of filter but if

they don't change that often and those distinct queries are killing your database

then this will help a lot. Just add a custom change_list.html template in your

project (eg: templates/<myapp>/change_list.html ):

{% extends "admin/change_list.html" %}
{% load admin_list i18n cache %}

{% block filters %}
{% cache 300 admin_filters request.GET request.path request.user.username

%}
{% if cl.has_filters %}

          <div id="changelist-filter">
            <h2>{% trans 'Filter' %}</h2>

{% for spec in cl.filter_specs %}{% admin_list_filter cl spec %}{%
endfor %}
          </div>

{% endif %}
{% endcache %}

{% endblock %}

Originally the example had request.GET.items  in there, but that is

wrong as in Python 3 that returns a generator. The template tag will use

the string representations of all the arguments so there's no reason to do

anything fancy there.



Bonus trick

frame = sys._getframe(1)
while frame:

if frame.f_code.co_name == 'render_change_form':
if 'request' in frame.f_locals:

request = frame.f_locals['request']
break

frame = frame.f_back
else:

raise RuntimeError("Could not find request object.")

# do stuff with request

This could be used in some specific cases (eg: you need the request in a widget's

render method), as a last resort ofcourse ;)

--

What did you do to make django admin faster?

This entry was tagged as ddjjjjajaaaannggoooooddjjjjajaaaannggooooo     a  aaaaddmmiiiiinnppyyttttthhooooonn

Similar entries: HHoooooww     t  ttttooooo     r  rrrruunn       uuWSSGGIIIII, SSppeeeeeeeeeddiiiiinngg       uupp     D  Djjjjajaaaannggooooo       ppaaaaaggiiiiinnaaaaatttttiiiioioooonn, PPrrrrroooooxxyyiiiiinngg     o  oooobbbbjjjjejeeeeccccctttttsssss     i  iiiinn

PPyyttttthhooooonn.

© Copyright 2025 by IIIIIIIoonneelllll ll    C  Crrrrriiiiisiisssstttttiiiiiiiaann     M  Măărrrrriiiiiiieeșșșșș. Powered by PPeelllllilliiiiiiccccaann. Theme based on flflaassssskkkyyyy.
AArrrrrcccchhiiiiiviivvveeTTaaggsssssRRSSSS you@example.com Subscribe Search