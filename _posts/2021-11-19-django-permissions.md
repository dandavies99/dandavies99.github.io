---
title: 'Fine Tuning Django User Permissions'
date: 2021-11-19
permalink: /posts/2021/11/django-permissions/
tags:
  - RSE
  - Research Software
  - Django
---

# Fine Tuning Django User Permissions
User permissions are an important consideration for any web application and the degree of complexity required will depend on the overall aim. This blog post will cover various aspects of user management and permissions within the [Django web framework](https://www.djangoproject.com/) that will likely be useful to different extents depending on your project. 

This post is aimed at developers with some familiarity of the Django framework and its authentication system who want a more granular permissions setup than is immediately available out of the box (see the five goals below). For a refresher on Django in general there's no better place than the offial [Django tutorials](https://docs.djangoproject.com/en/3.2/intro/tutorial01/). For a clear and thorough overview of permissions in Django, I highly recommend [this article](https://blog.urbanpiper.com/django-permissions/).

To give some context, I wrote this post after setting up a permissions framework for a database of scientific measurements relating to the performance of new battery materials. I'll use the main `Experiment` model in the `batteryDB` app for most examples. Overall I wanted to achieve the following five goals:
- [Goal 1](#goal-1-define-user-roles-using-groups): Have user roles with different levels of permissions. For simplicity, we will focus **read only users** who can search through the public data and **maintainers** who have full control over the scientific data.
- [Goal 2](#goal-2-set-up-groups-when-the-project-is-initialised-using-a-data-migration): Set up these user roles as soon as the project is initialised. 
- [Goal 3](#goal-3-ensure-roles-cannot-be-changed-by-locking-down-the-user-admin-page): Ensure that users cannot escalate their own privileges via Django's built in admin system.
- [Goal 4](#goal-4-enable-object-level-permissions-with-django-guardian): Have control over permissions at the object level -- not just the model level -- so that the same user can perform different actions on two instances of the same model.
- [Goal 5](#goal-5-set-object-level-permissions-automatically-using-signal-handlers): Make sure object-level permissions propagate automatically when new objects and users are created or changed. 

## Preliminaries
Before diving into the rest of the article, I'll put a quick recap here of some Django permissions basics. If you're familiar with the Django authentication system, you may want to skip this section. 
### Preliminary 1: Django Permissions 101
 Users and permissions are two key components of the Django [authentication system](https://docs.djangoproject.com/en/2.2/topics/auth/). This, like the rest of Django, is very well documented, but some key points are:

#### There are four default permissions.
As long as `django.contrib.auth` is in your `INSTALLED_APPS` [settings](https://docs.djangoproject.com/en/2.2/ref/settings/#std:setting-INSTALLED_APPS), four [default permissions](https://docs.djangoproject.com/en/2.2/topics/auth/default/#default-permissions) -- *add, delete, change, view* -- are created for each model. 

#### Permissions are essentially binary flags.
These flags determine whether users can perform a certain task on each  model. Groups can also be given permissions and users get the permissions of the groups they belong to. 

#### Assigning, removing and checking permissions is straightforward.
Assuming we have a user with username "A.User", we can grant them a permission for Experiments, after importing the relevant models:
```python
from django.contrib.auth.models import User, Permission
from django.contrib.contenttypes.models import ContentType
from batteryDB.models import Experiment

# Get the user, content_type and permission of interest
user = User.objects.get(username="A.User")
content_type = ContentType.objects.get_for_model(Experiment)
permission = Permissions.objects.get(
		codename="add_experiment", content_type=content_type
			)

# Give the user the permission
user.user_permissions.add(permission)
```

We needed to specify the `content_type` here because each permission has to be associated with one model, in this case `Experiment`. Since each model is represented by a [ContentType](https://docs.djangoproject.com/en/3.2/ref/contrib/contenttypes/#the-contenttype-model) in Django, it makes sense that each permission has a [ForeignKey](https://docs.djangoproject.com/en/3.2/topics/db/examples/many_to_one/) to ContentType. 

Checking if a user has a certain permission is a bit easier -- You can use the  `<app>.<action>_<modelname>` naming convention (note that the model is lower case):

```python
user.has_perm("batteryDB.add_experiment") # returns True
```

Removing the permission is as simple as replacing `.add()` with `.remove()`:
```python
user.user_permissions.remove(permission)
```

***Caveat**: If you try to run the above code in one go, you actually would need to reload the user from the database e.g. with `user = User.objects.get(...` before checking if a permission has been applied or removed. This is expected behaviour due to [permissions caching in Django.](https://docs.djangoproject.com/en/3.2/topics/auth/default/#permission-caching)*

#### You can add any custom permissions you want.
Given that they are basically yes/no flags, there's nothing complicated about adding extra permissions beyond the four default ones, which can be evaluated when needed. For example, we could add to the Experiment model:
```python
class Experiment:
  ...
  class Meta:
	permissions = [
	  ("change_experiment_status", "can change the status of an experiment")
	]
```

Which would allow you to check elsewhere in the code:
```python
user.has_perm("batteryDB.change_experiment_status")
```

#### Permissions are usually enforced in the _view_ layer.
 As we've seen, users either have a certain permission or they don't. The models don't enforce permissions because the model is not aware of the user performing an action. The action is usually defined in the view layer, which is why permissions are mostly enforced in the view layer too. A simple permissions check to evaluate whether a user making a request to view a list of experiments in the database might look like:
 ```python
from django.core.exceptions import PermissionDenied

def experiment_list_view(request):
  if not request.user.has_perm('batteryDB.view_experiment'):
    raise PermissionDenied()
  ...
```
 

### Preliminary 2: The custom User model
If starting a Django project from scratch, [it is highly recommended](https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project) to set up a custom user model. This gives more flexibility to extend the model should you need to in the future, for very little effort. If the `User` model turns out to be sufficient, great! You have only wasted a few minutes setting up a custom model, which works exactly like the built in `User` model. 

On the flip side, if you end up having to change the user model mid-project, this [can get messy](https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#changing-to-a-custom-user-model-mid-project) and will take significantly more time than setting up a custom model from the get-go. 

A custom model can be implemented in three steps before running `manage.py makemigrations` for the first time:

1. Define the `User` model in `models.py` of a suitably central app, or in a separate app:

```python
from django.contrib.auth.models import AbstractUser
class User(AbstractUser):
	pass
```

2. Register the model in the app's `admin.py`:

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import User

admin.site.register(User, UserAdmin)
```

3. Point `AUTH_USER_MODEL` to the new model in `settings.py`, e.g.:

```python
AUTH_USER_MODEL = "management.User"
```

If you are already too far into a project to delete and remake your migrations, and only want to store non-auth-related information about each user, [a OneToOneField to an additional model](https://docs.djangoproject.com/en/3.2/topics/auth/customizing/#extending-the-existing-user-model) will work fine. 

Finally, the `get_user_model` shortcut in `django.contrib.auth` will ensure the correct model (i.e. the one that `AUTH_USER_MODEL` is pointing to) is being used:

```python
### We no longer want to import this:
from django.contrib.auth.models import User
###

### Instead:
from django.contrib.auth import get_user_model

User = get_user_model()
###
```

## Goal 1: Define user roles using groups
We want to give users permissions based on their role and for this we will use groups. Groups also live in `django.contrib.auth.models` and users in a group have the permissions granted to that group. 

New groups can be created with any name:
```python
from django.contrib.auth.models import Group

Group.objects.create(name="Read only")
Group.objects.create(name="Maintainer")
```

A specific `user` can be added to a group:
```python
maintainers = Group.objects.get(name='Maintainer')
maintainer.user_set.add(user)
```

Assigning a `permission` to a group works slightly differently to how it works for users:
```python
maitainers.permissions.add(permission)
```

We can then check that users in the group have the correct permissions:
```python
user.has_perm('batteryDB.add_experiment') # returns True
```

***Caveat**: Similar to the previous example, you need to reload the user from the database before checking permissions have been applied due to [permissions caching](https://docs.djangoproject.com/en/3.2/topics/auth/default/#permission-caching).*

It's worth mentioning at this point that you can check what permissions a user is getting from the groups they're in vs. the permissions they have at the user level:

```
user.get_group_permissions() # returns set("batteryDB.add_experiment")
user.get_user_permissions() # returns set()
```

The philosophy of my setup is to generally have fixed sets of permissions associated with groups and not to use individual user permissions. This approach gives confidence that a user can only perform the same tasks as others in their group(s), but may not always be appropriate for your particular use case. 

Finally, remember that adding superusers to groups is pointless, because superusers [always have permission to do anything](https://docs.djangoproject.com/en/3.2/ref/contrib/auth/#django.contrib.auth.models.User.is_superuser), even if that permission doesn't exist. So if `user.is_superuser == True` then `user.has_perm(permission)` always returns `True` even if `permission` is totally made up.

## Goal 2: Set up groups when the project is initialised using a data migration 
We now know how to create groups, add users to them and assign, remove and check for permissions. What would be very handy would be the ability to build some standard groups with permissions into your project as soon as it is initialised. That way, groups with their associated permissions are ready to receive new users from day one.

Django has a neat solution for this in the form of [data migrations](https://docs.djangoproject.com/en/3.2/topics/migrations/#data-migrations). These are special migrations that change the data in the database itself rather than just the schema. 

First, create an empty migration file (in this case I have a separate app called management):
```bash
python manage.py makemigrations --empty management
```

This creates a new migration file which looks like this:
```python
from django.db import migrations

class Migration(migrations.Migration):

	dependencies = [
		("management", "0001_initial"),
	]
	
	operations = [
	]
```

Then, all we have to do is give it something to do in the list of `operations`. In a separate file within the same app called e.g. `initial_data.py` we write a function to create groups and assign permissions:
```python
from django.contrib.auth.management import create_permissions
from django.contrib.auth.models import Group, Permission

def populate_groups(apps, schema_editor):
"""
This function is run in migrations/0002_initial_data.py as an initial
data migration at project initialization. it sets up some basic model-level
permissions for different groups when the project is initialised.

Maintainer: Full permissions over the batteryDB app to add, change, delete, view
data in the database, but not users.
Read only: Not given any initial permissions. View permission is handled on a
per instance basis by Django Guardian (more on that later!).
"""

# Create user groups
user_roles = ["Read only", "Maintainer"]
for name in user_roles:
	Group.objects.create(name=name)

# Permissions have to be created before applying them
for app_config in apps.get_app_configs():
	app_config.models_module = True
	create_permissions(app_config, verbosity=0)
	app_config.models_module = None

# Assign model-level permissions to maintainers 
all_perms = Permission.objects.all()
maintainer_perms = [i for i in all_perms if i.content_type.app_label == "batteryDB"]
Group.objects.get(name="Maintainer").permissions.add(*maintainer_perms)
```
You'll notice that it was necessary to "create" permissions before applying them, even though we are only dealing with the default permissions that Django ships with. This has to do with the fact that the ContentTypes table is not created until after the migration has run, so we have to use this workaround to force the permissions into existence. See [this Django ticket](https://code.djangoproject.com/ticket/23422) for more info on this. 

Lastly, we go back to our new migration file and change it to:
```python
from django.db import migrations
from ..initial_data import populate_groups ## NEW

class Migration(migrations.Migration):

	dependencies = [
		("management", "0001_initial"),
	]
	
	operations = [migrations.RunPython(populate_groups)] ## NEW
```

And that's it! Now, when we run the migrations and start the server for the first time, the `Read only` and `Maintainer` groups will already exist, the latter having all the permissions associated with the `batteryDB` app.

We've kept the `populate_groups` function separate and imported it into the new migration for the simple reason that migrations are easily overlooked or even overwritten during code reviews. A separate python file is less likely to suffer the same fate. 

## Goal 3: Ensure roles cannot be changed by locking down the user admin page
The Django admin site does use model permissions out of the box: If the user has no permissions on a model, they can't see and access it in the admin. If they only have view and change permissions on a model, they can view and update instances but not add new ones. There is a slight quirk for being able to *add* users, in that you [must also have *change* permissions](https://docs.djangoproject.com/en/3.2/topics/auth/default/#id6) - otherwise anyone with *add_user* permissions could just create a superuser. 

This highlights just one issue that arises if you want non-superusers to interact with the Django admin site - as might well be the case for our maintainers. The default admin site needs to be tweaked in such cases and there is a very good article on this topic [here](https://realpython.com/manage-users-in-django-admin/), which I followed to produce the following custom UserAdmin:
```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from django.contrib.auth import get_user_model

User = get_user_model()

class CustomUserAdmin(UserAdmin):
  def get_form(self, request, obj=None, **kwargs):
    form = super().get_form(request, obj, **kwargs)
    is_superuser = request.user.is_superuser
    disabled_fields = set()

    # Prevent changing permissions without using groups
    if not is_superuser:
        disabled_fields |= {
            "is_superuser",
            "user_permissions",
        }
    
    # Prevent users changing own permissions
    if not is_superuser and obj is not None and obj == request.user:
        disabled_fields |= {
            "is_staff",
            "is_superuser",
            "groups",
            "user_permissions",
        }

    for f in disabled_fields:
        if f in form.base_fields:
            form.base_fields[f].disabled = True

    return form

admin.site.register(User, CustomUserAdmin)
```

Here, we have [disabled fields](https://docs.djangoproject.com/en/3.2/ref/forms/fields/#disabled) depending on the type of request being made. This gives us the flexibility that a staff user with appropriate permission (i.e. change user) would still be able to add/remove users from groups, affecting that users permissions in a controlled way, but would not be able to make them a superuser, nor add or remove individual permissions. 

## Goal 4: Enable object level permissions with Django Guardian 
There are many scenarios where you might want object-level (a.k.a. row-level) permissions, as opposed to model-level permissions, as we've seen so far. In our case, an `Experiment` with `status = "public"` should be viewable by all read only users, whereas an `Experiment` with `status = "private"` should only be viewable to the uploading user.

[Django Guardian](https://django-guardian.readthedocs.io/en/stable/) is written exactly for this purpose, is well documented and is [easily installed](https://django-guardian.readthedocs.io/en/stable/installation.html) into an existing project. We can make use of the shortcuts provided to assign and remove permissions for specific user:object combinations.

A simple function to decide on permissions might look like:
```python
from guardian.shortcuts import assign_perm
from django.contrib.auth.models import Group

def set_permissions(instance, **kwargs):
  """Set object-level permissions. The contributing user can modify the object if
     status is "private" but not if "public".
  """
  # Get permission codenames
  view = f"view_{instance._meta.model_name}"
  change = f"change_{instance._meta.model_name}"
  delete = f"delete_{instance._meta.model_name}"

  # Assign the creator view permission
  assign_perm(view, instance.user_owner, instance)

  # Assign maintainers all permissions
  group = Group.objects.get(name='Maintainer')
  for perm in [view, change, delete]:
      assign_perm(perm, group, instance)

  # Assign other users view permission if public
  if instance.status == "public":
      group = Group.objects.get(name="Read only")
      assign_perm(view, group, instance)
```
This example uses the `Experiment` model's `user_owner` attribute, but any `user` or `group` instance can be supplied. `remove_perm` works in the same way. 

A neat feature of Django Guardian is the ability to check how things are going by clicking on the "Object Permissions" button that has now appeared on change model admin pages. A table shows the user and group permissions have been applied correctly. If the model is public, it will look something like this (the `user_owner` in this case is `'dan'`):
![](https://i.imgur.com/jSaIgRQ.png)


"Can add experiment" shows as false for everyone because it is not set for this specific model *instance*. Remember that maintainers can add Experiments in general, as we set up in Goal 2, but they can't add this specific experiment because we haven't set that permission. It would make no sense to be able to add an existing instance of a model.

## Goal 5: Set object level permissions automatically using signal handlers
The last piece of the puzzle is where code from Goal 4 should go. This was the trickiest part to figure out for me but I found the closest example to what I was aiming for [here](https://coderbook.com/@marcus/how-to-restrict-access-with-django-permissions/) (at the very bottom of the post). 

Essentially, I wanted the correct permissions to be applied whenever an instance of a model was added or changed. It turns out that what I was looking for was Django [signals](https://docs.djangoproject.com/en/3.2/ref/signals/), and in particular  `post_save` signals. Signals allow *senders* to notify *receivers* that some action (e.g. saving an object) has taken place. The `post_save` signal takes place after a model's `.save()` method is called, so is exactly what we need. 

It is [good practice](https://docs.djangoproject.com/en/3.2/topics/signals/#connecting-receiver-functions) to keep signal handlers in a `signals/handlers.py` directory within the app of the models they handle. We move our function from Goal 4 into this file and modify it slightly:
```python
from django.contrib.auth.models import Group
from django.db.models.signals import post_save
from django.dispatch import receiver
from guardian.shortcuts import assign_perm, remove_perm
from ..models import Experiment

@receiver(post_save, sender=Experiment)
def set_permissions(sender, instance, **kwargs):
  """Set object-level permissions. The contributing user can modify the object if
     status is "private" but not if "public".
  """
  # Get permission codenames
  view = f"view_{instance._meta.model_name}"
  change = f"change_{instance._meta.model_name}"
  delete = f"delete_{instance._meta.model_name}"

  # Assign the creator view permission
  assign_perm(view, instance.user_owner, instance)

  # Assign maintainers all permissions
  group = Group.objects.get(name='Maintainer')
  for perm in [view, change, delete]:
      assign_perm(perm, group, instance)

  # Assign other users view permission if public
  if instance.status == "public":
      group = Group.objects.get(name="Read only")
      assign_perm(view, group, instance)

  # Remove view permissions from other users if private
  if instance.status == "private":
      group = Group.objects.get(name="Read only")
      remove_perm(view, group, instance)	
```
The first change we've made is to use the `post_save` receiver decorator to allow the function to receive signals from the `Experiment` model. 

Secondly, we have modified `set_permissions` to take a `sender` argument, which is required so that the function is only called when the `sender` specified by the `@receiver` decorator is supplied (see the docs [here](https://docs.djangoproject.com/en/3.2/topics/signals/#connecting-to-signals-sent-by-specific-senders)).

Lastly, we have used `remove_perm` when saving a private object instance. This is so that if an object already exists and is being changed to be private (rather than added new), other read only users can no longer view it. There is no harm done if the user/group doesn't already have the permission in question, so there is no need to check before performing `remove_perm()`.

Finally, to connect everything up, in `batteryDB/apps.py` we must modify the `ready()` method of `BatterydbConfig` to import the signals submodule - more details on how that works [here](https://docs.djangoproject.com/en/3.2/topics/signals/#connecting-receiver-functions).
```python
from django.apps import AppConfig

class BatterydbConfig(AppConfig):
	name = "batteryDB"
	
	def ready(self):
		import batteryDB.signals.handlers
```

And that's it! Now the actions performed in `set_permissions()` will take place every time an `Experiment` is added or changed. 

## Useful Links
As I said at the beginning, different sections of this post are probably more useful than others depending on your project. I did a lot of Googling when I was deciding how to implement my permissions system and some of the most helpful blog posts / forum answers are linked below:

- [What You Need to Know to Manage Users in Django Admin](https://realpython.com/manage-users-in-django-admin/)
- [Django: extending user model vs creating user profile model](https://stackoverflow.com/questions/29573138/django-extending-user-model-vs-creating-user-profile-model)
- [How to restrict access with Django Permissions](https://coderbook.com/@marcus/how-to-restrict-access-with-django-permissions/) (including using the `@receiver` decorator with `post_save` signals)
- [How to create groups and assign permission during project setup in django?](https://stackoverflow.com/questions/42743825/how-to-create-groups-and-assign-permission-during-project-setup-in-django)
- [Adding permissions programatically does not work as expected](https://stackoverflow.com/questions/27927159/django-1-7-adding-permissions-programmatically-does-not-work-as-expected) (Due to permissions caching, explained in the docs [here](https://docs.djangoproject.com/en/3.2/topics/auth/default/#permission-caching))