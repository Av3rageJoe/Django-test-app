# Django Tutorial

This tutorial teaches how to use Django as specified in the Skillshare Djacgo 101 course.

## Step 1: Setting up Django

In this lab pipenv is used to create a virtual environment to work out of.

The first thing to do is to make sure pipenv is installed using brew:

`brew install pipenv`

Once installed, install Django using pipenv:

`pipenv install django`

Then enter into the virtual environment using pipenv:

`pipenv shell`

### Starting your Django project

To start your django project, run the command:

`django-admin startproject <name of project> .`

This will create several default django files.

To run your initial server, cd into the directory of your manage.py file and run the command:

`python3 manage.py runserver 0.0.0.0:8000`

This will deploy your django server at http://localhost:8000. It will also inform you that you have 18 unapplied migrations.

A migration is when your app is trying to connect to a database of some kind (in this case sqlite3), and it's trying to provision your database so that you can configure a bunch of things. To fix this issue, type:

`python3 manage.py migrate`

This will fix all of the services that have not been provisioned as of yet.

Now that your app is running, you probably want to create an admin user to configure your app at *localhost:8000/admin*

On the command line, run:

`python3 manage.py createsuperuser`

Configure the options as prompted on screen. In this case, our creds are:

username: admin
password: djangoadmin

Now when we navigate to the `/admin` webpage, we can login with our new superuser

## Step 2: Creating a new app

To start an app in Django, run the command

`python3 manage.py startapp <name of app>`

This will then create a folder for the name of your app with files such as *admin.py, views.py, apps.py etc*

To activate the app however, we need to go into the settings.py file in your django directory and add it to your INSTALLED_APPS section.

In our case, we called the app 'fixtures', and so we need to add `'feed'` to the INSTALLED_APPS section

You have now successfully created your first app inside of Django

## Your first Django model

A model is a way to write code, in the form of a python class and will automatically map to a non-visible database which we can then modify.

In this section, we're going to create a model called "post" which we will then modify.

Inside of our *feed* folder, we want to open up *models.py* and add the following snippet of code.

<code>
class Post(models.Model):
    pass
</code>

What this will do is, if we were to tellk Django to look for new changes (called migrations), it will then generate a migrations file and provision our database to have a table called feed_post. This will allow us a really simple way to create and store data.

Lets begin to modify the code. Change our class to say:

<code>
class Post(models.Model):
    text = models.CharField(max_length=140, blank=False, null=False)
</code>

Once we've added in the code, we can run the command:

`python3 manage.py makemigrations`

This will create a migrations folder inside of our feed folder. This stored all of the information needed to create the python executable file and store it as a table to be added to our database.

To add it to our database, apply it to our migration:

`python3 manage.py migrate`

Now when we go back to our django web interface, we won't see anything as the table is still not managable. We need to make it managable in order to see it. 
To make it managable, we need to go into feed/admin.py and register our model here.

It will look like:

<code>
from .models import Post

class PostAdmin(admin.ModelAdmin):
    pass

admin.site.register(Post, PostAdmin)

</code>

This will then connect the Post class with the PostAdmin class (which as of present does nothing).

Now if we look inside of our admins folder on the web interface, we can see "Posts" under a "feed" section which we can add our first post to. Go ahead and do this, and you will see a Post object created. In Django, whenever we access a piece of data or a row in our database, it is called an object.

We can go ahead and change that object name by going to models.py and in our Post class, adding the following lines:

<code>
def __str__(self):
    return self.text
</code>

This is where self.text refers to the text variable created earlier, and so now when a post is created, it's name will be the value of the text variable, If we then refresh the webpage, we can see that the name of our object has now changed. 

Now we have provisioned a table in our database and we added a row into that table. In our table we now have two columns, one called `id` and one called `text`

However, we still can't see anything on our webpage...

## Your first View

This sections will show you how to actually be able to view what you've created so far.

In order to add a path, we need to go to mysite/urls.py and add our path to the urlpatterns section.

Lets say for example we wanted to rewrite the homepage. 

**Step 1:** 

Within the feed folder, create a urls.py file. And inside of this, enter the following code:

<code>
from django.urls import path

app_name = 'feed'

urlpatterns = [
    path('', renderSomeView, name='index'),
    
]</code>

