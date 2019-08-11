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

A-record #1
Type: A
Hostname: example.com
Value: directs to <your server IP address>

A-record #2
Type: A
Hostname: staging.example.com
Value: directs to <your server IP address>

Deploy basic copy of the staging site
-------------------------------------

1. Create directory for the source files somewhere in the home folder of nonroot user. Each site (staging or live) should have its own folder, which will contain a checkout of the source code managed by git, along with the database, static files and virtual environment.

/home/user/sites/staging.example.com

/home/user/sites/example.com

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

5. Run your application via Django development server and leave it running.

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

Install and configure Nginx web server
--------------------------------------

``Nginx`` is a free, open-source, high-performance HTTP server and reverse proxy, as well as an IMAP/POP3 proxy server.

1. Install and start Nginx.

.. code-block::

    $ sudo apt install nginx
    $ sudo systemctl start nginx

2. Configure Nginx to send requests to Django by creating Nginx ``config`` file.

Save this config to ``staging.example.com`` file inside of ``/etc/nginx/sites-avaliable`` folder.
This ``config`` file says that it will listen to ``staging.example.com`` domain and will 'proxy' all requests to the local port 8000 where it expects to find Django waiting to respond.

.. code-block::

    server {
        listen 80;
        server_name staging.example.com www.staging.example.com;

        location / {
            proxy_pass http://localhost:8000;
        }
    }

3. Add ``config`` to the enabled sites by creating ``symlink`` to it.

That's the Ubuntu's preferred way of saving Nginx configurations with the real config file in ``sites-available``, and a symlink in ``sites-enabled``.

.. code-block::

    $ export SITENAME=staging.example.com
    $ cd /etc/nginx/sites-enabled/
    $ sudo ln -s /etc/nginx/sites-available/$SITENAME $SITENAME

    # check that symlink has worked
    $ readlink -f $SITENAME

4. Reload Nginx and restart server.

.. code-block::

    $ sudo systemctl reload nginx
    $ cd ~/sites/$SITENAME
    $ ./virtualenv/bin/python manage.py runserver 8000

Visit staging.example.com in your browser.

Debugging tips and commands:

.. code-block::

    # test config file for errors, Nginx error logs go into /var/log/nginx/error.log
    $ sudo nginx -t

Switching to Gunicorn
---------------------

``Gunicorn`` is a Web Server Gateway Interface (WSGI) server implementation that is commonly used to run Python web applications.

1. Install Gunicorn.

.. code-block::

    $ ./virtualenv/bin/pip install gunicorn

2. Start your application with Gunicorn.

.. code-block::

    # example is folder of your Django project containing wsgi.py file
    $ ./virtualenv/bin/gunicorn example.wsgi:application

While Django's development server serves static file, Gunicorn doesn't. Configure Nginx to server static files.

3. Run collectstatic to copy all static files to folder where Nginx can find them.

.. code-block::

    $ ./virtualenv/bin/python manage.py collectstatic

4. Configure Nginx to serve static files collected in previous point.

Add second location directive to the Nginx config.

.. code-block::

    server {
        listen 80;
        server_name staging.example.com www.staging.example.com;

        location /static {
            alias /home/user/sites/staging.example.com/static;
        }

        location / {
            proxy_pass http://localhost:8000;
        }
    }

5. Reload Nginx and restart Gunicorn.

.. code-block::

    $ sudo systemctl reload nginx
    $ ./virtualenv/bin/gunicorn example.wsgi:application

Visit staging.example.com in your browser.

Switching to using Unix sockets
-------------------------------

Unix domain sockets are used by Nginx and Gunicorn to talk to each other and enable that multiple web applications (staging and live) can be served from the same server with single IP address.

Nginx proxy passes the requests to Gunicorn, which now listens to the configured Unix socket instead to the default http://localhost:8000 set under paragraph ``Install and configure Nginx web server``.

1. Update Nginx proxy settings.

.. code-block::

    server {
        listen 80;
        server_name staging.example.com www.staging.example.com;

        location /static {
            alias /home/user/sites/staging.example.com/static;
        }

        location / {
            proxy_pass http://unix:/tmp/staging.example.com.socket;
        }
    }

2. Reload Nginx and restart Gunicorn to listen on a configured socket.

.. code-block::

    $ sudo systemctl reload nginx
    $ ./virtualenv/bin/gunicorn --bind unix:/tmp/staging.example.com.socket example.wsgi:application

Visit staging.example.com in your browser.

