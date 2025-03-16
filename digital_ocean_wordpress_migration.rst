=========================================================
Migrating Wordpress site to the new Digital Ocean droplet
=========================================================

1. Create new Wordpress droplet
2. Copy /var/www/html from the old droplet to local machine
3. Dump mysql "wordpress" database and copy it to local machine
4. Paste /var/www/html from local machine to the new droplet
5. Import the database dump to the "wordpress" database on new droplet
6. Change "wordpress" database password to the one in wp-config.php
7. Update "A" DNS records to the IP of the new droplet


1. Create new Wordpress droplet
-------------------------------

Create lets encrypt certificate.


2. Copy /var/www/html from the old droplet to local machine
-----------------------------------------------------------

.. code-block::

    $ sftp root@<old_droplet_ip>
    $ get -r /var/www/html


3. Dump mysql "wordpress" database and copy it to local machine
---------------------------------------------------------------

.. code-block::

    $ ssh root@<old_droplet_ip>
    $ mysqldump wordpress > wordpress.sql
    $ sftp root@<old_droplet_ip>
    $ get wordpress.sql

4. Paste /var/www/html from local machine to the new droplet
------------------------------------------------------------

.. code-block::

    $ ssh root@<new_droplet_ip>
    $ rm -rf /var/www/html
    $ sftp root@<new_droplet_ip>
    $ put -r html /var/www/html

5. Import the database dump to the "wordpress" database on new droplet
----------------------------------------------------------------------

.. code-block::

    $ sftp root@<new_droplet_ip>
    $ put wordpress.sql
    $ ssh root@<new_droplet_ip>
    $ mysql
    $ show databases;
    $ use wordpress;
    $ drop table <table_name>, <table_name>,...;
    $ exit
    $ mysql wordpress < wordpress.sql
    $ mysql
    $ use wordpress;
    $ SELECT * FROM <table_name>

Check that tables in "wordpress" database were sucessfully imported.

6. Change "wordpress" database password to the one in wp-config.php
-------------------------------------------------------------------

    $ ssh root@<new_droplet_ip>
    $ cat /var/www/html/wp-config.php // search for DB_PASSWORD
    $ sudo myql
    $ SELECT User, Host, Db FROM mysql.db WHERE Db = 'wordpress';
    $ ALTER USER 'wordpress'@'localhost' IDENTIFIED BY '<DB_PASSWORD>';

7. Update "A" DNS records to the IP of the new droplet
------------------------------------------------------

Update DNS records for the domain to the IP of the new droplet.