This will import the same path variable as used in the mysite/urls.py file. In the urlpatterns section, the first path argument is left blank to specify the homepage, and in the second argument we need to supply some template to be rendered. 

**Step 2:**

Delete all of the contents within feed/views.py and replace it with the following code:

<code>
from django.views.generic import TemplateView

class HomePageView(TemplateView):
    template_name = "home.html"
</code>

This will tell the class to look for a template called home.html

**Step 3:**

Within feed/urls.py, add the following code:

<code>
from .views import HomePageView
</code>

and replace `renderSomeView` with `HomePageView.as_view()`

This will render our logic for us.

**Step 4:**

Now we want to go back to mysite/urls.py and add a new path for it to utilise.

Within urlpatterns, add `path('', include(feed_urls, namespace='feed')),`

However, currently feeds_urls and include do not exist yet within this file. To make them exist, add the code:

`from django.conf.urls import include`
`from feed import urls as feed_urls`

This will then import the `include` function from django, and our urls file from the *feed* folder, renamed to be `feed_urls`

Now when we refresh the page, the page will crash. This is because it is unable to find the template that we are trying to use. We will solve this in the next lab.

To go over what we've done in this section, we've:

1. We've created and populated two urls.py files (Django loves a naming convention). These have specified the urls we want to use.
2. We created an app_name called feed within our feed/urls.py file. This matches to our namespace in the mysite/feed.py file (more on that later)
3. In our views.py, we created a HomePageView() which overwrote our default Django homepage. However, because Django couldn't find the template 'home.html' for this file, everything crashed.

## Setting up your Templates Folder

Normally django will look for templates in it's default templates folder. However in this step we'll look at changing where the django looks for a template folder and add our own template to there.

Within mysite/settings.py, if we scroll down then we can see a section labeled `TEMPLATES`. This is where django will look for templates. 

Within this section add to the `DIRS` array "TEMPLATE_DIR". However, TEMPLATE_DIR is a variable that doesn't currently exist.

Therefore, above the `TEMPLATE` section, add the variable:

`TEMPLATE_DIR = os.path.join(BASE_DIR, "templates")`

Where BASE_DIR has been specified above to look for the parents parent of the URL.

Now because we used `os`, we must also include `import os` at the top of the settings.py file. 

If we were to reload the webpage now, then we can see that Django is checking *mysite/templates/home.html* for the template. However, that also doesn't exist and so needs created. 

Under the top level mysite folder (or alongside your mysite folder if you only have one), create a folder called templates and within that a file called *home.html*

In this file, add `<h1> Hello World </h1>` as sample text, and refresh your django webpage to prove it has successfully worked.

In the real world, you will likely use the same template for multiple different files. Therefore writing out the same file over and over again gets real tedious real fast. It'd be better to create a base file and use that each time you need a strong starting point.

Create a file called *base.html* in your templates folder, and type `html:5` and hit enter. This will populate a bunch of generic HTML data.

Change the title, and give it some content in the body.

The issue currently though, is our Django app doesn't know about *base.html*. It's still trying to pull from *home.html*. Lets edit the *home.html* file to extend to the base.html.

Clear all of the contents currently in *home.html* and add in the following:

`{% extends "base.html" %}`

In the next lesson, we will learn how to inject custom info into our home.html file, while using our base.html file

## Using template blocks

A block in html looks like thos `{% block body %} {% endblock %}`. This is a block called body, and what it does is when a file is imported, you can populate this block with your own data when called.

Go ahead and out the above snippet of code into the body section of your *base.html* file.

To call the block, go into *home.html* and add in:

<code>
{% block body %}
    Stuff in here from home.html
{% endblock %}
</code>

Now if you look at the source code for our default homepage in Django, you will see everythin in base.html, plus our injected code from home.html.

Lets do the same for the title section, by changing the title in base.html to `{% block title %} {% endblock %}`

If you ever forget to include a title or body section within your import, you could include within the *base.html* blocks a generic value. For example:

`{% block body %} Whoops, something went wrong! Sorry about that... {% endblock %}`

Now, if we were to remove the body block from the *home.html* file, then we'd get our default message!

## Custom Page Context

This section will detail how to add custom content and logic to our home page.

Open up *feed/views.py*, and edit the class to contain the code:

