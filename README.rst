About
=====

.. DANGER::
   Beware killer rabbits! And don't use this in anything remotly
   related to a production or public deployment!!!

This is a vagrant configuration to build a development VM for the GC site/ABC.
It contains:

  * A LAMP stack
  * PhpBB 3.0.12
  * Django 1.7 (b2)
  * Not much else yet


Requirements
============

1. Have VirtualBox installed
2. Have Vagrant installed
3. Have access to KoffeinFlummi's GCWeb.


Installation
============

* Clone this repository.
* Clone Koffeinflumis GCWeb repository into the ./src/GCWeb
* Clone the gcabc_ Repository into ./src/gcabc
* Change to the repository root and run::

    vagrant up

  `vagrant up` will basicall do the following things:

  * Download a Ubuntu 14.04 vagrant image from ubuntu.com and set up a
    VM with it
  * Add port forwarding from the client to the host so you can access
    phpBB/ABC from http://localhost:10080 on the host.
  * Provision the VM with the Software above, especially:
    * Add a MySQL database for phpBB/ABC
    * Import the Tables/Data of a new phpBB installation
    * Configure Apache to serve ./projects/GCWeb


Customizations
==============

The mysql database on the client is initalized with the data from a
newly installed phpbb 3.0.12. If you want to use different data you
can replace the file `files/phpbb_initial_data.sql` with a mysql dump
of your choice *before* you build the box the first time. 

If you want to replace the data later you can replace the file 
`phpbb_initial_data.sql` and run `./bin/phpbb_reinitialize_database.sh`.


Usernames/Passwords
===================

* systems user/password: vagrant
  (default of the VM image)
* mysql db/user/password: phpbb 
  (configure in `manifests/site.pp` and used in `files/*config.php)
* phpbb admin user/password: admin/password
  (part of ` phpbb_initial_data.sql`)


Available Web services / local paths
====================================

We configure an apache virtual host on the guest that serves GCWeb
like described in it's README, and django trough a ProxyPass rule
for the urls /djangoadmin and /abc.

* Customized phpBB (with GCWeb)
  http://localhost:80 (guest), forwarded to
  http://localhost:10080 (host)
  guest path: /var/www/html
  served by apache, started: automatically
  configured by vagrant/puppet

  guest path: /vagrant/projects/GCWeb
  host path: ./projects/GCWeb

* gcabc (Prototype to implement ABC in django)
  http://localhost:80/abc +
  http://localhost:80/djangoadmin (guest), forwarded to
  http://localhost:10080/abc +
  http://localhost:10080/djangoadmin (host)

  guest path:
    * python environment: /home/vagrant/gcdjango
    * django application: /vagrant/projects/gcabc

  Needs to be started manually from the host with::

    ./bin/django_runserver.sh
    
  You can get a django shell with::
    ./bin/django_shell.sh

  Before it is usable you have to run the migrations on the host
  which is not done automatically.

  $ vagrant ssh
  vagrant@... $ cd /vagrant/projects/gcabc
  vagrant@... $ /home/vagrant/gcdjango/bin/python manage.py migrate

  If you don't start the django server you'll get a `bad gateway` error
  for /abc and /djangoadmin
