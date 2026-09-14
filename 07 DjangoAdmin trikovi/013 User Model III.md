
# Migrating to a Custom User Model Mid-project in Django

In this course, you'll learn how to set up a Custom User Model development environment with Docker in order to build and deploy a RESTful API.

Django's default user model comes with a relatively small number of fields. powered by Python, Django, and Django REST Framework.

Because these fields are not sufficient for all use cases, a lot of Django projects switch to custom user models.

Switching to a custom user model is easy before you migrate your database, but gets significantly more difficult after that since it affects foreign keys, many-to-many View Course relationships, and migrations, to name a few.

To avoid going through this cumbersome migration process, Django's official documentation highly recommends you set up a custom user model at the start of the project even if the default one is sufficient.

Up to this day, there's still no official way of migrating to a custom user model mid-project. The Django community is still discussing what the best way to migrate is in api architecture aws devops the following ticket.

In this article, we'll look at a relatively easy approach to migrating to a custom user heroku kubernetes model mid-project. The migration process we're going to use isn't as destructive as machine learning python react some of the other ones found on the internet and won't require any raw SQL task queue testing vue executions or modifying migrations by hand. web scraping For more on creating a custom user model at the start of a project, check out the "Creating a Custom User Model" in Django article.

## Custom User Model

### Migration Process

Migrating to a custom user model mid-project is a potentially destructive action. Add New Fields Because of that, I've prepared a dummy project you can use to test the migration Django Admin process before moving on to your actual codebase.

### Rename User Table/Model

The dummy project we're going to be working with is called django-custom-user. It's a simple todo app that leverages the user model.

Clone it down:

```sh
git clone --single-branch --branch base git@github.com:duplxey/django-custom-user.git
cd django-custom-user
```

Create a new virtual environment and activate it:

```sh
python3 -m venv venv && source venv/bin/activate
```

Install the requirements:

```sh
(venv)$ pip install -r requirements.txt
```

Spin up a Postgres Docker container:

```sh
$ docker run --name django-todo-postgres -p 5432:5432 \
    -e POSTGRES_USER=django-todo -e POSTGRES_PASSWORD=complexpassword123 \
    -e POSTGRES_DB=django-todo -d postgres
```

Alternatively, you can install and run Postgres outside of Docker if that's your preference. Just make sure to go to core/settings.py and change the
DATABASES  credentials accordingly.

Migrate the database:

```sh
(venv)$ python manage.py migrate
```

Load the fixtures:

```sh
(venv)$ python manage.py loaddata fixtures/auth.json --app auth
(venv)$ python manage.py loaddata fixtures/todo.json --app todo
```

These two fixtures added a few users, groups, and tasks to the database and created a superuser with the following credentials:

```sh
username: admin
password: password
```

Next, run the server:

```sh
(venv)$ python manage.py runserver
```

Lastly, navigate to the admin panel at <http://localhost:8000/admin>, log in as the superuser, and make sure that the data has been loaded successfully.

### Migration Process start

The migration process we're going to use assumes that:

1. Your project doesn't have a custom user model yet.
2. You've already created your database and migrated it.
3. There are no pending migrations and all the existing migrations have been applied.
4. You don't want to lose any data.

If you're still in the development phase and the data in your database isn't important, you don't have to follow these steps. To migrate to a custom user model, you can simply wipe the database, delete all the migration files, and then follow the steps here.

Before following along, please fully back up your database (and codebase). You should also try the steps on a staging branch/environment before moving to production.

Migration Steps

1. Point AUTH_USER_MODEL  to the default Django user in settings.py.
2. Replace all User  references with AUTH_USER_MODEL  or get_user_model() accordingly.
3. Start a new Django app and register it in settings.py.
4. Create an empty migration within the newly created app.
5. Migrate the database, so the empty migration gets applied.
6. Delete the empty migration file.
7. Create a custom user model in the newly created app.
8. Point DJANGO_USER_MODEL  to the custom user.
9. Run makemigrations.

Let's begin!

Step 1
To migrate to a custom user model we first need to get rid of all the direct User references. To do that, start by adding a new property named AUTH_USER_MODEL  in settings.py like so:

```py
# core/settings.py

AUTH_USER_MODEL = 'auth.User'
```

This property tells Django what user model to use. Since we don't have a custom user model yet we'll point it to the default Django user model.

Step 2
Next, go through your entire codebase and make sure to replace all User references with AUTH_USER_MODEL or get_user_model() accordingly:

```py
# todo/models.py

class UserTask(GenericTask):
    user = models.ForeignKey(to=AUTH_USER_MODEL, on_delete=models.CASCADE)

    def __str__(self):
        return f'UserTask {self.id}'

class GroupTask(GenericTask):
    users = models.ManyToManyField(to=AUTH_USER_MODEL)

    def __str__(self):
        return f'GroupTask {self.id}'
```

Don't forget to import AUTH_USER_MODEL  at the top of the file:

from core.settings import AUTH_USER_MODEL

Also make sure that all the third-party apps/packages you use do the same. If any of them reference the User  model directly, things might break. You don't have to worry about this much since most of the popular packages that leverage the User  model don't reference it directly.

Step 3
Moving on, we need to start a new Django app, which will host the custom user model.

I'll call it users but you can pick a different name:

(venv)$ python manage.py startapp users

If you want, you can reuse an already existing app, but you need to make sure that there are no migrations within that app yet; otherwise, the migration process won't work due to Django's limitations.

Register the app in settings.py:

```py
# core/settings.py

INSTALLED_APPS = [
'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',
'todo.apps.TodoConfig',
'users.apps.UsersConfig', # new
]
```

Step 4
Next, we need to trick Django into thinking that the users app is in charge of the auth_user  table. This can usually be done with the migrate command and the -- fake  flag, but not in this case because we'll run into InconsistentMigrationHistory since most migrations depend on auth  migrations.

Anyways, to bypass this, we can use a hacky workaround. First, we'll create an empty migration, apply it so it gets saved to django_migrations  table, and then swap it with the actual auth_user  migration.

Create an empty migration within the users app:

```sh
(venv)$ python manage.py makemigrations --empty users

Migrations for 'users':
  users\migrations\0001_initial.py
```

This should create an empty migration named users/migrations/0001_initial.py.

Step 5
Migrate the database so the empty migration gets added to the django_migrations table:

```sh
(venv)$ python manage.py migrate

Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions, todo, users
Running migrations:
  Applying users.0001_initial... OK
```

Step 6
Now, delete the empty migration file:

```sh
(venv)$ rm users/migrations/0001_initial.py
```

Step 7
Go to users/models.py and define the custom User  model like so:

```py
# users/models.py

from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    class Meta:
    db_table = 'auth_user'
```

Do not add any custom fields yet. This model has to be the direct replica of Django's default user model since we'll use it to create the initial auth_user  table migration.

Also, make sure to name it User , otherwise you might run into problems because of content types. You'll be able to change the model's name later.

Step 8
Navigate to your settings.py and point AUTH_USER_MODEL  to the just created custom user model:

```py
# core/settings.py

AUTH_USER_MODEL = 'users.User'
```

If your app is not called users make sure to change it.

Step 9
Run makemigrations  to generate the initial auth_user  migration:

```sh
(venv)$ python manage.py makemigrations

Migrations for 'users':
  users\migrations\0001_initial.py
    - Create model User
```

And that's it! The generated migration has already been applied when you first ran migrate by Django's auth  app, so running migrate again won't do anything.

Add New Fields
Once you've got a custom user model set up, it's easy to add new fields.

To add a phone  and address  field, for example, add the following to the custom user model:

```py
# users/models.py

class User(AbstractUser):
    phone = models.CharField(max_length=32, blank=True, null=True) # new
    address = models.CharField(max_length=64, blank=True, null=True) # new

    class Meta:
        db_table = 'auth_user'
```

Don't forget to import models  at the top of the file:

```py
from django.db import models
```

Next, make migrations and migrate:

```sh
(venv)$ python manage.py makemigrations
(venv)$ python manage.py migrate
```

To make sure the fields have been reflected in the database bash into the Docker container:

```sh
docker exec -it django-todo-postgres bash
```

Connect to the database via psql :