<code>

def get_context_data(self, **kwargs):
    context = super().get_context_data(**kwargs)
    context['my_thing'] = "Hello world :P this is dynamic"

    return context

</code>

Where kwargs means key word arguments.

Now, if we go back to our *home.html* page and in the body block put double curly brackets {{}} then we can call the variable within the class. 

For example, edit the body_block within *home.html* to say:

`{{ my_thing }}`

And if we reload our webpage, then we can see the content has been updated!

To see a bunch of different classes you can use in a template, you can visit https://ccbv.co.uk

## Displaying dynamic posts

Now we're going to attempt to get some dynamic data using `get_context_data`.

First things first, go to *feed/views.py* and edit the name of the context from `'my_thing'` to `'posts'`

Then, go into your admin dashboard and add a bunch more posts. 

This is because in this lesson we're going to dynamically get these posts, from our table, and store them on our page.

So, we're going to change our `context['posts]` in the *views.py* file to call the Post class (in our *models.py* file), and display all of our posts.

Change the line of code in *views.py* to:

`context['posts'] = Post.objects.all()`

And make sure to import Post at the top of the file by adding in:

`from .models import Post`

Now, in the *home.html* file, swap out `my_thing` for `posts`

This should now display a very ugly version of our posts in the django dashboard.

In a database when we look for data, it is called a query. And so when we look up data in this instance, we use a query and get a set of data back. 

Lets try and loop through it more dynamically.

To do this, we can replace the content of our block in *home.html* with a for loop like so:

<code>

{% block body %}
    {% for post in posts %}
        {{ post.text }}
        <hr>
    {% endfor %}
{% endblock %}

</code>

This will look through the posts context, and then on each post it will call the text field, which is defined in our *models.py* file as the actual content of the individual post, and output it to the webpage! Pretty nifty huh.

## Adding images

Django has a function called `models.FileField()` (to be added to the models.py page) which allows you to upload any file to, however in this instance we will use a seperate package from [GitHub](https://github.com/jazzband/sorl-thumbnail)

To install the package, run `pip3 install sorl-thumbnail` in the command line.

Withing *settings.py* we then need to add `'sorl.thumbnail',` to the `INSTALLED_APPS` section

Following the documentation on the Github page, if we want to then add it to our model, we need to go to our models page and add the following code:

```
from sorl.thumbnail import ImageField

class Item(models.Model):
    image = ImageField(upload_to='whatever')

```

However, because we have now imported the ImageField package, we could also just call it directly within our Post class like so:


```class Post(models.Model):
    text = models.CharField(max_length=140, blank=False, null=False)
    image = ImageField()

    def __str__(self):
        return self.text
```

When we try and run our server now though, we get an error saying the package `Pillow` is not installed. Instead of installing this manually, lets now create a *requirements.txt* file which will store everything we need to import.

This file should have one line which says "Pillow" in it for now.

From the command line, run `pip3 install -r requirements.txt`

We will then need to migrate the project to apply the new thumbnail app.

Lets run `python3 manage.py migrate` to provision a new little thing in our database.

The migrations that we just ran came from the third party sorl.thumbnails. When we makemigrations here, we want to take a snapshot of what we currently have, and compare that to what we used to have, and if theres any changes in there then to make a migration file. We can do this by running the command `python3 manage.py makemigrations`.

When we run this, it notices that we have not configured a default setting for the ImageField model. At the prompt, enter `1` to make a one-off default, and then enter `1` again to see if it breaks or not.

It worked, and if we look under our *feed/migrations* folder then we'll notice that we have a new migrations file. If we look into it, then we can see that it has a dependency on the original file, alongside adding in our new field into the table called `image`

We now want to run `python3 manage.py migrate` to include the new migration

Now if we navigate to our admin dashboard and click into one of the posts, we can see an option to add an image. Huzzah! Wasn't that easy?

Lets delete all of our current posts in the admin dashboard and then create a new post to test file uploads. 

Once created, view the post and try and click on the image. Notice how it says the file doesn't exist?

In addition to this, if you look into your *mysite* directory, then you'll notice that the image has been uploaded into there. So it definitely does exist right?

Also, if we were to sya host thousands of posts with thousands of images, our directory is going to get very busy very quickly. We don't want that, we want our files to go into a specific directory. So lets go ahead and try to do that.

## Media Folder

So to setup a dedicated media folder, we need to navigate to our *settings.py* file, and tell it the destination to store mdeia files. We can do this by adding the line:

```
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')
```

This is exactly what we did with the templates folder previously, but instead this time we're using the media folder.

We now need to modify our *mysite/urls.py* file. First things first, we need to import our settings. So add the line at the top of the file:

`from django.conf import settings`

We also need to import the static library, so also add

`from django.conf.urls.static import static`

Moving down to the bottom of our file, lets add some actual code.

```
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

