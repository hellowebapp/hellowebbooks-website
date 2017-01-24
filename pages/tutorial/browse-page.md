# Setting Up Basic Browse Pages

Right now our app allows people to sign up, create a `Thing`, and then edit
their `Thing`. However, the only way for users to browse all the `Thing`s your
web app contains is by going to your homepage. Let's set up a basic browse page.

***Note:** We won't be setting up text search, like Google, due to it being way
more complex and beyond the scope of this book. If you're interested in learning
more about how to do this, check out Haystack
([http://hellowebapp.com/28](http://hellowebapp.com/28)).*

As we only have a few fields on our model, let's build a page that just lists
every `Thing` we have by name.

You'll probably want to add browse pages for each of those fields as you fill
out your model with new fields for what you're specifically building. For
example, my first Django project had fields for price range and location, and
therefore "narrow by price" and "narrow by location" pages. We're going to build
a simple example here, which is easily copied for other use-cases.

## Set up your URL routing

Back to the familiar URLs-views-templates routine. Add in this new line:

{title="urls.py", lang="python", line-numbers=on, starting-line-number=28}
```
# our new browse flow
# leanpub-start-insert
url(r'^browse/name/$',
    views.browse_by_name, name='browse'),
url(r'^browse/name/(?P<initial>[-\w]+)/$', 
    views.browse_by_name, name='browse_by_name'),
# leanpub-end-insert

# password reset URLs
url(r'^accounts/password/reset/$', 
    password_reset,
    {'template_name': 'registration/password_reset_form.html'},
    name="password_reset"),
```

We're going to set up two new URLs at once: The first, which we'll list out
every `Thing`'s name with a link to its page. The second will allow us to
specify a letter (*/browse/name/a/* to search for names of `Thing`s that start
with "a," for example.)

## Set up the view

Next, off to your *views.py* to add this new view:

{title="views.py", lang="python", line-numbers=on, starting-line-number=98}
```
def browse_by_name(request, initial=None):
    if initial:
        things = Thing.objects.filter(name__istartswith=initial)
        things = things.order_by('name')
    else:
        things = Thing.objects.all().order_by('name')

    return render(request, ‘search/search.html', {
        'things': things,
        'initial': initial,
    })
```

We're going to write a view that works for both URL routes. Neat, right?

The top line of the view where we have `initial=None`, this simply says that if
there is no value passed in, then to assign it to `None`. So */browse/*, which
doesn't pass in initial letter, will have initial assigned to `None`. If you
didn't have the `None` part, Python would whine that it expects something if
it's empty.

Then we use an if-statement to determine what kind of database query we want to
do, whether we are passing something in or not.

We're also using `name__istartswith`, another queryset filter like `contains`
which we used before (More info:
[http://hellowebapp.com/29](http://hellowebapp.com/29)). Note that we don't
*have* to use a single letter to search — this code will work whether you
searched for 'a' or 'hello', grabbing everything in your database that starts
with your query.

Fairly simple, and making one view rather than two means less code, and ergo
less maintenance. 

## Create the template

Last, our template. We're going to create one "search" template that'll work for
both flows here and we can extend in the future for other browse cases.

Create a new directory for search in your templates, and then create your
*search.html* file:

{linenos=off}
```
$ cd collection/templates/
collection/templates $ mkdir search
collection/templates $ cd search
collection/templates/search $ touch search.html
```

***Note: **Technically we're browsing, not searching, but I find calling the
pages and process "search" easier.*

Our *search.html* file:

{title="search.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Browse - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>
        Browse Things{% if initial %} Starting with 
        '{{ initial|title }}'{% endif %}
    </h1>

    {% for letter in 'abcdefghijklmnopqrstuvwxyz' %}
    <a href="{% url 'browse_by_name' initial=letter %}" 
    {% if initial == letter %}class="active"{% endif %}>
        {{ letter|upper }}
    </a>
    {% endfor %}

    {% for thing in things %}
    <ul>
        <li>
            <a href="{% url 'thing_detail' slug=thing.slug %}">
                {{ thing.name }}
            </a>
        </li>
    </ul>
    {% empty %}
    <p>Sorry, no results!</p>
    {% endfor %}
{% endblock content %}
```

![](images/browse_page.png) 

We're doing quite a bit of fun things here.

* First, our headline. We're modifying the headline based on whether we're
    narrowing down by initial or not. Users will get a descriptive headline for
    both cases.
* Note that in the headline, we have `initial|title` — one of the template tags
    we talked about earlier in Chapter 4. Why not `|upper`? I mentioned before
    this actually works with any letter or word, and chose `|title` so, if we
    searched for "hello" it would be displayed as "Hello rather than "HELLO." 
* To display links for every letter in the alphabet, we're looping over the
    alphabet in the template. Again, less code (as compared to writing out all
    the links) means less maintenance. 
* We're adding an `active` class to the currently selected letter, which you can
    style using CSS.
* We're looping over the lowercase alphabet but making it uppercase when
    displayed. This way our links look like `/browse/name/a/` rather than
    `/browse/name/A/`. Both work, but the former looks better.
* We loop over every result in our list and display a link to the `Thing`.
* Last, `{% empty %}` is what the for loop will display if the list we pass in
    is empty.

## Updating your nav and setting up redirects

There isn't a link to this page in the main nav, so we should update the nav:

{title="base.html", lang="html+django", line-numbers=on,starting-line-number=22}
```
<li>
    <a href="{% url 'contact' %}">Contact</a>
</li>
# leanpub-start-insert
<li>
    <a href="{% url 'browse' %}">Browse</a>
</li>
# leanpub-end-insert
```

The second nitpick is our URL — */browse/name/* works, as well as
*/browse/name/a/*... but */browse/* will 404 since we haven't set up a route for
it.  It's not linked to, but someone might edit the URL in their browser. Let's
redirect */browse/* page to */browse/name/* (adding `permament` makes it a 301
redirect, rather than 302).

Updated *urls.py*:

{title="urls.py", lang="python", line-numbers=on,starting-line-number=2}
```
# added RedirectView to this import statement
from django.views.generic import (TemplateView, 
# leanpub-start-insert
    RedirectView,
# leanpub-end-insert
)
```

{title="urls.py", lang="python", line-numbers=on,starting-line-number=28}    
```
    # our new redirect view
# leanpub-start-insert
    url(r'^browse/$', RedirectView.as_view(
        pattern_name='browse', permanent=True)),
# leanpub-end-insert
    url(r'^browse/name/$',
        views.browse_by_name, name='browse'),
    url(r'^browse/name/(?P<initial>[-\w]+)/$',
        views.browse_by_name, name='browse_by_name'),
```

Like `TemplateView`, which we used before to display a template without writing
a view (for our *About* and *Contact* pages), `RedirectView` is another generic
view that let's us avoid writing extra code and sets up a simple redirect.

We have another set of URLs that could benefit from this same treatment. Let's
redirect */things/* to our browse page as well:

{title="urls.py", lang="python", line-numbers=on,starting-line-number=22}
```
# leanpub-start-insert
    url(r'^things/$', RedirectView.as_view(
        pattern_name='browse', permanent=True)),
# leanpub-end-insert
    url(r'^things/(?P<slug>[-\w]+)/$',
        views.thing_detail, name='thing_detail'),
    url(r'^things/(?P<slug>[-\w]+)/edit/$',
        views.edit_thing, name='edit_thing'),
```

Test the redirects out by going to *http://localhost:8000/browse/* and
*http://localhost:8000/things/* and watch them automatically redirect to the
correct pages.

There we go, a great start to building browsable pages for your website
visitors. Commit your work!