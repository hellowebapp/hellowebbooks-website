# Adding a Registration Page

Right now our web app, if launched, would let our users browse the items and
objects we list but with no way for them to sign up for an account and to create
their own objects. Shall we fix that?

## Installing our first third party plugin for registration

One of the best things about Django and its open-source community is the sheer
number of plugins that developers created to make others' (and our) lives
easier.

We're going to install a plugin called *django-registration-redux*
([http://hellowebapp.com/23](http://hellowebapp.com/23)) that'll help us set up
everything we need for user registration.

We're going to use pip to install. Make sure you're in your virtual environment,
then type this into your command line:

{linenos=off}
    $ pip install django-registration-redux==1.3

Next, you'll need to tell Django that we've installed this plugin — like we did
when we added `collection` but this time it's an external app we're installing.
Head to `settings.py` and add `registration` to the list:

{title="settings.py", lang="python", line-numbers=on,starting-line-number=32}
```
INSTALLED_APPS = (
    'collection',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'django.contrib.humanize',
# leanpub-start-insert
    'registration',
# leanpub-end-insert
)
```

We'll also need to add more setting to our *settings.py* file, according to
django-registration-redux's documentation
([http://hellowebapp.com/24](http://hellowebapp.com/24)). Add this line at the
bottom of *settings.py*:

{linenos=off, line-numbers=on,starting-line-number=88}
```
ACCOUNT_ACTIVATION_DAYS = 7
```

django-registration-redux will set up account activation emails automatically
for new users, and this required setting lets us specify the number of days the
user has to activate the account before the ability to use the account expires.
Let's keep it at the default `7` for now.

Last, django-registration-redux comes with some migration files that we need to
apply to our database. Apply these using `python manage.py migrate`:

```
$ python manage.py migrate
Operations to perform:
  Apply all migrations: sessions, admin, collection, contenttypes, auth, registration
Running migrations:
  Rendering model states... DONE
  Applying registration.0001_initial... OK
  Applying registration.0002_registrationprofile_activated... OK
  Applying registration.0003_migrate_activatedstatus... OK
```

### Setting up the URLs

We're going to basically import django-registration-redux's URLs for use in our
app, which is fairly simple.

Head over to `urls.py` and add the lines indicated:

{title="urls.py", lang="python", line-numbers=on,starting-line-number=19}
```
from django.conf.urls import url, 
# leanpub-start-insert
                             include

    url(r'^accounts/', 
        include('registration.backends.simple.urls')),
# leanpub-end-insert
    url(r'^admin/', admin.site.urls),
]
```

The new line basically says, "For any URL path starting with 
*accounts/*, search for a matching URL path in django-registration-redux's URLs."

### Add "email" ability to your app

Django comes with the ability to send emails if you set up an email server, or
you can just output the contents of the emails directly to your command line in
a similar window to where `python manage.py runserver` is running. Let's set up
the latter (we can add actual email functionality when we launch the app).

Head back to *settings.py* and add these lines to the bottom:

{title="settings.py", lang="python", line-numbers=on,starting-line-number=90}
```
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
DEFAULT_FROM_EMAIL = 'testing@example.com'
EMAIL_HOST_USER = ''
EMAIL_HOST_PASSWORD = ''
EMAIL_USE_TLS = False
EMAIL_PORT = 1025
```

While we're here, let's tell Django what page we want to redirect to after
successful registration. Add after the code block above:

{title="settings.py", lang="python", line-numbers=on,starting-line-number=97}
```
LOGIN_REDIRECT_URL = "home"
```

### Adding in the required templates

django-registration-redux also assumes that we have several templates already
added to our app (More info:
[http://hellowebapp.com/25](http://hellowebapp.com/25)). We're going to create a
new folder in our templates directory to hold these new templates:

{linenos=off}
```
$ cd collection/templates/
collection/templates $ mkdir registration
collection/templates $ cd registration
collection/templates/registration $ touch registration_form.html
```

Continue to use `touch` to create blank files in this directory with the
following file names:

* registration_form.html (already created)
* registration_complete.html
* login.html
* logout.html

Thank goodness that the code inside of these templates will be fairly easy to
create.

### Start filling in our templates

The majority of code in these templates is just text or a form, which is created
by Django. For the sake of brevity, I recommend the following content for each
file:

{title="login.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Login - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Login</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
{% endblock content %}
```

{title="logout.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Logout - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Logged Out</h1>
    <p>You have been logged out!</p>
{% endblock content %}
```

{title="registration_complete.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Registration Complete - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Registration Complete</h1>
    <p>Your account has been registered!</p>
{% endblock content %}
```

{title="registration_form.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Registration Form - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Registration Form</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
{% endblock content %}
```

### Add links to nav to login and logout

The last thing we should do is add a link to login and logout in our navigation.
Update your *base.html* layout file:

{title="base.html", lang="html+django", line-numbers=on, starting-line-number=25}
```
# leanpub-start-insert
    {% if user.is_authenticated %}
    <li>
        <a href="{% url 'auth_logout' %}">Logout</a>
    </li>
    {% else %}
    <li>
        <a href="{% url 'auth_login' %}">Login</a>
    </li>
    <li>
        <a href="{% url 'registration_register' %}">Register</a>
    </li>
    {% endif %}
# leanpub-end-insert
</ul>
```

Note the two extra links we're adding to the navigation: `{% if
user.is_authenticated %}` does what it looks like it does — returns `True` if
the current user is logged into your app or not. Django by default automatically
adds the current user for use in templates.

Everything you need for someone to create an account, activate their account,
and login or logout should all be set up now! Open up your website and check it
out:

![](images/reg_form.png) 

Try creating a couple new users. 

![](images/reg_complete.png) 

Back in your admin (*http://localhost:8000/admin/*) you can see the new users
created: 

![](images/admin_new_users.png) 

Unfortunately there's no way yet for our new users to start adding `Thing`s —
users can only register, log in, and log out at the moment. 

## Setting up password reset functionality

Django comes with a lot of native password reset functionality that we just need
to activate by setting up the appropriate URLs and templates.

Like earlier, here's the full list of templates to create in the registration
directory:

* password_change_done.html
* password_change_form.html
* password_reset_complete.html
* password_reset_confirm.html
* password_reset_done.html
* password_reset_email.txt
* password_reset_form.html

And the content to put in the templates — quite a few files, but very simple
content.

{title="password_change_done.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Password Change Done - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Password Change Done</h1>
    <p>Your password was changed.</p>
{% endblock content %}
```

{title="password_change_form.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Password Change Form - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Password Change Form</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
{% endblock content %}
```

{title="password_reset_complete.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Password Reset Complete - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Password Reset Complete</h1>
    <p>Your password has been reset!</p>
{% endblock content %}
```

{title="password_reset_confirm.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Confirm Password Reset - {{ block.super }}
{% endblock title %}

{% block content %}
    {% if validlink %}
    <h1>Confirm Password Reset</h1>
    <p>Please enter your new password twice so we can verify you
    typed it in correctly.</p>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Change password" />
    </form>
    {% else %}
    <h1>Password reset unsuccessful</h1>
    <p>The password reset link was invalid, possibly because it has
    already been used. Please request a new password reset.</p>
    {% endif %}
{% endblock content %}
```

{title="password_reset_done.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Password Reset Done - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Password Reset Complete</h1>
    <p>Check your email for a link to reset your password!</p>
{% endblock content %}
```

{title="password_reset_email.txt", lang="html+django"}
```
{% autoescape off %}
You're receiving this email because you requested a password
reset. Please go to the following page and choose a new 
password:
{% block reset_link %}
{{ protocol }}://localhost:8000{% url 
    'django.contrib.auth.views.password_reset_confirm'
    uidb64=uid token=token %}
{% endblock %}
Your username, in case you've forgotten: {{ user.username }}
{% endautoescape %}
```

{title="password_reset_form.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Password Reset Form - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Password Reset Form</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
{% endblock content %}
```

### Setting up the URLs

We'll need to tell Django we're using its password reset/recover feature, so add
these URLs to your *urls.py*:

{title="urls.py", lang="python"}
```
...
# leanpub-start-insert
from django.contrib.auth.views import (
   password_reset, 
   password_reset_done,
   password_reset_confirm,
   password_reset_complete
)
# leanpub-end-insert
```

{title="urls.py", lang="python", line-numbers=on,starting-line-number=22}
```
    # the new password reset URLs
# leanpub-start-insert
    url(r'^accounts/password/reset/$', 
        password_reset,
        {'template_name':
        'registration/password_reset_form.html'},
        name="password_reset"),
    url(r'^accounts/password/reset/done/$',
        password_reset_done,
        {'template_name':
        'registration/password_reset_done.html'},
        name="password_reset_done"),
    url(r'^accounts/password/reset/(?P<uidb64>[0-9A-Za-z]+)-(?P<token>.+)/$', 
        password_reset_confirm,
        {'template_name':
        'registration/password_reset_confirm.html'},
        name="password_reset_confirm"),
    url(r'^accounts/password/done/$', 
        password_reset_complete,
        {'template_name':
        'registration/password_reset_complete.html'},
        name="password_reset_complete"),
# leanpub-end-insert
    url(r'^accounts/',
        include('registration.backends.simple.urls')),
    url(r'^admin/', admin.site.urls),
]
```

We're overriding the default URL paths that come with Django so we can point to
the template we created. Otherwise, Django will just point to the admin
templates, which wouldn't match the style and layout of our web app.

Also, note that our import statement is wrapped in parentheses - when your
imports get too long, add parentheses so the statement can span multiple lines.
Otherwise, Python will throw an "unexpected indent" error.

### Adding a link for password reset

Password change and password reset sound similar, but a password change is for
users who are already logged in, and password reset is for users who can't log
in because they forgot their password. Therefore, users should reset their
passwords from the login page. Update your login template to the following:

{title="login.html", lang="html+django"}
```
{% extends 'base.html' %}
{% block title %}
    Login - {{ block.super }}
{% endblock title %}

{% block content %}
    <h1>Login</h1>
    <form role="form" action="" method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <input type="submit" value="Submit" />
    </form>
    # leanpub-start-insert
    <p>
        <a href="{% url 'password_reset' %}">
            Forgot your password?
        </a>
    </p>
    # leanpub-end-insert
{% endblock content %}
```

Check out your website in the browser and play around with registering new
"users," logging in, and logging out.

When you put in an email to reset the password for an account, check out your
command line window to see your "email", which should look something like the
below:

{linenos=off}
```
[27/Dec/2014 00:29:34] "GET /accounts/password/reset/ HTTP/1.1" 200 1079
MIME-Version: 1.0
Content-Type: text/plain; charset="utf-8"
Content-Transfer-Encoding: 7bit
Subject: Password reset on localhost:8000
From: testing@example.com
To: tracy+auser@limedaring.com
Date: Sat, 27 Dec 2014 00:29:47 -0000
Message-ID: <20141227002947.9192.76935@Orion.local>
You're receiving this email because you requested a password 
reset for your user account at localhost:8000.

Please go to the following page and choose a new password:

http://localhost:8000/accounts/password/reset/Mg3xw-7efc197726f67cb60e84/
Your username, in case you've forgotten: auser

Thanks for using our site!
The localhost:8000 team
```

Paste the link into your browser to complete the password reset process:

![](images/confirmpassword.png)

![](images/resetcomplete.png)

Now you see how password resetting functionality is built into Django and just
needs a few templates created to access it. 

Next chapter, we're going to tie these new users to objects in our database, so
someone can create an account and a new object at the same time, as well as log
in and update their object as well. Commit your work if you haven't already!