This goes, if the debug setting is true, then add to the urlpatterns a route to the MEDIA_URL, as defined in the settings.py file.

Now, if we change our media file uploaded in the admin console and try and access it, we can see that we go to /media and can view the file! We can also see a new folder now called **media** which will store all of the media files. 

Note this only works because debug is set to true. If it was set to false we wouldn't be able to view this page as the if statement above would fail. in a production environment, we'd want to use something like nginx to setup a reverse proxy.

In the nexr lesson, we're going to show how to add the image to our template.

## Adding images to your template

If we were to go into our *home.html* file, then we can see that we call the post by running `{{ post.text }}`. However, if we were to run `{{ post.image }}` then wall we'll get is the name of the image, and not the actual image itself. So how do we do that?

Well, we could do `{{ post.image.url }}`, but that only shows the full url. Knowing a bit of html, we could then do `<img src="{{ post.image.url }}">`. But then our image shows up huge! And it's not a small image. Displaying one of these? Okay the file isn't massive. However if we were to display a hundred? Users would have to download hundreds of megabytes of data, which will take way too long! So, what do we do?

Well, we are using sorl.thumbnail, which is sorta designed for the image to show up as a thumbnail and not a huge file with loads of data. If we reference the docs, then we can see we can write:

`{% load thumbnail %}` at the top of our home.html page, and then inside of our body put:

```
{% thumbnail post.image "200x200" crop="center" as im %}
    <img src="{{ im.url }}" width="{{ im.width }}" height="{{ im.height }}">
{% endthumbnail %}
```

Where we have made sure to reference `post.image` as where to call our image from, and inside of the image source it knows to call the image url. If we look at the image of the url on the website too, then we can see that thumbnail.sorl has made use of caching too to save us time when loading the image again! Pretty nifty huh.

In the next lesson we'll look at what happens if you were to click on an image, and how we can make it display that image in full on another page.

## Adding a detail view

To be able to click into an image and see the full uncompressed version, we use something called a "detail view". 

To do this, lets go to our views file, and add DetailView at the end of our `django.views.generic` import like so:

`from django.views.generic import TemplateView, DetailView`

We now need to create a new class for DetailView, just as we did with TemplateView. Add the following code to views.py:

```
class PostDetailView(DetailView):
    template_name = "detail.html"
    model = Post
```

Where the class inherits DetailView, and the model is inherited from the Post class in *models.py*. What we're effectively saying at this point is we have this model and we have this DetailView. If we go to a particular url then automatically fetch that image for us, that particular `Post`. 

We now need to add a new path to our *feed/urls.py* file. In the `urlpatterns` block, add the following code:

`path('detail/<int:pk>/', PostDetailView.as_view(), name='detail'),`

This way when it renders a view, it will look at the integer supplied and rename it to be that of the primary key. The PostDetailView() class will be what triggers to display the image.

Make sure to also import the PostDetailView class at the top of the page:

`from .views import HomePageView, PostDetailView`

When we view the webpage now, we'll see that if we try and navigate to one of our posts, we get a template not found error, and that's because it can't find detail.html as it doesn't exist, so lets create it.

Under our templates folder, create a page called *detail.html* and add the following code:

```

{% extends "base.html" %}

{% block title %}
    Page for the images!
{% endblock %}


{% block body %}
    This is where my image will go
{% endblock %}

```

Now lets go ahead and actually make the detail template do what it's meant to do!

## Detail template

So the question really is, how do we tell detail.html which file it needs to load?

Now, we know based on the number in the url it can work out which post it's meant to be referring to. And if we run

`{{ object }}` within the body of our file, then we get the text result back. And this is because within the Post class, it is taught to return `self.text` as the result of the object when called!

