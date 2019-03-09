==============================
Deploy Django on Ubuntu server
==============================

Prerequisites
-------------

1. Registered domain (example.com throughout this cheatsheet)
2. Server running Ubuntu 16.04
3. Nonroot user account with sudo privileges
4. SSH access to the user accounts (root and nonroot)

Configuring Domains for Staging and Live
----------------------------------------

Point your staging and live domains to the server. Create two 'A-records', which point your domains at your server's IP address.

Type: A / Hostname: example.com / Value: directs to <your server IP address>
Type: A / Hostname: staging.example.com / Value: directs to <your server IP address>

Deploy basic copy of the staging site
-------------------------------------

1. Create directory for the source files somewhere in the home folder of nonroot user. Each site (staging or live) should have its own folder, which will contain a checkout of the source code managed by git, along with the database, static files and virtual environment.

/home/user/sites
    staging.example.com
        subfolders & files

    example.com
        subfolders & files

2. Push your local code to one of the code-sharing sites like GitHub and clone it into the corresponding folder on your server.

.. code-block::

    $ export SITENAME=staging.example.com
    $ git clone <https to your repository at GitHub> ~/sites/$SITENAME

3. Create virtual environment and install dependencies.

.. code-block::

    # create virtual environment named virtualenv with python3.6
    $ python3.6 -m venv virtualenv

    # install dependencies
    $ ./virtualenv/bin/pip install -r requirements.txt

4. Create database.

.. code-block::

    $ ./virtualenv/bin/python manage.py migrate

5. Run your application and leave it running.

.. code-block::

    $ ./virtualenv/bin/python manage.py runserver 0.0.0.0:8000


6. Confirm that application works from the server.

.. code-block::

    # open second SSH server shell
    $ curl localhost:8000

7. Reach application from your local machine.

.. code-block::

    # open local terminal
    $ curl staging.example.com:8000
