# Procedures for Hosting Applications on the Server

## Preparing a new server

In order to prepare a new server to setup a CoachPodium instance or any Laravel app, you can follow these steps. But before we can, the server needs to have the following services up and running:

### Required services
* Nginx Server
* PHP
* Nodejs
* MySQL
* Git
* Composer
* NPM
* Certbot

***

### SSHing the server

`$ ssh user_name@server_ip_address` enter SSH password when prompted

Updating the installed packages or services

* `$ sudo apt update` updating the packages and services meta information
* `$ sudo apt upgrade -y` upgrading packages and services without prompting

***

### Installing the Nginx

* `$ sudo apt update`
* `$ sudo apt install nginx`
* `$ sudo service nginx start`

***

### Installing the MySQL

* `$ sudo apt update`
* `$ sudo apt install mysql-server`
* `$ sudo service mysql start`

#### Using MySQL and setup a default password for the root user
* `$ sudo mysql`
* `mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'ppp12345';`
* `mysql> exit`

#### Securing the MySQL

* `$ sudo mysql_secure_installation`

This will take you through a series of prompts where you can make some changes to your MySQL installation’s security options. The first prompt will ask whether you’d like to set up the Validate Password Plugin, which can be used to test the password strength of new MySQL users before deeming them valid.

If you elect to set up the Validate Password Plugin, any MySQL user you create that authenticates with a password will be required to have a password that satisfies the policy you select. The strongest policy level — which you can select by entering 2 — will require passwords to be at least eight characters long and include a mix of uppercase, lowercase, numeric, and special characters:

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: Y

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
```

```
Please set the password for root here.


New password: ppp12345

Re-enter new password: ppp12345
```

```
Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y
```

***If you are unable to connect to MySQL using the root password, change the password and the connection privilege. Follow these steps:***

* `$ sudo mysql`
* `mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'ppp12345';`


#### Create a dedicated MySQL user

* `$ sudo mysql -uroot -p`
* `mysql> CREATE USER 'user_name'@'localhost' IDENTIFIED BY 'qqq12345';`
* `mysql> GRANT CREATE, ALTER, DROP, INSERT, UPDATE, INDEX, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'user_name'@'localhost' WITH GRANT OPTION;`
* `mysql> FLUSH PRIVILEGES;`
* `mysql> exit`
* `$ mysql -uuser_name -p`

***

### Installing the latest version of Git

* `$ sudo add-apt-repository ppa:git-core/ppa` press enter when prompted
* `$ sudo apt update`
* `$ sudo apt install git`

***

### Installing the PHP

* `$ sudo add-apt-repository ppa:ondrej/php` press enter when prompted
* `$ sudo apt update`
* `$ sudo apt install php8.2 php8.2-cli`
* `$ sudo apt install php8.2-{bz2,curl,mbstring,intl,fpm,mysql,xml,pdo}` installing PHP extensions
* `$ sudo apt install php8.2-imagick` installing the required extension for CoachPodium helper service
* `$ sudo service php8.2-fpm start`
* `$ php -v`

Installing the another version of PHP

* No need to add the PPA because it has already been included.
* `$ sudo apt update`
* `$ sudo apt install php8.0 php8.0-cli`

#### Changing the PHP version
* `$ sudo update-alternatives --config php`
* Select a version from numbered prompt

***

### Installing the composer

* `$ cd /var/www`
* `$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`
* `$ php -r "if (hash_file('sha384', 'composer-setup.php') === 'e21205b207c3ff031906575712edab6f13eb0b361f2085f1f1237b7126d785e826a450292b6cfd1d64d92e6563bbde02') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"`
* `$ php composer-setup.php`
* `$ sudo mv composer.phar /usr/local/bin/composer` to access composer anywhere from the terminal
* `$ php -r "unlink('composer-setup.php');"`
* `$ composer` verification

***

### Installing NVM and Nedejs

* `$ curl --version` check if the curl has been installed
* `$ sudo apt update && sudo apt install curl` install curl if not found
* `$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash`
* `$ source ~/.bashrc`
* `$ nvm -v` verify
* `$ nvm install 18.*.*`
* `$ nvm use node` use the latest version of node
* `$ nvm alias default 18.*.*` to access this version of node from anywhere

***

### Installing Cerbot

* `$ sudo snap instal certbot --classic` using the snapd package manager
* `$ sudo ln -s /snap/bin/certbot /usr/bin/certbot` to access cerbot from anywhere

***

## Creating an instance of the application running on the server

Creating an application name in `/var/www`

* `$ cd /var/www`
* `$ mkdir app.coachpodium.com`

> Publishing the VPS repository code to specified application directory

* `$ cd /var`
* `$ mkdir repo` creating a folder to track all VPS repo
* `$ cd repo`
* `$ git init --bare app.coachpodium.git`
* `$ git branch -M main` creating main branch as a main branch
* `$ cd app.coachpodium.git/hooks`
* `$ touch post-receive`
* `$ sudo nano post-receive`

```
#!/bin/sh

