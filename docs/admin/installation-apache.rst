.. _installation apache:

===================
Install with apache
===================

.. _Apache: https://httpd.apache.org/
.. _Apache Debian:
    https://cwiki.apache.org/confluence/display/HTTPD/DistrosDefaultLayout#DistrosDefaultLayout-Debian,Ubuntu(Apachehttpd2.x):
.. _README.Debian:
    https://salsa.debian.org/apache-team/apache2/raw/master/debian/apache2.README.Debian
.. _Apache Arch Linux:
    https://wiki.archlinux.org/index.php/Apache_HTTP_Server
.. _Apache Fedora:
    https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-apache-http-server/index.html
.. _Apache directives:
    https://httpd.apache.org/docs/trunk/mod/directives.html
.. _Getting Started:
    https://httpd.apache.org/docs/current/en/getting-started.html
.. _Terms Used to Describe Directives:
    https://httpd.apache.org/docs/current/en/mod/directive-dict.html
.. _Configuration Files:
    https://httpd.apache.org/docs/current/en/configuring.html
.. _ProxyPreserveHost: https://httpd.apache.org/docs/trunk/mod/mod_proxy.html#proxypreservehost
.. _LoadModule:
    https://httpd.apache.org/docs/2.4/mod/mod_so.html#loadmodule
.. _DocumentRoot:
    https://httpd.apache.org/docs/trunk/mod/core.html#documentroot
.. _Location:
    https://httpd.apache.org/docs/trunk/mod/core.html#location
.. _uWSGI Apache support:
    https://uwsgi-docs.readthedocs.io/en/latest/Apache.html
.. _mod_proxy_uwsgi:
    https://uwsgi-docs.readthedocs.io/en/latest/Apache.html#mod-proxy-uwsgi

.. sidebar:: further read

   - `Apache Arch Linux`_
   - `Apache Debian`_ and `README.Debian`_
   - `Apache Fedora`_
   - `Apache directives`_

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: entry

----

**Install** :ref:`apache searx site` using :ref:`filtron.sh <filtron.sh overview>`

.. code:: bash

   $ sudo -H ./utils/filtron.sh apache install

**Install** :ref:`apache searx site` using :ref:`morty.sh <morty.sh overview>`

.. code:: bash

   $ sudo -H ./utils/morty.sh apache install

----

The apache HTTP server
======================

If Apache_ is not installed, install it now. If apache_ is new to you, the
`Getting Started`_, `Configuration Files`_ and `Terms Used to Describe
Directives`_ documentation gives first orientation.  There is also a list of
`Apache directives`_ *to keep in the pocket*.

.. tabs::

   .. group-tab:: Ubuntu / debian

      .. code:: sh

         sudo -H apt-get install apache2

   .. group-tab:: Arch Linux

      .. code:: sh

         sudo -H pacman -S apache
         sudo -H systemctl enable httpd
         sudo -H systemctl start http

   .. group-tab::  Fedora / RHEL

      .. code:: sh

         sudo -H dnf install httpd
         sudo -H systemctl enable httpd
         sudo -H systemctl start httpd

Now at http://localhost you should see any kind of *Welcome* or *Test* page.
How this default intro site is configured, depends on the linux distribution
(compare `Apache directives`_).

.. tabs::

   .. group-tab:: Ubuntu / debian

      .. code:: sh

         less /etc/apache2/sites-enabled/000-default.conf

      In this file, there is a line setting the `DocumentRoot`_ directive:

      .. code:: apache

         DocumentRoot /var/www/html

      And the *welcome* page is the HTML file at ``/var/www/html/index.html``.

   .. group-tab:: Arch Linux

      .. code:: sh

         less /etc/httpd/conf/httpd.conf

      In this file, there is a line setting the `DocumentRoot`_ directive:

      .. code:: apache

         DocumentRoot "/srv/http"
         <Directory "/srv/http">
             Options Indexes FollowSymLinks
             AllowOverride None
             Require all granted
         </Directory>

      The *welcome* page of Arch Linux is a page showing directory located at
      ``DocumentRoot``.  This is *directory* page is generated by the Module
      `mod_autoindex <https://httpd.apache.org/docs/2.4/mod/mod_autoindex.html>`_:

      .. code:: apache

         LoadModule autoindex_module modules/mod_autoindex.so
         ...
         Include conf/extra/httpd-autoindex.conf

   .. group-tab::  Fedora / RHEL

      .. code:: sh

         less /etc/httpd/conf/httpd.conf

      In this file, there is a line setting the ``DocumentRoot`` directive:

      .. code:: apache

          DocumentRoot "/var/www/html"
          ...
          <Directory "/var/www">
              AllowOverride None
              # Allow open access:
              Require all granted
          </Directory>

      On fresh installations, the ``/var/www`` is empty and the *default
      welcome page* is shown, the configuration is located at::

        less /etc/httpd/conf.d/welcome.conf

