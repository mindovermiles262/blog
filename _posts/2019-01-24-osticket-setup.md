---
title: Secure Installation of OSTicket
date: 2019-01-24
layout: post
author: andrew
categories: [linux, "system administration" ]
---

## Using Ubuntu 18.04 with Nginx and MySQL

At my work we had been using a crappy ticketing system. It took more time to close tickets than to solve the actual issues inside of the ticket. I ended up writing a selenium script to automate the task but it was still annoying. When I returne from vacation one time I ran the script against the 80 tickets in the queue from when I was gone. It took 4 1/2 hours to run.  It was time for a new ticketing system.

I ended up choosing osTicket because it was highly recommended on /r/sysadmin.  I also liked it because it was written in php (I had also been considering Request Tracker, but it was written in Perl, which I don't know and don't want to learn).  Unfortunatly, most (if not all) tutorials out there are set up to use php5 and apache. That's just not going to fly with me. PHP5 has been discontinued since the beginning of the year and Apache should die a slow, miserable death (with all the stupid plugins you have to use). I wanted to use PHP7 and Nginx. So I figured it out and now I'll tell you how to do it as well.

There are a few gotchas at the time of this writing. Should you follow this tutuorial and these gotcha's have been corrected, please email me and I will update this post.

Awesome. Let's get started!

### Installing the goodies

I will be starting from a standard installation of Ubuntu 18.04 Server. If you're following along you should do this as well.

1. Enable UNIVERSE in apt:

    `sudo add-apt-repository universe` OR 

    edit the file by hand: edit `/etc/apt/sources.list` by appending `universe` on the end of all 3 entries

2. Update and Upgrade packages:

    `sudo apt-get update && sudo apt-get upgrade -y`

3. Install MySQL

    `sudo apt-get install mysql-server`

4. Install PHP:

    `sudo apt-get install php php-fpm php-pear php-imap php-apcu php-cgi php-common php-mbstring php-net-socket php-gd php-xml-util php-mysql php-gettext php-bcmath php-intl`

4. Install Nginx

    `sudo apt-get install nginx`

5. Install Miscellanious Packages:

    `sudo apt-get install unzip wget apt-utils`

### Setting up MySQL

Let's first set up MySQL to be a bit more secure. Run `mysql_secure_installation` from the terminal and run through the prompts.

Then we need to create the new osTicket database and user:

```
$ mysql -u root -p

mysql> CREATE DATABASE osticketdb;

mysql> GRANT ALL PRIVILEGES ON osticketdb.* TO osticketUser IDENTIFIED BY "password123";

mysql> exit
Bye
```

Bingo. Remember these creds for later.

### osTicket Installation

Now we can get started with the installation of osTicket.  We're going to be using the latest stable version, 1.10.4

On our server let's create a new folder for the installation and get the .zip

```
$ sudo mkdir -p /var/www/osticket

$ cd /var/www/osticket

[osticket]$ sudo wget https://github.com/osTicket/osTicket/releases/download/v1.10.4/osTicket-v1.10.4.zip

[osticket]$ unzip osTicket-v.1.10.4.zip

[osticket]$ ls -l
-rw-rw-r--  1 root    root    8693499 osTicket-v1.10.4.zip
drwxr-xr-x  2 root    root       4096 scripts/
drwxr-xr-x 13 root    root       4096 upload/
```

We need to make one change to the default configuration for osTicket to work. First we need to copy the sample `ost-sampleconfig.php`. The default settings should be ok for now.

```
[osticket]$ sudo cp upload/include/ost-sampleconfig.php upload/include/ost-config.php
```

Perfect. The next step is to set up our webserver, Nginx.

### Nginx with SSL Setup

Now that we have our database and app set up, we just need to set up Nginx so that we can serve the application to our users.  We do that by writing a new configuration file for nginx telling it how it should handle the HTTP requests it gets from users. Let's make a new configuration file, and then enable it so we don't forget.

```
$ sudo touch /etc/nginx/sites-available/osticket.conf

$ sudo ln -s /etc/nginx/sites-available/osticket.conf ../sites-enabled/
```

Now we need to edit the configuration file to handle all of the requests it will receive. Open the file with your favorite text editor and make it look like this:

```
server {
	listen 80;
	server_name www.osticket.corp.moxiesoft.com osticket.corp.moxiesoft.com;
	return 301 https://$host$request_uri;
}

server{
	listen 443 ssl;
	ssl_certificate		/etc/ssl/certs/osticket.pem;
	ssl_certificate_key	/etc/ssl/private/osticket.key;
	ssl_ciphers		    EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
	ssl_protocols		TLSv1.1 TLSv1.2;

	server_name www.osticket.mydomain.com osticket.mydomain.com;
	root /var/www/osticket/upload;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	index index.php;

	gzip on;
	gzip_min_length 1000;
	gzip_types text/plain application/x-javascript text/xml text/css application/xml;

	set $path_info "";

	# Deny access to all files in the include directory
	location ~ /include {
		deny all;
		return 403;
	}

	# Requests to /api/* need their PATH_INFO set, this does that
	if ($request_uri ~ "^/api(/[^\?]+)") {
		set $path_info $1;
	}

	# /api/*.* should be handled by /api/http.php if the requested file does not exist
	location ~ ^/api/(tickets|tasks)(.*)$ {
		try_files $uri $uri/ /api/http.php$is_args$args;
	}

	# /scp/ajax.php needs PATH_INFO too, possibly more files need it hence the .*\.php
		if ($request_uri ~ "^/scp/.*\.php(/[^\?]+)") {
		set $path_info $1;
	}

	# Make sure requests to /scp/ajax.php/some/path get handled by ajax.php
	location ~ ^/scp/ajax.php/(.*)$ {
		try_files $uri $uri/ /scp/ajax.php$is_args$args;
	}

	location ~ ^/ajax.php/(.*)$ {
		try_files $uri $uri/ /ajax.php$is_args$args;
	}

	location / {
		index     index.php;
		#try_files $uri $uri/ /index.php$is_args$args;
	}

	location ~ \.php$ {
		fastcgi_pass 	unix:/run/php/php7.2-fpm.sock;
		fastcgi_param  	SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		include        	fastcgi_params;
		include 	snippets/fastcgi-php.conf;
		#fastcgi_param  PATH_INFO    $path_info;
  }
}
```

Whew! That was a lot of typing! (I know you just copied and pasted it - so be sure to change the `server_name` in both server blocks)

You'll notice that I've already included the ssl_certificates and keys. I've showed you how to generate those files in a previous post, [Self Signed SSL Certificates](https://andyduss.com/blog/2019/01/24/self-signed-ssl-certs.html). Be sure to follow the link and add the certificates, otherwise it won't work.

We can verify that our configuration file works by running a syntax check on it:

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Then change ownership and restart the services

```
$ sudo chown -R www-data:www-data /var/www/osticket

$ sudo systemctl restart nginx php7.2-fpm
```

### Up and running (Kind of)

That should take care of the majority of the installation.  We can now go to our web browser and go to www.osticket.mydomain.com and see the installer page

![Installer Page](https://i.imgur.com/ARehb0f.png)

Click continue and fill out the basic information.  Under database settings use the database, username, and password we set up before.

![Database Settings](https://i.imgur.com/5bi01K3.png)

Click "Install" when you're ready. It'll spin and spin for a bit, but you should eventually get a "Congratulations!" page. Let's try out our new ticketing sytem!

Open your web browser and go to the osticket domain you've set up and you should see a beautiful interface! It must be working! Let's log in and see.

To log in as a technician/administrator, we need to go to the special `/scp/` login page. SCP stands for "Staff Control Panel"

Logging in and we see a beatiful ... fuck-a-duck. What is this? CSRF?

### What is CSFR?

CSFR stands for "Cross Site Forgery Request." This is a feature of web browsers that makes the web more secure by only allowing traffic from the same domain. If you want to learn more you can check out this great youtube video by Computerphile on CSFR

<iframe width="560" height="315" src="https://www.youtube.com/embed/vRBihr41JTo" frameborder="0" allow="encrypted-media;" allowfullscreen></iframe>

Now that we know what CSFR is, how do we provide the token for our app to work? Simple. Edit the `upload/include/class.ostsession.php` file.  Around line 190 you should see the lines:

```
catch (DoesNotExist $e) {
    $this->data = new SessionData(['session_id' => $id]);
}
```

Then, add one line to make the block look like this:

```
catch (DoesNotExist $e) {
    $this->data = new SessionData(['session_id' => $id]);
    $this->data->session_data = "";
}
```

Simple enough. Restart the services and log in again. Success!

### One last Gotcha

For the time being, everything appers to work correcntly. But once you go into the Admin Panel and try to view a tooltip, you'll notice that the bubbles are not populated with information. Instead, looking at the Network tab in your dev tools, you'll see an AJAX error.

![Naked Tooltip](https://i.imgur.com/CLLX7Sf.png)

This can be easily fixed by making a change to some php code on the server.

Open up `upload/include/class.osticket.php` in your text editor and find the `get_path_info()` function (It's right around line 365). Comment it out or make it look like the function below:

```
function get_path_info() {
    if(!empty($_SERVER['PATH_INFO']))
    return $_SERVER['PATH_INFO'];

    if(isset($_SERVER['ORIG_PATH_INFO']))
    return $_SERVER['ORIG_PATH_INFO'];

    $path_info = substr($_SERVER['REQUEST_URI'], strlen($_SERVER['SCRIPT_NAME']));
    if (strpos($path_info, '?') !== false) {
        $path_info = substr($path_info, 0, strpos($path_info, "?"));
    }
    if (isset($path_info[0]) && $path_info[0] == '/') {
        return $path_info;
    }

    return null;
}
```

Reload the services one last time and the tooltips should now work!

Lastly, while you're still on the server, go ahead and delete the `upload/setup/` directory.

With that, you should be all set up. You can now go into the Admin Panel and configure osTicket to your heart's desire.