We could change it to say `object.image`, however that'll only return the name of the file as we saw before! Now knowing a bit of html, we could call `object.image.url` and stick that in an image source. After all, we want to show the full image right? 

So lets tidy up our body by putting in the following code:

```
<h3> Text </h3>
{{ object }}
<h3> Image </h3>
<img src="{{ object.image.url }}" style ='width: 100%; height: auto;'>
```

And huzzah! That now works as expected. In the next lesson we'll make the images clickable (without hardcoding!)

## Template links

Now, what we could do is just include a href in our files to the page we want to display, and leave it like that, like so:

`<a href="/detail/{{post.id}}/">View details</a>`

But, then if we change the name of the detail path, this will break everything.

Now, we know that all of the urls within feed, are under the namespace 'feed'. This means that we can say in a template, in our detail.html file, to call the url 'feed', and more specifically the 'detail' url within feed (as defined in our urls.py files) like so:

`<a href="{% url 'feed:detail' post.id %}">View details</a>`

Include the above code in your home.html file.

Then we can go ahead and tidy up or body section of our home.html file, to make all of the text and image clickable. Replace the current code with the new code:

```
    {% for post in posts %}
        <a href="{% url 'feed:detail' post.id %}" style="display: inline-block;">
            {{ post.text }} <br>
            
            {% thumbnail post.image "200x200" crop="center" as im %}
                <img src="{{ im.url }}" width="{{ im.width }}" height="{{ im.height }}">
            {% endthumbnail %}
        </a>
        <hr>
    {% endfor %}
```

Lets now add a go back button to our details page to send us back to the home page.

Within detail.html, in the body section at the top, add the following code:

`<a href="{% url 'feed:index' %}"> &larr; Go back! </a>`

Unlike before, this url doesn't take any parameters like the detail page did, needing to take in the post ID. Therefore, we can omit this.

Now, let's add an upload form.

# Adding an upload form

First things first, in our feed folder lets add a file called forms.py. In the file add the following code:

```
from django import forms

class PostForm(forms.Form):
    pass
```

And that is pretty much all we need to do to create/call a form. 

We could give it some fields with, for example, what we want to include in our form. For example, we could include a character field that accepts text, and an image field which accepts images. Lets modify the code to the below:

```
from django import forms

class PostForm(forms.Form):
    text = forms.CharField()
    image = forms.FileField()
```

Note that when an image is uploaded, there is no validation. be careful with this as we know how nasty those php files can get!

Lets now add a url for the forms page within our *feed/urls.py* file.

`path('post/', AddPostView.as_view(), name='post'),`

note that we've not actually configured our AddPostView class yet inside of our views file, so lets go ahead and do that. Remember to add `AddPostView` to the import list too.

First of all elts import the `FormView` in *views.py*. This will allow us to upload stuff to our webpage.

Now, within *views.py*, add the following code:

```

from .forms import PostForm

class AddPostView(FormView):
    template_name= "new_post.html"
    success_url = "/" #goes back to the home page
    form_class = PostForm

```

This will create the AddPostView class, which when a form is successfully uploaded will redirect to the homepage ("/"), and to access the page you need to navigate to /new_post.html, and it utilises the PostForm class created in the *forms.py* page.

Lets go ahead and create the new_post.html file in our templates folder.

Inside of it, include the following code:

```
{% extends "base.html" %}

{% block title %}
    Upload a new image!
{% endblock %}


{% block body %}
    <p>Empty</p>
{% endblock %}
```

Now lets try and render the form to actually show up. We can do this by calling the form in a template like so:

```
{% block body %}
    <p>Empty</p>
    {{ form.as_p }}
{% endblock %}
```

This will display the form class with its text and image characteristics in a paragraph form.

However, right now it does nothing. So lets go ahead and create a POST form that sends data when a button is clicked:

```
{% block body %}
    <form method="POST" action="">
        {{ form.as_p }}
        <div>
            <button type="submit">Submit form!</button>
        </div>
    </form>

{% endblock %}
```

However, if we try and submit the button, it will fail as there is no CSRF token. Pretty nifty that security is thought about!

Adding this is very simple too. Inside of the form, simply include the code `{% csrf_token %}`

However, when we submit our data still nothing happens!

