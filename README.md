apache2-vhosts
==============

This role creates a number of apache2 vhosts along with individual user acounts.
This is used in an enviroment when a number of sites are hosted and need to be
accessed by several developers. SSH is locked down so that only public key
logins are allowed.

Role Variables
--------------

There are a number of variables that can be passed. Some will be needed on a
per-host basis while others will be role based.

### Apache variables

    apache_packages:
      - apache2
      - apache2-doc
      - apache2-utils
      - apache2-mpm-itk
      - libapache2-mod-php5

    apache_modules:
      - alias.conf
      - alias.load
      - auth_basic.load
      - authn_file.load
      - ...

    apache_host: "127.0.0.1"
    apache_port: "80"
    apache_ssl_port: "443"

#### apache_packages

A list of all the apt apache2 packages that are needed. By default the
apache2-mpm-itk worker is installed so that each vhost can be run as the user
using it.

#### apache_modules

A list of all the enabled apache2 modules and the config files. Everything
listed here is symlinked from /etc/apache2/mods-available to
/etc/apache2/mods-enabled

#### apache_host

The IP adderss to bing apache2 to. Used for the Listen line in the apache config
file as well as for the VirtualHost lines in the vhost config files.

#### apache_port

The port apache2 should listen on. Used for the Listen line in the apache config
file as well as for the VirtualHost lines in the vhost config files.

#### apache_ssl_port

The SSL port apache2 should listen on. Used for the Listen line in the apache
config file as well as for the VirtualHost lines in the vhost config files.


### PHP variables

    php_packages:
      - php5-common
      - php5-cli
      - php5-suhosin
      - php5-apc
      - php-pear
      - php5-gd
      - php5-memcache
      - php5-mcrypt
      - php5-gmp
      - php5-mysql
      - php5-curl

    post_max_size: "10M"
    upload_max_filesize: "10M"
    php_timezone: "Australia/Brisbane"
    memory_limit: "32M"
    html_errors: "Off"

#### php_packages

A list of all the apt PHP packages that are needed.

#### post_max_size, upload_max_filesize, php_timezone, memory_limit, html_errors

Settings used in the php.ini file. These will likely be set per-host.


### Vhost settings

    deleted_vhost_sites:
      - {
          host: 'del.example.com',
          user: 'delexample',
          group: 'delexample',
          admin_email: 'admin@del.example.com',
          disabled: True
        }

    disabled_vhost_sites: &disabled
      - {
          host: 'dev.example.com',
          user: 'devexample',
          group: 'devexample',
          admin_email: 'admin@dev.example.com',
          disabled: True
        }

    vhost_sites:
      - *disabled
      - {
          host: 'example.com',
          user: 'example',
          group: 'example',
          admin_email: 'admin@example.com',
          ssl: {
            enabled: False,
            ssl_certificate: '/path/to/ssl/cert',
            ssl_certificate_key: '/path/to/ssl/private_key'
          },
          aliases: [
            www.example.com
            au.example.com
          ]
        }


#### deleted_vhost_sites

Any sites listed here will be sure to not be present on the server. User
accounts and public_html folders will be deleted and the site config files will
be disabled and removed.

#### disabled_vhost_sites

Sites listed here will still have their user accounts and public_html folders
present but they will not be able to log in and the vhost config file won't be
enabled within apache.

#### vhost_sites

This is a merge of disabled and any extra sites that should be enabled. It will
ensure that the user account is present and that they can log in via SSH.
public_html folders will be created and the config will be symlinked to
sites-enabled so that apache will server their content.


### SSH settings

As part of the creation of the vhosts all files within files/ssh/keys are
combined and added to each enabled user account. This allows a number of users
to log into each account as the hosting user.
