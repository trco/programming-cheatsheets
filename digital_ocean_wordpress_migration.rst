=========================================================
Migrating Wordpress site to the new Digital Ocean droplet
=========================================================

1. Create new Wordpress droplet
2. Copy /var/www/html from the old droplet to the local machine
3. Dump mysql "wordpress" database and copy it to the local machine
4. Paste /var/www/html from local machine to the new droplet
5. Import the database dump to the "wordpress" database on new droplet
6. Change "wordpress" database password to the one in wp-config.php
7. Update "A" DNS records to the IP of the new droplet


1. Create new Wordpress droplet
-------------------------------

Select Wordpress droplet from marketplace and create Lets Encrypt certificate in the process of spinning of the droplet.


2. Copy /var/www/html from the old droplet to the local machine
---------------------------------------------------------------

.. code-block::

    $ sftp root@<old_droplet_ip>
    $ get -r /var/www/html


3. Dump mysql "wordpress" database and copy it to the local machine
-------------------------------------------------------------------

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
    # check that import was successful
    $ SELECT * FROM <table_name>;

6. Change "wordpress" database password to the one in wp-config.php
-------------------------------------------------------------------

.. code-block::

    $ ssh root@<new_droplet_ip>
    # search for DB_PASSWORD
    $ cat /var/www/html/wp-config.php
    $ sudo myql
    $ SELECT User, Host, Db FROM mysql.db WHERE Db = 'wordpress';
    $ ALTER USER 'wordpress'@'localhost' IDENTIFIED BY '<DB_PASSWORD>';

7. Update "A" DNS records to the IP of the new droplet
------------------------------------------------------

Update "A" DNS records for the domain to the IP of the new droplet.

8. Setup Apache for Wordpress
-----------------------------

.. code-block::

    $ ssh root@<new_droplet_ip>
    
    # delete all files in /etc/apache2/sites-enabled
    # delete all files in /etc/apache2/sites-enabled
    # paste config below into the file: /etc/apache2/sites-available/<config_name>.conf
    
    # enable required modules and follow instructions
    $ sudo a2enmod rewrite
    $ sudo a2enmod headers
    $ sudo a2enmod expires
    $ sudo a2enmod ssl
    
    # enable site configuration
    $ sudo a2ensite <config_name>
    $ sudo systemctl restart apache2
    
    # setup ssl ceritficate
    $ sudo apt update
    $ sudo apt install certbot python3-certbot-apache
    $ sudo certbot --apache -d kvickekvacke.com -d www.kvickekvacke.com

.. code-block::

<VirtualHost *:80>
    ServerName kvickekvacke.com
    ServerAlias www.kvickekvacke.com
    DocumentRoot /var/www/html
    
    <Directory /var/www/html>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/kvickekvacke-error.log
    CustomLog ${APACHE_LOG_DIR}/kvickekvacke-access.log combined
</VirtualHost>