Well, first things first, when we submit a file we need to include the `enctype`. So lets go ahead and do that.

```
{% block body %}
    <form method="POST" action="" enctype="multipart/form-data">
        {{ form.as_p }}
        <div>
            <button type="submit">Submit form!</button>
        </div>
        {% csrf_token %}
    </form>

{% endblock %}
```

Now we get redirected back to the homepage as expected. However, if we look in our backend then our post still doesn't exist (somewhat expectedly!)

We have a method called `def form_valid` which takes the arguments `(self, form)`. This will redirect us back to our success_url when triggered, if the form is in fact valid.

Inside of our views file, lets add in that method.

```
class AddPostView(FormView):
    template_name= "new_post.html"
    success_url = "/" #goes back to the home page
    form_class = PostForm

    def form_valid(self,form):
        print("This was valid!!")
        print("This was valid!!")
        print("This was valid!!")
        print("This was valid!!")
        return super().form_valid(form)
```

Now if we upload a valid form, we can see in our terminal that it is valid. Lets go ahead now and try to create a new post.

```
class AddPostView(FormView):
    template_name= "new_post.html"
    success_url = "/" #goes back to the home page
    form_class = PostForm

    def form_valid(self,form):
        #create a new post

        new_object = Post.objects.create(
            text = form.cleaned_data['text']
            image = form.cleaned_data['image']
        )

        return super().form_valid(form)
```

Where we call the Post class to create a new objext, with `forms.cleaned_data` pulling the values of the text and image fields.

Now if we try and post our file, it works!

However, we still have no link to it on our home page :\(

Let's go ahead and create this:

`<a href="{% url 'feed:post' %}"> Add a new post! &rarr </a>`

In the next lesson we'll look to make the newest posts appear at the top of the feed

## Sorting Default Posts

Right now our posts are in order from oldest to newest. Lets go ahead and reverse this

Now in our home.html class, we loop through the posts from start to finish by calling the HomePageView class and extracting the 'post' field.

By modifying the code in our class, we can order our posts by default so that whatever is the newest post has it's id returned first. We do this by going into our views.py file and adding the order_py function to one line in our HomePageView class like so:

`context['posts'] = Post.objects.all().order_by('-id')`

In the next lesson we'll look at Django messages.

## Django Message Framework

The Django Message Framework allows for sending a little bit of information from one page to another, but it only does it once and won't do it again unless prompted.

So in this case we want to create a message when a user submits an image saying 'hey you created an image!'. This should be fairly simple as it's already baked into Django

So first things first, we want to go to our *views.py* file and import the messages app. We also need to make sure that it's in our TEMPLATES[context_processors] section of *settings.py* too, in order to be able to import it.

So within *views.py*, add the following line at the top:

`from django.contrib import messages`

Now we want to scroll down to our form class to create the message displayed once the post is created.

Add the messages line to our `form_valid` function like so:

```
    def form_valid(self,form):
        #create a new post

        new_object = Post.objects.create(
            text = form.cleaned_data['text'],
            image = form.cleaned_data['image'],
        )
        messages.add_message(request, messages.SUCCESS, 'Your post was successful')
        return super().form_valid(form)
```

We can view the settings for the contrib.messages app by looking at teh django docs online.

To display the message, we're going to add some code to the body of our base.html file like so:

```
    {% if messages %}
        <ul class="messages">
            {% for message in messages %}
                <li{% if message.tags %} class="{{ message.tags }}"{% endif %}>{{ message }}</li>
            {% endfor %}
        </ul>
    {% endif %}
```

What this will do is check if there is a message (which gets fired when a new post is successfully created), check to see if there are any tags, which is our SUCCESS tag, and then display the message.

However, if we try and upload an image now, then it will fail as it's trying to use an argument called request which has never been passed (and according to ccbv.co.uk, the form_valid function doesn't take the request parameter). So what do we do?

Well looking at the ccbv docs for the FormView class, we can see that a bunch of functions do take requests. One of which is the dispatch() function.

So we want to use this function earlier on in the class to set the `self.request` parameter. The only reason we're doing this, is so that we can access it anywhere else in our class using self.request.

So lets add the following code above our message function:

```
def dispatch(self, request, *args, **kwargs):
    self.request = request
    return super().dispatch(request, *args, **kwargs)
```

