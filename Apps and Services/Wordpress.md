---
created: Tue 2023-05-02 @ 06:32 PM
modified: Wed 2023-12-27 @ 01:39 PM
---
Each subheaders (VM, Docker, LXC / CT) are different ways you can install this. Out of all of these, I think the LXC / CT turnkey approach is the most user friendly while providing the best security out of the box.

All this being said, I don't use Wordpress any longer. But these are my install notes for when I did.


# Installation Options #

## VM ##

- Create the VM or use an existing VM
- Login via terminal

```
sudo apt update && sudo apt upgrade -y

sudo apt install apache2 apt-transport-https mariadb-server mariadb-client php php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl} -y

sudo ufw app list

sudo ufw allow in "Apache Full"

sudo ufw allow OpenSSH

sudo ufw allow Samba

sudo ufw enable

sudo mysql_secure_installation
```

```
Secure MariaDB options:

1. Hit Enter for none
2. N (Switch to unix_socket authentication)
3. N (Change the root password)
4. Y (Remove anonymous users)
5. Y (Disallow root login remotely)
6. Y (Remove test database and access to it)
7. Y (Reload privilege tables now)
```

```
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Change the DirectoryIndex line to look like this:
```
<IfModule mod_dir.c>  
        DirectoryIndex index.php index.cgi index.pl index.html index.xhtml index.htm  
</IfModule>
```

```
sudo usermod -aG www-data administrator

sudo mysql -u root -p
```

```
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

CREATE USER 'wordpress'@'%' IDENTIFIED BY 'password123';

GRANT ALL ON wordpress.* TO 'wordpress'@'%';

QUIT;
```

```
sudo nano /etc/apache2/sites-available/yourwebsite.com.conf
```

```c
<VirtualHost *:80>
     ServerAdmin youremail@gmail.com
     ServerName yourwebsite.com
     ServerAlias www.yourwebsite.com
     DocumentRoot /var/www/yourwebsite.com/
</VirtualHost>
```

```
cd /tmp

curl -O https://wordpress.org/latest.tar.gz

tar xzvf latest.tar.gz

touch /tmp/wordpress/.htaccess

cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php

mkdir /tmp/wordpress/wp-content/upgrade

sudo cp -a /tmp/wordpress/. /var/www/yourwebsite.com

sudo rm latest.tar.gz

sudo rm -rf wordpress

sudo chown -R www-data:www-data /var/www/yourwebsite.com

sudo find /var/www/yourwebsite.com/ -type d -exec chmod 750 {} \;

sudo find /var/www/yourwebsite.com/ -type f -exec chmod 640 {} \;

sudo a2ensite yourwebsite.com.conf

sudo a2dissite 000-default.conf

sudo a2enmod rewrite

sudo systemctl restart apache2

curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

- Copy the output into a note somewhere and insert in place of the temporary code in this next file:

```
sudo nano /var/www/yourwebsite.com/wp-config.php
```

```php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the installation.
 * You don't have to use the web site, you can copy this file to "wp-config.php"
 * and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * Database settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config-php/
 *
 * @package WordPress
 */

define('FORCE_SSL_ADMIN', true);
if ($_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
$_SERVER['HTTPS']='on';

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** Database username */
define( 'DB_USER', 'wordpress' );

/** Database password */
define( 'DB_PASSWORD', 'password123' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

/**#@+
 * Authentication unique keys and salts.
 *
 * Change these to different unique phrases! You can generate these using
 * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
 *
 * You can change these at any point in time to invalidate all existing cookies.
 * This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/**#@-*/
define('FS_METHOD', 'direct');

/**
 * WordPress database table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the documentation.
 *
 * @link https://wordpress.org/support/article/debugging-in-wordpress/
 */
define( 'WP_DEBUG', false );

/* Add any custom values between this line and the "stop editing" line. */

define('WP_SITEURL','https://yourwebsite.com');
define('WP_HOME','https://yourwebsite.com');

/* That's all, stop editing! Happy publishing. */

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
```

- Go to Nginx Proxy Manager and point yourwebsite.com at the VM IP address, port 80 and set the SSL certificate

- Go to http://192.168.70.72 to set up Wordpress. It's pretty straightforward. It should also have redirected the address to yourwebsite.com

- That's it. Consult https://www.flash2hack.com/?p=543 and https://www.flash2hack.com/?p=603 for any issues.

=========

- Here is some alternate apache conf code to play with as needed

```
<VirtualHost *:80>

ServerAdmin youremail@gmail.com

DocumentRoot /var/www/html/wordpress
ServerName yourwebsite.com
ServerAlias www.yourwebsite.com

<Directory /var/www/html/wordpress/>

Options FollowSymLinks
AllowOverride All
Require all granted

</Directory>

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```



## Docker (don't) ##

- This method is not recommended for production, only development
- Much more difficult to use a domain name with docker install
- The code is not included here since it is unsupported. This note is only meant as a reminder to not go down the docker path for Wordpress, in my and lots of others opinion.



## LXC / CT ##

* Use Turnkey Linux Wordpress CT template

* The setup is pretty straightforward

* After initial OS installation, login to the server via SSH and attach the container.

Check your current version:
```
php --version
```

* As of this writing it shows as 
	```
	PHP 7.3.31-1~deb10u1
	```

```
apt update

apt upgrade -y
```

```
apt -y install lsb-release apt-transport-https ca-certificates

wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg

echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

apt update

apt -y install php7.4

php --version
```

* Confirm the PHP version is now 7.4

* Install missing modules:

```
apt-get install php7.4-{bcmath,bz2,intl,gd,mbstring,mysql,zip,curl,dom,imagick}
```

* Disable the old PHP and enable the new one

```
a2dismod php7.3

systemctl restart apache2

a2enmod php7.4

systemctl restart apache2
```

* In Wordpress, go to Tools-->Site Health. All of the PHP issues should be solved.
	* Go [here](https://dev.to/samtarling/upgrading-apache-from-php-7-3-to-php-7-4-3a3h) if there are still any PHP issues

* At this point it's a good idea to run update and upgrade again.
	* It'll want to upgrade 7.3 but that should have no effect on 7.4 and/or Wordpress.

```
apt update

apt upgrade -y
```

```
cd /var/www/wordpress
```

Edit the config so that it will work will the domain name and NPM

```
nano wp-config.php
```

Add this after the beginning commented out instructions:

```
define('FORCE_SSL_ADMIN', true);
if ($_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
$_SERVER['HTTPS']='on';
```

Further down look for this section and make it look like this:

```
// Single-Site (serves any hostname)
// For Multi-Site, see: https://www.turnkeylinux.org/docs/wordpress/multisite
define('WP_SITEURL','https://yourwebsite.com');
define('WP_HOME','https://yourwebsite.com');
```


# Theme #

Go in Appearance-->Themes

* Install and activate the Baskerville theme


# Email #

- Follow this for now
	- https://wpmailsmtp.com/docs/how-to-set-up-the-gmail-mailer-in-wp-mail-smtp
- Note to self: Although this is a fairly easy setup, I don't think it needs to be this complex. Next time you need to set up email again, look into using the built in PHP option or something else BEFORE using this.