Using environment variables to adjust settings for production
-------------------------------------------------------------

1. Install django-environ.

.. code-block::

    $ ./virtualenv/bin/pip install django-environ

2. Create .env file inside of the main folder sites/staging.example.com and add variables to it.

.. code-block::

    DEBUG=False
    SITENAME=staging.example.com
    SECRET_KEY=82jy%x5^g+ln5yoxf(y-yxu9+r#l4wynb8-09=naf58nfmeog=
    EMAIL_USE_TLS=True
    EMAIL_HOST=smtp.gmail.com
    EMAIL_HOST_USER=example@gmail.com
    EMAIL_HOST_PASSWORD=test1234
    EMAIL_PORT=587
    DEFAULT_FROM_EMAIL=example@gmail.com
    DATABASE_NAME=testdb
    DATABASE_USER=testuser
    DATABASE_PASSWORD=test1234

3. Setup variables in settings.py.

You should use separate settings file (for example dev.py) for local development.

.. code-block::

    settings.py

    import os
    import environ


    env = environ.Env(
        # set casting, default value
        DEBUG=(bool, False)
    )
    # reading .env file
    environ.Env.read_env('.env')

    DEBUG = env('DEBUG')
    SECRET_KEY = env('SECRET_KEY')

    # example of email settings
    EMAIL_USE_TLS = env('EMAIL_USE_TLS')
    EMAIL_HOST = env('EMAIL_HOST')
    EMAIL_HOST_USER = env('EMAIL_HOST_USER')
    EMAIL_HOST_PASSWORD = env('EMAIL_HOST_PASSWORD')
    EMAIL_PORT = env('EMAIL_PORT')
    DEFAULT_FROM_EMAIL = env('EMAIL_HOST_USER')
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

    # example of Postgres database settings
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': env('DATABASE_NAME'),
            'USER': env('DATABASE_USER'),
            'PASSWORD': env('DATABASE_PASSWORD'),
            'HOST': 'localhost',
            'PORT': '',
        }
    }

4. Fix ALLOWED_HOSTS with Nginx.

By default Nginx strips out the Host headers from requests it forwards. Original host headers are forwarded if proxy_set_header is added to Nginx conf.

.. code-block::

    server {
        listen 80;
        server_name staging.example.com www.staging.example.com;

        location /static {
            alias /home/user/sites/staging.example.com/static;
        }

        location / {
            proxy_pass http://unix:/tmp/staging.example.com.socket;
            proxy_set_header Host $host;
        }
    }

    $ sudo systemctl reload nginx
    $ ./virtualenv/bin/gunicorn --bind unix:/tmp/staging.example.com.socket example.wsgi:application

Visit staging.example.com in your browser.

Start Gunicorn automatically
----------------------------

Server should start Gunicorn automatically on boot or should reload it if it crashes.

1. Create Systemd config file.

Save this config to ``gunicorn-staging.example.com.service`` file inside of ``/etc/systemd/system`` folder.

.. code-block::

    [Unit]
    Description=Gunicorn server for staging.example.com

    [Service]
    Restart=on-failure
    User=test
    WorkingDirectory=/home/user/sites/staging.example.com
    EnvironmentFile=/home/user/sites/staging.example.com/.env

    ExecStart=/home/user/sites/staging.example.com/virtualenv/bin/gunicorn \
    --bind unix:/tmp/staging.example.com.socket \
    example.wsgi:application

    [Install]
    WantedBy=multi-user.target

After changing Systemd config file always run ``deamon-reload`` before ``systemctl restart`` to see the effect of the changes.

2. Tell Systemd to start Gunicorn with the ``systemctl`` command.

.. code-block::

    # tell Systemd to load config file
    $ sudo systemctl daemon-reload

    # tell Systemd to always load the service on boot
    $ sudo systemctl enable gunicorn-staging.example.com

    # start the service
    $ sudo systemctl start gunicorn-staging.example.com

Running Gunicorn manually with ``./virtualenv/bin/gunicorn --bind unix:/tmp/staging.example.com.socket example.wsgi:application`` is not needed anymore, since Gunicorn now runs all the time.

Debugging tips and commands:

.. code-block::

    # check Systemd logs
    $ sudo journalctl -u gunicorn.staging.example.com

    # check the validity of the service configuration
    $ systemd-analyze verify /etc/systemd/system/gunicorn-staging.example.com.service

    # start the service
    $ sudo systemctl start gunicorn-staging.example.com
