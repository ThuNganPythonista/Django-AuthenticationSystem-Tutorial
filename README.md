# Djang Tutorial - AUTHENTICATION SYSTEM(SET UP REGISTER & LOGIN BACK-END FOR YOUR WEB)

In this tutorial, I would introduce to you some basic features of authentication system in Django framework including registration, login, password reset, userprofile.

### *Take note:* ###
+ *You should have your own project that finished front-end interface of registration, login, password reset and userprofile in advance, consisting of some basic settings of Django also*


+ *One of unique features of Django is the Apps in Django. The Apps feature allows us to divide applications into independent sub-apps, each performing a specific task, and developers can easily integrate between these apps without the need for extensive source code changes. In this case, we will create an app named " User " for authentication* => This one, I wrote a post on Facebook about it.

### What will you learn ? ###

(1) Custom User Model (AbstractUser vs AbstractBaseUser)
(2) Custom Model Manager
(3) Settings, Form, Register User in Admin

### FEATURE 1 : REGISTRATION ###

**USER MODEL**:  

+ **AbstractUser**: You understand simply that Django provide you abstract class from its model module `django.contrib.auth.models`. Use `AbstractUser` when you just want to inherit from `AbstractUser` such as username, email, password, first_name, last_name, v.v. (common fields and pre-built methods). You only can remove the field "name" and add some more fields. 

  Now, you will create `models.py` file under your app :

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    date_of_birth = models.DateField(null=True, blank=True)
    def __str__(self):
          return self.username
```

Here, you already inherited `date_of_birth` and `email` from `AbstractUser`


+ **AbstractBaseUser**: Same as `AbstractUser`, the difference is that you will create a completely new based on your idea. You will define yourself fields such as username, email ... It will be the better choice if you want more control over how your user model is built. For example, n the case of a model with 3 fields: color, code, logan, if you want more flexibility and do not want to use Django's default fields like username, email, you can use AbstractBaseUser.

Similarly, you create `models.py` file under your app :

(1) Create a custom manager:

```python

from django.contrib.auth.models import BaseUserManager

class CustomUserManager(BaseUserManager):
    def create_user(self, email, date_of_birth, password=None, **extra_fields):
        if not email:
            raise ValueError('The Email field must be set')

        email = self.normalize_email(email)
        user = self.model(email=email, date_of_birth=date_of_birth, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, date_of_birth, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)

        return self.create_user(email, date_of_birth, password, **extra_fields)

```
(2)Create a custom user model:

```python

from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin
from django.db import models

class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    date_of_birth = models.DateField(null=True, blank=True)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['date_of_birth']

    def __str__(self):
        return self.email

```

**SETTINGS**:  

Include the following line in the `settings.py` file to inform Django to utilize the newly defined custom user class.








Define RegistrationForm 
 + Create forms.py in your app "user" : 

```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm


class LoginForm(forms.Form):
    email = forms.EmailField(max_length=65)
    password = forms.CharField(max_length=65, widget=forms.PasswordInput)
    

class RegisterForm(UserCreationForm):
    class Meta:
        model=User
        fields = ['username','email','password1','password2']
```

**Explain :** 

Here, we import User and UserCreationForm meaning that Django provides for coders to build user registration

So, what will we do with two libraries provided ?

=> We inherit them

1) `class LoginForm` :

We name it LoginForm to create a login form that inherit from forms.Form of Django

+ The first field, email, to define email address accepts only valid email. If you fill in a name or a text, it will return error.


+ The second field, password, is to define password input. CharField defines that this is a text and `widget=forms.PasswordInput` to hide password.


2) `class RegisterForm` :

+ Django provided for developers `UserCreationForm` including some fields such as **username**, **password 1** and **password 2**


+ `class Meta` : to overwrite, configure, annotate or customize a model class =)) For example, we define a class called "CoffeeStore", but we want its name in database which is "Starbucks". Yes, we use `class Meta`. You imagine that you have a video clip and you want to edit it, and then you use `class Meta`


+ model = `User` : Django provides `User` as a build-in model. In this case, this line of code will allow your registration form working with User, a build-in model class of Django


+ Both lines of code :
```python
    model = User
    fields = ['username','email','password1','password2']
```
 
=> We will customize model User by these fields : username, email, password1, password2

**Step 2**:  Define Register in views.py under your app User

We will import some essential module from Django

```python
    from django.shortcuts import render, redirect
    from django.contrib import messages
    from django.contrib.auth import login, authenticate, logout
    from .forms import LoginForm, RegisterForm
    from django.views import View

```

Currently, although we can use `def` to define a function for registration; however, I recommend we use `class`. It will help us add more features later.

**Now we have two cases POST or GET**

```python
  class Register(View)
    def sign_up(request):
        if request.method == 'GET':
            form = RegisterForm()
            return render(request, 'users/register.html', { 'form': form})  
        if request.method == 'POST':
            form = RegisterForm(request.POST) 
            if form.is_valid():
                user = form.save(commit=False)
                user.username = user.username.lower()
                user.save()
                messages.success(request, 'You have singed up successfully.')
                login(request, user)
                return redirect('posts')
            else:
                return render(request, 'users/register.html', {'form': form})
```

=> It means that there are two situations

+ The first one, GET method, when users click to the register button, for instance, it returns the register template (HTML). This is because the GET method will take data from server.


+ The second one, POST method, when users already filled in their information and press the button register, that information will be sent to database. This is because the POST method will handle a request sent to server.


=> Here, these lines of code already defined these two cases. This is the meaning of `class Register` in views.py. Moreover, you can go through each lines of code to find out more such as `                user = form.save(commit=False)
`, `                login(request, user)
`, ....


**Step 3**: 

When we code vanilla HTML, we will code register.html like this :

```angular2html
    <form class="form-register" method="POST" action="">
            <div class="contain-form-register">
                <div class="title-register">
                    <h2>REGISTER</h2>
                </div>
                <div class="chua-2cai-fillin">
                    <input type="text" name="fname" id="fname-input" placeholder="USERNAME">
                </div>
                <div class="chua-2cai-fillin">
                    <input type="text" name="email" id="email-input" placeholder="email">
                </div>
                <div class="chua-2cai-fillin">
                    <input type="password" name="password" id="password-input" placeholder="password">
                    {% csrf_token %}
                </div>
                <div class="contain-button-register">
                        {% csrf_token %}
                        <button type="submit">REGISTER</button>
                </div>
            </div>
            <i class="fas fa-arrow-left" style="color: #000000;"></i><span>Back to Home </span>
	    </form>
```

YES, IT'S VANILLA HTML !! BUT NOW, WE WILL NOT CODE LIKE THAT ANYMORE.

WE ALREADY DECLARE FIELDS (username, email, password1, password2) in admin module.

So, we just need to use `{{ form.as_p }}` to list those in html template.

Now, register.html should code like this :

```python
    {% extends 'base.html' %}
    
    {% block content %}
    
    <form method="POST" action="register">
        {% csrf_token %}
        <h2>Sign Up</h2>
        {{ form.as_p }}		
        <input type="submit" value="Register" />
    </form>
    
    {% endblock content%}
```

+ `        {% csrf_token %}
` : protect your web against CSRF attack (Cross-Site Request Forgery)

That's all, nothing new. Just take note `        {% csrf_token %}
` and `        {{ form.as_p }}		
`

