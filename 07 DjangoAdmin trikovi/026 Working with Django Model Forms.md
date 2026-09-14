
# Working with Django Model Forms

Django comes with loads of built in features making it easy to quickly build web apps from scratch. That is, if you know exactly how to build what you need.

While Django comes with extensive documentation with examples for most topics, I still find that it’s often easier to learn the right (or at least common) syntax for the feature I need in blog posts and tutorials. There seems to be a lot left out (or simply assumed to be common knowledge) in the behemoth that is the Django documentation. Of course the size of the documentation doesn’t make it easy to find things either.

Case in point: the model forms. The documentation for model forms has quite a bit of explanation on the basic uses and some more specific use cases but was lacking in some important areas. I’ll go over the basics to give newcomers a bit of background and then explain in detail the parts I had to research myself.

**Disclaimer**: After doing the research necessary for this article I’ve seen that much of what I thought was left out was in fact in the documentation spread out through other topics. Regardless, I’m happy to have done the work myself and I hope this provides a clear and concise explanation of how to work with ModelForms.

## Contents

1. Models
2. Forms
3. ModelForms
4. Customizing the Form
5. Conclusion

As always, you can get the code that I’ve written for this article on my github repository.

## Models

Django models are the easiest way to create database tables with built in CRUD functionality. Models are a parallel to database tables and are defined as classes which inherit from the django.db.models.Model . Model attributes (equivalent to rows) can be defined as class attributes using the appropriate model field class. You can find the list of all available fields here. We can create a new model describing a news article, with attributes title and body, like so:

```py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
```

The only notable difference between the CharField we used for the title and the TextField we used for the body is that Django recommends the TextField for large bodies of text.

With our new model we can create Articles and store them in our database. There are several ways we can do this, but the one we’re interested in here is with a client side form. Before we look at the ModelForm let’s take a quick look at how Django handles regular forms.

## Forms

If you’ve never used Django forms before, I suggest taking a look at my article Diving into Django Forms. Either way, I’ll give a brief summary with an example so we can continue.

Say we want to create a form for users to create new articles. We’ll need to have fields for each of the attributes in our model that we want the user to supply: title and body. We can define our own form class that inherits from the form. Form class and use it to easily create and render the HTML, set attributes and validate
the data.

```py
class ArticleForm(forms.Form):
    title = forms.CharField(label='Title: ', widget= 

forms.TextInput(attrs={'maxlength': 50, 'class':'form_input'}))
    body = forms.CharField(label='Body: ', widget= 

forms.Textarea(attrs={'class':'form_input'}))
```

