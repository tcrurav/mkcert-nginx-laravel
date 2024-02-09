# Laravel 10 HTTPs using nginx, mkcert in WSL2 Ubuntu 22.04

This project is just a simple Laravel 10 project using mkcert to create a self signed certificate. The project is hosted in a WSL2 Ubuntu 22.04 machine.

The final result will make possible to access a web site in your own PC so that you get the Chrome browser message that you are browsing a secure site:

![Before log in](/screenshots/screenshot-01.png)

## Getting Started

Read the links in the acknowlegements section bellow to get an idea about mkcert.

You can get an idea of how https certificates work with this tutorial:
[Secure Web Site using certificates](/docs/DPL-2012-2013-doc10-UT-2-https.pdf)

## Prerequisites

You need a working environment with:
* [Ubuntu 22.04 in WSL2](https://apps.microsoft.com/detail/9PN20MSR04DW?hl=en-US&gl=US) - The OS this project has been installed on.
* [Git](https://git-scm.com) - You can install it from https://git-scm.com/downloads.
* [Laravel](https://laravel.com/) - A PHP framework.

## General Installation instructions

First you need an environment with linux. In my case:
 - Ubuntu 22.04 on WSL2 on Windows 11.

Then clone this project:
```
git clone https://github.com/tcrurav/mkcert-nginx-laravel.git
```

You may want to check that the project is working with laravel dev server:
```
cp .env.example .env
composer install
php artisan serve
```

Now that you are sure the project is working with laravel artisan server it's time to start with the real work. 

Let's install mkcert on Ubuntu 22.04 WSL2: (read mkcert project for more infos/options) https://github.com/FiloSottile/mkcert).
```
sudo apt install libnss3-tools
sudo apt install mkcert
```

Now create the Certificate Authority (CA):

![Creating CA](/screenshots/screenshot-02.png)

Then create the certificate files:
```
$ mkcert localhost 127.0.0.1 ::1

Created a new certificate valid for the following names 📜
 - "localhost"
 - "127.0.0.1"
 - "::1"
```

Then move the created files to the following directories:
```
mv localhost+2.pem /etc/ssl/certs/localhost+2.pem
mv localhost+2-key.pem /etc/ssl/private/localhost+2-key.pem
```

Now you have to install nginx on linux. I have followed this tutorial: https://www.rosehosting.com/blog/configure-php-fpm-with-nginx-on-ubuntu-22-04/

The main steps are:

1. Update and upgrade your ubuntu 22.04.
```
sudo apt update -y && sudo apt upgrade -y
```

2. Install PHP-FPM
```
sudo apt install software-properties-common ca-certificates lsb-release apt-transport-https -y LC_ALL=C.UTF-8 add-apt-repository ppa:ondrej/php

sudo apt update
sudo apt install php8.1 php8.1-fpm php8.1-mysql php8.1-mbstring php8.1-xml php8.1-curl
```

3. Edit your default nginx site. (Maybe first make a backup of the file)
```
sudo vim /etc/nginx/site-available/default
```

4. This file should content this:
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # SSL configuration
    #
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    include snippets/my-own-mkcert.conf;

    access_log /var/log/nginx/yourdomain.com-access.log;
    error_log  /var/log/nginx/yourdomain.com-error.log error;

    root /mnt/c/MisCosas/bicycles/public;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html index.php;

    server_name _;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
}
```

5. create a file /etc/nginx/snippets/my-own-mkcert.conf:
```
ssl_certificate /etc/ssl/certs/localhost+2.pem;
ssl_certificate_key /etc/ssl/private/localhost+2-key.pem;
```

6. Start php and nginx. In my case I'm old styled and don't use the new systemctl.
```
sudo service php8.1-fpm start
sudo service nginx start
```

Your nginx should be running.

Now Finally Install chrome on your Ubuntu 22.04 on WSL2 to test your HTTPs server. For that I have followed the following tutorial: https://scottspence.com/posts/use-chrome-in-ubuntu-wsl. The main steps are:
```
sudo apt update && sudo apt -y upgrade && sudo apt -y autoremove
sudo wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt -y install ./google-chrome-stable_current_amd64.deb
```

Now you can run Chrome from the ubuntu 22.04 command line:
```
google-chrome
```

And finally you can test your laravel 10 project with the url https://localhost
![Before log in](/screenshots/screenshot-01.png)

Enjoy!!!

## Built With

* [Visual Studio Code](https://code.visualstudio.com/) - The Editor used in this project
* [mkcert](https://github.com/FiloSottile/mkcert) - mkcert is a simple tool for making locally-trusted development certificates. It requires no configuration.

## Acknowledgments

* Thank you to my student <strong>Kevin</strong> (https://github.com/KRodVal) who has told me about mkcert and given the first instructions to start with it.
* https://www.rosehosting.com/blog/configure-php-fpm-with-nginx-on-ubuntu-22-04/. Configure nginx on ubuntu 22.04 with PHP support.
* https://scottspence.com/posts/use-chrome-in-ubuntu-wsl. Install chrome on Ubuntu on WSL2.
* https://github.com/FiloSottile/mkcert. mkcert is a simple tool for making locally-trusted development certificates. It requires no configuration.
* https://gist.github.com/PurpleBooth/109311bb0361f32d87a2. A very complete template for README.md files.