.. _apache searx site:

Apache Reverse Proxy
====================

.. sidebar:: public to the internet?

   If your SearXNG instance is public, stop here and first install :ref:`filtron
   reverse proxy <filtron.sh>` and :ref:`result proxy morty <morty.sh>`, see
   :ref:`installation scripts`.  If already done, follow setup: *SearXNG via
   filtron plus morty*.

To setup a Apache revers proxy you have to enable the *headers* and *proxy*
modules and create a `Location`_ configuration for the SearXNG site.  In most
distributions you have to un-comment the lines in the main configuration file,
except in :ref:`The Debian Layout`.

.. tabs::

   .. group-tab:: Ubuntu / debian

      In the Apache setup, enable headers and proxy modules:

      .. code:: sh

         sudo -H a2enmod headers
         sudo -H a2enmod proxy
         sudo -H a2enmod proxy_http

      In :ref:`The Debian Layout` you create a ``searx.conf`` with the
      ``<Location /searx >`` directive and save this file in the *sites
      available* folder at ``/etc/apache2/sites-available``.  To enable the
      ``searx.conf`` use :man:`a2ensite`:

      .. code:: sh

         sudo -H a2ensite searx.conf

   .. group-tab:: Arch Linux

      In the ``/etc/httpd/conf/httpd.conf`` file, activate headers and proxy
      modules (LoadModule_):

      .. code:: apache

	 FIXME needs test

         LoadModule headers_module modules/mod_headers.so
         LoadModule proxy_module modules/mod_proxy.so
         LoadModule proxy_http_module modules/mod_proxy_http.so

   .. group-tab::  Fedora / RHEL

      In the ``/etc/httpd/conf/httpd.conf`` file, activate headers and proxy
      modules (LoadModule_):

      .. code:: apache

	 FIXME needs test

	 LoadModule headers_module modules/mod_headers.so
         LoadModule proxy_module modules/mod_proxy.so
         LoadModule proxy_http_module modules/mod_proxy_http.so

With ProxyPreserveHost_ the incoming Host HTTP request header is passed to the
proxied host.

.. _apache searx via filtron plus morty:

.. tabs::

   .. group-tab:: SearXNG via filtron plus morty

      Use this setup, if your instance is public to the internet, compare
      figure: :ref:`architecture <arch public>` and :ref:`installation scripts`.

      1. Configure a reverse proxy for :ref:`filtron <filtron.sh>`, listening on
         *localhost 4004* (:ref:`filtron route request`):

      .. code:: apache

         <Location /searx >

             # SetEnvIf Request_URI "/searx" dontlog
             # CustomLog /dev/null combined env=dontlog

             Require all granted

             Order deny,allow
             Deny from all
             #Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
             Allow from all

             ProxyPreserveHost On
             ProxyPass http://127.0.0.1:4004
             RequestHeader set X-Script-Name /searx

         </Location>

      2. Configure reverse proxy for :ref:`morty <searx morty>`, listening on
      *localhost 3000*

      .. code:: apache

         ProxyPreserveHost On

         <Location /morty >

             # SetEnvIf Request_URI "/morty" dontlog
             # CustomLog /dev/null combined env=dontlog

             Require all granted

             Order deny,allow
             Deny from all
             #Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
             Allow from all

             ProxyPass http://127.0.0.1:3000
             RequestHeader set X-Script-Name /morty

         </Location>

      For a fully result proxification add :ref:`morty's <searx morty>` **public
      URL** to your :origin:`searx/settings.yml`:

      .. code:: yaml

         result_proxy:
             # replace example.org with your server's public name
             url : https://example.org/morty
             key : !!binary "insert_your_morty_proxy_key_here"

         server:
             image_proxy : True

uWSGI support
=============

