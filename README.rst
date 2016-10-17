django-emailmgr
===============

Django Email Manager (emailmgr) is an application that helps with post-registration email management".
You can associate multiple email addresses to a single Django User.

This application comes into the picture only after a user has been created, activated
and logged in.

If your requirements is for email management prior to user registration please take a 
look at ``django-registration`` or ``django-email-confirmation``.

This application was inspired by bitbucket.org's email management system as well as 
the mentioned applications.


Installation
============

* Make sure you have python 2.6+ and can install from PyPI
* Use ``easy_install``: ``easy_install django-emailmgr``
* Use pip: ``pip install django-emailmgr``
* Install with Git from GitHub

  a. Run ``git clone http://github.com/un33k/django-emailmgr``
  b. ``cd django-emailmgr``
  c. run ``python setup.py``

* Install from a zip file

  a. Download ``wget https://github.com/un33k/django-emailmgr/zipball/master``
  b. Unzip the downloaded file
  c. Change into django-emailmgr directory
  d. Run ``python setup.py``


Usage
=====

* Add ``emailmgr`` to ``INSTALLED_APPS``, right after all other Django specific Apps
* Follow the instruction in the ``Current Features`` at the top of this file for usage.
* Use the templates in test directory as an example to create your own templates
* Include this application's urls or create your own urls for this application
* Run ``python manage.py syncdb`` or ``python manage.py migrate`` and enjoy


Current Features
----------------

1. This app subscribes to the ``post_save`` signals for (user) and creates an email address object with the following attributes

   * email = user.email (if not blank)
   * is_active = True
   * is_primary = True
   * is_activation_sent = Don't care
   * identifier = a random string (used for activation)
   
   Notes:

   * User login is not required
   * Email is only created if user has a valid email address
   * This email is automatically considered as primary and activation is skipped

2. Latch on the ``post_delete`` signals on (user) and deletes all email addresses associated with the (just) deleted user

3. Provides URL to:

   A. Adds an email address to the logged in user's account:

      * ``/email/add/``
      * Email address is created and associated with the logged in user
      * Email address remains inactive and cannot be made primary
      * User is redirected to ``/email/list/``
      * ``email_list`` and ``email_form`` are passed into the template

   B. Deletes an email address from a user account

      * ``/email/delete/<identifier>/``
      * Existing email address with the above identifier is deleted
      * Primary email address cannot be delete
      * Once the email is deleted, user is redirected to ``/email/list/``

   C. Sends activation link for a newly added email address (sendto = user's primary email address)

      * ``/email/send_activation/<identifier>/``
      * An activation email is sent to the logged in user's primary address
      * Note: all emails remain inactive unless activated
      * Once the email is sent, user is redirected to ``/email/list/``

   D. Activates an email address when user clicks on an activation link

      * ``/email/activate/<identifier>/``
      * Note: link was emailed to user's primary email address
      * Matched email will be activated (then eligible to be promoted to primary)
      * Once the email is activated, user is redirected to ``/email/list/``

   E. Makes an activated email address the primary email

      * ``/email/make_primary/<identifier>/``
      * Only activated email addresses can be promoted to be the primary email address
      * ``User.email`` is set to the newly promoted primary email address
      * Note: Only one email address can be set to primary
      * Once the email is made primary, user is redirected to ``/email/list/``

4. More to come ... patches & enhancements are welcomed (http://github.com/un33k/django-emailmgr)

Assumptions
-----------

* Django user has been created

  * Either via proper registration and activation or via the admin interface or scripts

* This application first looks for its templates in: ``EMAIL_MGR_TEMPLATE_PATH``, then it looks for ``<template_dir>/emailmgr/``

  * This way, projects can place the required templates at a location of their choice relative to the <tempalate_dir>

* Three templates are found in the template directory as stated above

  * email subject template: ``emailmgr_activation_subject.txt``

    * extra context: ``current_site``

  * email message body template: ``emailmgr_activation_message.txt``

    * extra context: ``current_site``, ``activate_url`` & ``user``

  * email list & manipulation template: ``emailmgr_email_list.html``

    * extra context: ``emails_list`` and ``add_email_form``

      * ``email_list`` includes all emails for the users

        * ``add_email_form`` is a form for adding a new email address to a user

        * sorted by:

          1. primary - set directly via django or by emailmgr
          2. activated - confirmed emails and can be set primary
          3. not activated but activation email sent
          4. not activated and activation email not sent


TODO
=====

* Add more goodies