Here we’re telling Django that we want our form to have a CharField (which is rendered as `<input type=”text”>)` with a max length of 50 and a class of form_input. We also define a CharField for the body but tell Django that we want to use a textarea for this one, as the body of the article can be much longer than the title.

You’ll notice that both fields in our form are very similar to those used in our model. This code is redundant and if our model was more complex this would be a very inefficient way to work. We’ll see how we can avoid repeating ourselves, in the section on ModelForms.

Now that we have our form we can render it in a template by creating an instance of it in our view, passing it to the render function using the context dictionary under the key ‘form’ and writing {{form}} wherever we need it in the template. There are other ways to render it in the template, some of which we’ll show later. If any of them are confusing feel free to check the rendering section of my forms article, linked above.

Once everything is rendered, we still need to process and validate the data received from the form, which we’ll do in the view. Afterwards, we’ll want to create a new Article object and add it to the database using the data that the user supplied. We can do all of this manually using the form and model classes that we’ve written but this isn’t very efficient. In the next few sections we’ll see how we can do this using ModelForms, but first, the hard way:

```py
if request.method == 'POST':
    # create a form instance and populate it with data from the  request:
    form = ArticleForm(request.POST)

# check whether it's valid:
if form.is_valid():
    # Create a new Article instance using the data
    new_article = 
        Article(title=form.cleaned_data['title'], 
            body=form.cleaned_data['body'])
            # Save the article to the database
    new_article.save()

# return some response:
return HttpResponse("Thanks for your data!")

# If the form was invalid send the user back to fix it
else:
    return render(request, 'formyapp/formPage.html', context)
```

As in any view that works with a form, we have separate behavior for GET and POST requests. Once we’ve received data in a post request, we validate the form using the is_valid() function and if the form is valid, we create an Article object with the form’s data. Finally, we save the new object to the database.

Note: Form.cleaned_data is a dictionary containing the data after it has been cleaned (for example properly capitalizing the title) and validated. You can read more about it in the Django documentation.

You’ll notice that in the code we’ve written we haven’t validated or cleaned the model instance (the Article) in any way. There are many situations where we could have special constraints for our forms that aren’t relevant to the models themselves and vice versa. This is one reason that there are separate methods for validating the form and validating models. Some examples:

We could have a model for users of our site and a form for them to use in order to sign up. While forms like this often have 2 password fields (one for the password input and another for confirmation) there’s no need for both to be stored in the database. The form validates that the passwords match, in order to ensure the user didn’t make a mistake, and then stores a single password for that user.

On the other hand we might want to ensure that all article titles are unique. The form has no way (while sticking to OOP principles and leaving the concerns of the Articles to be dealt with by the Article class) of knowing whether an article with a certain title already exists. Checking uniqueness of the data would be the job of the model validation.

We can also use the clean method of the form to validate interdependent fields or use the `clean_<fieldname>()` method to clean a specific field. Likewise, we can use the model’s clean method to do things like setting default values for fields left empty.

ModelForms

As you saw in the previous section creating forms to use with our models can add repetitive code and extra complexity to our project. Django recognizes the need for creating database objects from user data and provides us with a handy class called the ModelForm which makes a lot of the repetitive parts easier.

Django’s ModelForm class gives us all sorts of built in form functionality that it receives from the regular form class (creating and rendering the form fields, easy form validation etc.) and combines that with functionality we can use to easily create and save a database object. Let’s redo our article example, this time using ModelForms.

We can create a form class for our Article model by creating a new class that inherits from the ModelForm class. Django ModelForms use a nested Meta class to define many of the form’s attributes and behaviors. Let’s create a ModelForm for Articles:

```py
class ArticleForm(ModelForm):
    class Meta:
        model = Article
        fields = ‘__all__’ 
```

The model attribute defines which model the form is for and the fields attribute defines which of the model attributes we would like to use as fields in our form. We can choose all, as we have above, give a list of fields to include like so:

```py
class ArticleForm(ModelForm):
    class Meta:
        model = Article
        fields = ['title', 'body']
```

or a list of fields to exclude like this:

```py
class ArticleForm(ModelForm):
    class Meta:
        model = Article

        exclude = []
```

Next we need to create a new form instance in our view:

```py
def formRender(request):
    context = {}
    form = ArticleModelForm()
    context['form'] = form
    return render(request, 'formyapp/formPage.html', context)
```

We can then pass the form to a template and render it in several ways. I explain the main ones in detail in my article on forms so feel free to check it out if you want more details. We’ll render the form here by writing {{form.as_p}} in the django template and get the following HTML:

```html
<form action="" method="post">
    <input type="hidden" name="csrfmiddlewaretoken"
    value="BwpXGRXCdUC5p9f0FeuRKzQdQHDANpTVX2EuzLqFup60EU93BsAb">
    <p>
        <label for="id_title">Title:</label>
        <input type="text" name="title" maxlength="200" 
required="" 
        id="id_title">
    </p>
    <p>
        <label for="id_body">Body:</label>
        <textarea name="body" cols="40" rows="10" required="" 
        id="id_body"></textarea></p>
        <input type="submit" value="Submit">
</form>
```

In order to process the data, we’ll add similar logic as before, checking the method of the request and acting accordingly. For a GET request, we’ll do as we did above. For a POST request we’ll create a form based on the data we’ve received, validate it using the ModelForm’s is_valid() method and then save the object in the database by saving the form. Let’s see what the code looks like and then I’ll explain what I mean.

```py
def formRender(request):
    context = {}
    if request.method == 'POST':
        # create a form instance and populate it with data from the request:
        form = ArticleModelForm(request.POST)

        # check whether it's valid:
        if form.is_valid():
            form.save()
            return HttpResponse("Thanks for your data!")

        # If the form was invalid send the user back to fix it
        else:
            context['form'] = form
            return render(request, 'formyapp/formPage.html', context)
    # if a GET (or any other method) we'll create a blank form
    else:
        form = ArticleModelForm()
        context['form'] = form
        return render(request, 'formyapp/formPage.html', context)
```

Saving a ModelForm in this way does several things:

1. If the form has not yet been validated, it will be validated now. (If the form is invalid an error will be raised)
2. Fields that were optional in the form are given their default value if one exists.
3. A new instance of the model is saved and added to the database.

Note: save() accepts a parameter commit which defaults to true and determines whether the instance is added immediately to the database. If we set commit to false, save() returns an instance we can process further in which case we must call save() again when we are ready to store the instance.

The save() method also takes an optional instance parameter (which is supposed to be an instance of the model) which can be used to update that specific instance’s data.

Customizing the Form

There are many reasons we might want to change the form from the defaults given by Django. We might want to set up our own labels or add classes that we can use for styling. All of this can be done inside the Meta class of the ModelForm. We can set the widgets attribute to contain a dictionary with the names of fields we want to change and the widget we want to use.

```py
class ArticleForm(ModelForm):
    class Meta:
        model = Article
        fields = ‘__all__’
        widgets = {
            'body': forms.Textarea(attrs={'class':'form_input'})}
```

This is obviously very similar to the way we did it for the regular form class, but I thought I’d save you the trouble of looking back at the documentation. If you want to see what other features can be customized, though, do have a look.

Conclusion

Finding the exact syntax you’re looking for can often feel like a treasure hunt without a map and let’s be honest: sometimes all you really want is the fastest answer so you can get back to creating the amazing application you’re working on. I hope I was able to save you some time, or at the very least provide an interesting read. See you soon!