git --work-tree=/application_directory --git-dir=/bare_repo_directory checkout -f
```

in case of ours
```
#!/bin/sh

git --work-tree=/var/www/app.coachpodium.com --git-dir=/var/repo/app.coachpodium.git checkout -f
```
* `$ chmod +x post-receive` creating it as a bash executable

Pushing to remote repo

* Inside your local project repo
* `$ git remote add origin_name ssh://ssh_user_name@server_ip_address/bare_repo_address`  in our case, look for the next bullet point
* `$ git remote add live ssh://abhishekk@198.145.67.34/var/repo/app.coachpodium.git`
* `$ git push origin_name local_branch:main`

> #### Note
>
> *When pulling changes from github/gitlab, specify origin_name; otherwise, git tries to pull changes from the VPS server too.*

Installing Composer and NPM in serve (One time setup)

* SSH to your server
* `$ cd /var/www/app.coachpodium.com`
* `$ cp .env.example .env`
* `$ php artisan key:generate`
* `$ composer install`
* `$ npm install`
* `$ php artisan storage:link`
* Fill `.env` details
* `$ npm run prod` producing assets
* `$ php artisan migrate --seed`

> #### Note
>
> If new package(s) are included in the local repository, then we need to install the package(s) using the respective package manager on the server too.

Setting up permission on the application directory

* [SSHing the server](#sshing-the-server)
* `$ cd /var/www/app.coachpodium.com`
* `$ sudo chown -R www-data:www-data /var/www/app.coachpodium.com/public`
* `$ sudo chmod 755 /var/www`
* `$ sudo chmod -R 755 /var/www/app.coachpodium.com/bootstrap/cache`
* `$ sudo chmod -R 755 /var/www/app.coachpodium.com/storage`

Configuring Nginx


* `$ cd /var/nginx/sites-available`
* `$ touch app.coachpodium.com`
* `$ sudo nano app.coachpodium.com`
```
server {
	server_name app.coachpodium.com; # domain name
	set $base /var/www/app.coachpodium.com; # app directory
	root $base/public;

    # disable broadcasting the nginx version
    server_tokens off;

	# index.php
	index index.php index.html index.htm;

	# index.php fallback
	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

	# handle .php
	location ~ \.php$ {
		include snippets/fastcgi-php.conf;
        # responsible php version to handle the request
		fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        # or need to use another php, install and use
		# fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
	}

	location ~ /\.ht {
		deny all;
	}
}
```
* Check whether the nginx configuration is correct using
* `$ nginx -t`
* If correct then enable site using
* `$ ln -s /etc/nginx/sites-available/app.coachpodium.com /etc/nginx/sites-enabled/`
* Reload nginx to listen for a request using `$ sudo service nginx reload`

Generating SSL certificate using Certbot

* `$ certbot --nginx`
* Select site from the prompt


#### Build the NPM script automatically after it is pushed to the server

I order to use this feature we need to edit the `post-receive` hook

* `$ cd /var/repo/app.coachpodium.git/hooks`
* `$ sudo nano post-receive` and add the following line
```
export PATH=$PATH:/root/.nvm/versions/node/v18.18.0/bin/
cd /var/www/coach-podium.codewingsaas.com
echo "Installing packages"
echo "Building packages started ..." && npm run server:build && echo "Building packages completed"
```

after that the hook looks like this
```
#!/bin/sh

git --work-tree=/var/www/app.coachpodium.com --git-dir=/var/repo/app.coachpodium.git checkout -f

export PATH=$PATH:/root/.nvm/versions/node/v18.18.0/bin/
cd /var/www/app.coachpodium.com
echo "Installing packages"
echo "Building packages started ..." && npm run prod && echo "Building packages completed"
```

#### Adding an extra security layer on top of the Nginx configuration

Add these to your Nginx configuration in `/etc/nginx/sites-available/site_configuration`. If adding these policies breaks the working application or website, try to modify them and verify.

> Only allowing HTTPS connections or HTTP Strict Transport Security

```
add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains; preload';
```

> Improved header for Preventing XSS attack or Content Security Policy

```
add_header Content-Security-Policy "default-src 'self'; font-src *;img-src * data:; script-src *; style-src *";
```

> XSS protection

```
add_header X-XSS-Protection "1; mode=block";
```

> Disabling to iframe application/website or X-Frame option

```
add_header X-Frame-Options "SAMEORIGIN";
```

> Prevent to sniff the assets or Content Type Option

```
add_header X-Content-Type-Options nosniff;
```

> Protecting from malicious referrer or Referrer Policy

```
add_header Referrer-Policy "strict-origin";
```

> Instructing the browser to allow specified API or Permission Policy

```
add_header Permissions-Policy "geolocation=(),midi=(),sync-xhr=(),microphone=(),camera=(),magnetometer=(),gyroscope=(),fullscreen=(self),payment=(),interest-cohort=()";
```