Be warned, with this setup, your instance isn't :ref:`protected <searx
filtron>`, nevertheless it is good enough for intranet usage.  In modern Linux
distributions, the `mod_proxy_uwsgi`_ is compiled into the *normal* apache
package and you need to install only the :ref:`uWSGI <searx uwsgi>` package:

.. tabs::

   .. group-tab:: Ubuntu / debian

      .. code:: sh

         sudo -H apt-get install uwsgi

         # Ubuntu =< 18.04
         sudo -H apt-get install libapache2-mod-proxy-uwsgi

   .. group-tab:: Arch Linux

      .. code:: sh

         sudo -H pacman -S uwsgi

   .. group-tab::  Fedora / RHEL

      .. code:: sh

         sudo -H dnf install uwsgi

The next example shows a configuration using the `uWSGI Apache support`_ via
unix sockets and `mod_proxy_uwsgi`_.

For socket communication, you have to activate ``socket =
/run/uwsgi/app/searx/socket`` and comment out the ``http = 127.0.0.1:8888``
configuration in your :ref:`uwsgi ini file <uwsgi configuration>`.  If not
already exists, create a folder for the unix sockets, which can be used by the
searx account (see :ref:`create searx user`):

.. code:: bash

   sudo -H mkdir -p /run/uwsgi/app/searx/
   sudo -H chown -R searx:searx /run/uwsgi/app/searx/

If the server is public; to limit access to your intranet replace ``Allow from
all`` directive and replace ``192.168.0.0/16`` with your subnet IP/class.

.. tabs::

   .. group-tab:: Ubuntu / debian

      .. code:: apache

	 LoadModule headers_module /usr/lib/apache2/mod_headers.so
	 LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
	 LoadModule proxy_uwsgi_module /usr/lib/apache2/modules/mod_proxy_uwsgi.so

	 # SetEnvIf Request_URI /searx dontlog
	 # CustomLog /dev/null combined env=dontlog

	 <Location /searx>

	     Require all granted
	     Order deny,allow
	     Deny from all
	     # Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
	     Allow from all

	     ProxyPreserveHost On
	     ProxyPass unix:/run/uwsgi/app/searx/socket|uwsgi://uwsgi-uds-searx/

	 </Location>

   .. group-tab:: Arch Linux

      .. code:: apache

	 FIXME needs test

         LoadModule proxy_module modules/mod_proxy.so
         LoadModule proxy_uwsgi_module modules/mod_proxy_uwsgi.so

         # SetEnvIf Request_URI /searx dontlog
         # CustomLog /dev/null combined env=dontlog

         <Location /searx>

             Require all granted
             Order deny,allow
             Deny from all
             # Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
             Allow from all

             ProxyPreserveHost On
             ProxyPass unix:/run/uwsgi/app/searx/socket|uwsgi://uwsgi-uds-searx/

	 </Location>

   .. group-tab::  Fedora / RHEL

      .. code:: apache

	 FIXME needs test

	 LoadModule proxy_module modules/mod_proxy.so
         LoadModule proxy_uwsgi_module modules/mod_proxy_uwsgi.so
         <IfModule proxy_uwsgi_module>

             # SetEnvIf Request_URI /searx dontlog
             # CustomLog /dev/null combined env=dontlog

             <Location /searx>

                 Require all granted
                 Order deny,allow
                 Deny from all
                 # Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
                 Allow from all

                 ProxyPreserveHost On
                 ProxyPass unix:/run/uwsgi/app/searx/socket|uwsgi://uwsgi-uds-searx/

	     </Location>

         </IfModule>

   .. group-tab:: old mod_wsgi

      We show this only for historical reasons, DON'T USE `mod_uwsgi
      <https://uwsgi-docs.readthedocs.io/en/latest/Apache.html#mod-uwsgi>`_.
      ANYMORE!

      .. code:: apache

         <IfModule mod_uwsgi.c>

             # SetEnvIf Request_URI "/searx" dontlog
             # CustomLog /dev/null combined env=dontlog

             <Location /searx >

                 Require all granted

                 Options FollowSymLinks Indexes
                 SetHandler uwsgi-handler
                 uWSGISocket /run/uwsgi/app/searx/socket

                 Order deny,allow
                 Deny from all
                 # Allow from fd00::/8 192.168.0.0/16 fe80::/10 127.0.0.0/8 ::1
                 Allow from all

             </Location>

         </IfModule>

.. _restart apache:

Restart service
===============

.. tabs::

   .. group-tab:: Ubuntu / debian

      .. code:: sh

         sudo -H systemctl restart apache2
         sudo -H service uwsgi restart searx

   .. group-tab:: Arch Linux

      .. code:: sh

         sudo -H systemctl restart httpd
         sudo -H systemctl restart uwsgi@searx

   .. group-tab::  Fedora / RHEL

      .. code:: sh

         sudo -H systemctl restart httpd
         sudo -H touch /etc/uwsgi.d/searx.ini


disable logs
============

For better privacy you can disable Apache logs.  In the examples above activate
one of the lines and `restart apache`_::


  # SetEnvIf Request_URI "/searx" dontlog
  # CustomLog /dev/null combined env=dontlog

The ``CustomLog`` directive disable logs for the whole (virtual) server, use it
when the URL of the service does not have a path component (``/searx``) / is
located at root (``/``).

.. _The Debian Layout:

The Debian Layout
=================

Be aware that the Debian layout is quite different from the standard Apache
configuration.  For details look at the README.Debian_
(``/usr/share/doc/apache2/README.Debian.gz``).  Some commands you should know on
Debian:

* :man:`apache2ctl`:  Apache HTTP server control interface
* :man:`a2enmod`, :man:`a2dismod`: switch on/off modules
* :man:`a2enconf`, :man:`a2disconf`: switch on/off configurations
* :man:`a2ensite`, :man:`a2dissite`: switch on/off sites