Note that the return function literally does nothing, it returns exactly what it was provided initially. In classes, the functions are run when the class is initiated, and in order too therefore there is no need to call it. We can change our messages function now to say `self.request` and it should all work!

## Adding bootstrap

Now we're going to add in a front end theme using bootstrap 5. To do this, go to https://getbootstrap.com and take the href link to the cdn.

`<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">`

Now we want to add this on every page, so lets use our *base.html* file to do that. Add the above href to the head of base.html. Now if we refresh our website then things look ever so slightly different. 

Now we want to add a navbar, and we can get this code by going to the navbar section of getbootstrap.com. Choose a navbar from the docs and copy & paste the code into the body of *base.html*

Edit the names of the headers to whatever you feel like, and change the urls to match that of what you need. Below is what we've used in this example:

```
    <nav class="navbar navbar-expand-lg bg-body-tertiary">
        <div class="container-fluid">
          <a class="navbar-brand" href="/">Image Sharer</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
              <li class="nav-item">
                <a class="nav-link active" aria-current="page" href="{% url 'feed:index' %}">Home</a>
              </li>
              <li class="nav-item">
                <a class="nav-link" href="{% url 'feed:post' %}">Add New Image</a>
              </li>
            </ul>
            <form class="d-flex" role="search">
              <input class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
              <button class="btn btn-outline-success" type="submit">Search</button>
            </form>
          </div>
        </div>
      </nav>
```

Now we want to add a container to centre things (Note that containers in html are very different to containers as we know them). In our *base.html* file, add

```    
<div class="container"> 
    {% block body %} Whoops, something went wrong! Sorry about that... {% endblock %}
</div>
```

We'll also delete the post section now too from *home.html*, since we have it in our navbar

Now our navbar section is taking up a lot of space. However, luckily for us what we can do is **include** it as a template, and then just call the template. Create a folder called includes and in that folder create a file called navbar.html in our templates folder and cut all of our navbar code into there.

Now lets try and make the images look nicer but turning them into cards. In this instance we went to getbootstrap.com, looked up code for some cards and added it to our home.html file to make things look nicer.

Here is the code we took from bootstrap:

```
<div class="card" style="width: 18rem;">
  <img src="..." class="card-img-top" alt="...">
  <div class="card-body">
    <h5 class="card-title">Card title</h5>
    <p class="card-text">Some quick example text to build on the card title and make up the bulk of the card's content.</p>
    <a href="#" class="btn btn-primary">Go somewhere</a>
  </div>
</div>
```

And then here is our original home.html code:

```
{% for post in posts %}
    <a href="{% url 'feed:detail' post.id %}" style="display: inline-block;">
        {{ post.text }} <br>
        
        {% thumbnail post.image "200x200" crop="center" as im %}
            <img src="{{ im.url }}" width="{{ im.width }}" height="{{ im.height }}">
        {% endthumbnail %}
    </a>
    <hr>
{% endfor %}
```

We then modified it to incorporate the cards style, and now it looks like so:

```
{% for post in posts %}

<div class="card" style="width: 18rem;">
    {% thumbnail post.image "200x200" crop="center" as im %}
        <img src="{{ im.url }}" width="{{ im.width }}" height="{{ im.height }}">
    {% endthumbnail %}
    
    <div class="card-body">
        <h5 class="card-title"> {{ post.text }} </h5>
        <a href="{% url 'feed:detail' post.id %}" class="btn btn-primary">Go to image</a>
    </div>
</div>
<hr>

{% endfor %}
```

Lets go ahead and modify things ever so slightly, by having more than one image on a row:

```
<div class="row">
    {% for post in posts %}
        <div class="col-sm-6 col-md-3">
            <div class="card" style="width: 18rem;">
                {% thumbnail post.image "200x200" crop="center" as im %}
                <img src="{{ im.url }}" width="{{ im.width }}" height="{{ im.height }}">
                {% endthumbnail %}
                
                <div class="card-body">
                <h5 class="card-title"> {{ post.text }} </h5>
                <a href="{% url 'feed:detail' post.id %}" class="btn btn-primary">Go to image</a>
                </div>
            </div>
            <hr>
        </div>
    {% endfor %}
</div>
```