```sh
root@967e9158a787:/# psql -U django-todo

psql (14.5 (Debian 14.5-1.pgdg110+1))
Type "help" for help.
```

And inspect the auth_user  table:

```sh
django-todo=# \d+ auth_user
    Table 
"public.auth_user"
    Column    |           Type           | Collation | Nullable |
Default              | Storage  | Compression | Stats target | Description
--------------+--------------------------+-----------+----------
+----------------------------------+----------+-------------+--------------
+-------------
 id           | integer                  | | not null | generated 
by default as identity | plain    | | |
 password     | character varying(128) | | not null |
| extended | | |
 last_login   | timestamp with time zone | | |
| plain    | | |
 is_superuser | boolean                  | | not null |
| plain    | | |
 username     | character varying(150) | | not null |
| extended | | |
 first_name   | character varying(150) | | not null |
| extended | | |
 last_name    | character varying(150) | | not null |
| extended | | |
 email        | character varying(254) | | not null |
| extended | | |
 is_staff     | boolean                  | | not null |
| plain    | | |
 is_active    | boolean                  | | not null |
| plain    | | |
 date_joined  | timestamp with time zone | | not null |
| plain    | | |
 phone        | character varying(32) | | |
| extended | | |
 address      | character varying(64) | | |
| extended | | |
```

You can see that the new fields named phone  and address  have been added.

Django Admin
To display the custom user model in the Django admin panel you first need to create a new class that inherits from UserAdmin  and then register it. Next, include phone  and address  in the fieldsets.

The final users/admin.py should look like this:

```py
# users/admin.py

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin

from users.models import User

class CustomUserAdmin(UserAdmin):
    fieldsets = UserAdmin.fieldsets + (
        ('Additional info', {'fields': ('phone', 'address')}),
    )

admin.site.register(User, CustomUserAdmin)
```

Run the server again, log in, and navigate to a random user. Scroll down to the bottom and you should see a new section with the new fields.

If you wish to customize the Django admin even further, take a look at The Django admin site from the official docs.

Rename User Table/Model

At this point, you can rename the user model and the table as you normally would.

To rename the user model, simply change the class name, and to rename the table change the db_table  property:

```py
# users/models.py

class User(AbstractUser): # <-- you can change me
    phone = models.CharField(max_length=32, blank=True, null=True)
    address = models.CharField(max_length=64, blank=True, null=True)

    class Meta:
        db_table = 'auth_user' # <-- you can change me
```

If you remove the db_table  property the table name will fall back to `<app_name>_<model_name>`.

After you're done with your changes, run:

```sh
(venv)$ python manage.py makemigrations
(venv)$ python manage.py migrate
```

I generally wouldn't recommend renaming anything, because your database structure will become inconsistent. Some of the tables will have the users_prefix, while some of them will have the auth_  prefix. But on the other hand, you could argue that the User  model is now a part of the users app, so it shouldn't have the auth_  prefix.

In case you decide to rename the table, the final database structure will look similar to this:

```sh
django-todo=# \dt

                     List of relations
 Schema |            Name             | Type  |    Owner
--------+-----------------------------+-------+-------------
 public | auth_group                  | table | django-todo
 public | auth_group_permissions      | table | django-todo
 public | auth_permission             | table | django-todo
 public | django_admin_log            | table | django-todo
 public | django_content_type         | table | django-todo
 public | django_migrations           | table | django-todo
 public | django_session              | table | django-todo
 public | todo_task                   | table | django-todo
 public | todo_task_categories        | table | django-todo
 public | todo_taskcategory           | table | django-todo
 public | users_user                  | table | django-todo
 public | users_user_groups           | table | django-todo
 public | users_user_user_permissions | table | django-todo
```

## Conclusion

Even though this problem of migrating to a custom user model mid-project has been around for quite a while there's still no official solution.

Unfortunately, a lot of Django developers have to go through this migration process, because the Django documentation doesn't emphasize enough that you should create a custom user model at the start of the project. Maybe they could even include it in the tutorial?

Hopefully the migration process I've presented in the article worked for you without any issues. In case something didn't work for you or you think something could be improved, I'd love to hear your feedback.

You can get the final source code from the django-custom-user repo.
