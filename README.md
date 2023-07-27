====================
django-organizations
====================

.. start-table

.. list-table::
    :stub-columns: 1

    * - Summary
      - Groups and multi-user account management

    * - Status
      - |docs| |travis| |version| |wheel| |supported-versions| |supported-implementations|


Targets & testing
-----------------

The codebase is targeted and tested against:

* Django 3.2.x against Python 3.7, 3.8, 3.9, 3.10
* Django 4.0.x against Python 3.8, 3.9, 3.10
* Django 4.1.x against Python 3.8, 3.9, 3.10

To run the tests against all target environments, install `tox
<https://testrun.org/tox/latest/>`_ and then execute the command::

    tox

Fast testing
------------

Testing each change on all the environments takes some time, you may
want to test faster and avoid slowing down development by using pytest
against your current environment::

    pip install -r requirements-test.txt
    py.test

Supply the ``-x`` option for **failfast** mode::

    py.test -x

Project goals
-------------

django-organizations should be backend agnostic:

1. Authentication agnostic
2. Registration agnostic
3. Invitation agnostic
4. User messaging agnostic

Etc.

Installing
==========

First add the application to your Python path. The easiest way is to use
`pip`::

    pip install django-organizations

Check the `Release History tab <https://pypi.org/project/django-organizations/#history>`_ on
the PyPI package page for pre-release versions. These can be downloaded by specifying the version.

You can also install by downloading the source and running::

    $ python setup.py install

By default you will need to install `django-extensions` or comparable libraries
if you plan on adding Django Organizations as an installed app to your Django
project. See below on configuring.

Configuring
-----------

Make sure you have `django.contrib.auth` installed, and add the `organizations`
application to your `INSTALLED_APPS` list:

.. code-block:: python

    INSTALLED_APPS = (
        ...
        'django.contrib.auth',
        'organizations',
    )

Then ensure that your project URL conf is updated. You should hook in the
main application URL conf as well as your chosen invitation backend URLs:

.. code-block:: python

    from organizations.backends import invitation_backend

    urlpatterns = [
        ...
        url(r'^accounts/', include('organizations.urls')),
        url(r'^invitations/', include(invitation_backend().get_urls())),
    ]

Auto slug field
~~~~~~~~~~~~~~~

The standard way of using Django Organizations is to use it as an installed app
in your Django project. Django Organizations will need to use an auto slug
field which are not included. By default it will try to import these from
django-extensions, but you can configure your own in settings. The default:

.. code-block:: python

    ORGS_SLUGFIELD = 'django_extensions.db.fields.AutoSlugField'

Alternative:

.. code-block:: python

    ORGS_SLUGFIELD = 'autoslug.fields.AutoSlugField'

Previous versions allowed you to specify an `ORGS_TIMESTAMPED_MODEL` path. This
is now ignored and the functionality satisfied by a vendored solution. A
warning will be given but this *should not* have any effect on your code.

- `django-extensions <http://django-extensions.readthedocs.org/en/latest/>`_
- `Django AutoSlug <https://github.com/justinmayer/django-autoslug/>`_
- `django-slugger <https://gitlab.com/dspechnikov/django-slugger/>`_

Registration & invitation backends
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify a different invitation backend in your project settings, and
the `invitation_backend` function will provide the URLs defined by that
backend:

.. code-block:: python

    INVITATION_BACKEND = 'myapp.backends.MyInvitationBackend'


Usage Overview
==============

For most use cases it should be sufficient to include the app views directly
using the default URL conf file. You can customize their functionality or
access controls by extending the base views.

There are three models:

* **Organization** The group object. This is what you would associate your own
  app's functionality with, e.g. subscriptions, repositories, projects, etc.
* **OrganizationUser** A custom `through` model for the ManyToMany relationship
  between the `Organization` model and the `User` model. It stores additional
  information about the user specific to the organization and provides a
  convenient link for organization ownership.
* **OrganizationOwner** The user with rights over the life and death of the
  organization. This is a one to one relationship with the `OrganizationUser`
  model. This allows `User` objects to own multiple organizations and makes it
  easy to enforce ownership from within the organization's membership.

The underlying organizations API is simple:

.. code-block:: python

    >>> from organizations.utils import create_organization
    >>> chris = User.objects.get(username="chris")
    >>> soundgarden = create_organization(chris, "Soundgarden", org_user_defaults={'is_admin': True})
    >>> soundgarden.is_member(chris)
    True
    >>> soundgarden.is_admin(chris)
    True
    >>> soundgarden.owner.organization_user
    <OrganizationUser: Chris Cornell>
    >>> soundgarden.owner.organization_user.user
    >>> <User: chris>
    >>> audioslave = create_organization(chris, "Audioslave")
    >>> tom = User.objects.get(username="tom")
    >>> audioslave.add_user(tom, is_admin=True)
    <OrganizationUser: Tom Morello>




