<h1>Wordpress with Docker Compose on GCP</h1>
<h3> Introduction </h3>
Here I am going to install Docker and want to run wordpress using docker-compose. For wordpress as you know it needs apache/nginx with php, mysql/mariadb and wordpress. I am using a simple Google Compute Engine where I have used different container for nginx, mariadb and wordpress. I will show you how those are composed.

<h3> Getting Started with Docker and Docker Compose  </h3> 

* First we need one Google Compute Engine.
* Need to install docker, I have installed docker CE  following this [documentation](https://docs.docker.com/install/linux/docker-ce/centos/)
* Once docker is installed and started, need to install docker-compose. Installing docker-compose is very simple. You need to run this command 
``curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose``. Once that is done give execute permission to /usr/local/bin/docker-compose ``chmod +x /usr/local/bin/docker-compose``
* Test the Installation

``docker-compose --version``

``docker-compose version 1.21.0, build 5920eb0``

So docker-compose is installed on our box. 

<h3>How docker-compose works </h3>

* We will create on directory for our project

 ``mkdir composetest``

``cd composetest``

``touch docker-compose.yml``

``vi docker-compose.yml``

Paste the below configuration 

   
    nginx: 
    image: nginx:latest
    ports:
        - '80:80'
    volumes:
        - ./nginx:/etc/nginx/conf.d   
        - ./logs/nginx:/var/log/nginx        
        - ./wordpress:/var/www/html        
    links:
        - wordpress
    restart: always
    mysql:
    image: mariadb
    ports:
        - '3306:3306'
    volumes:
        - ./db-data:/var/lib/mysql
    environment:
        - MYSQL_ROOT_PASSWORD=password
    restart: always 
    wordpress:
    image: wordpress:4.7.1-php7.0-fpm
    ports:
        - '9000:9000'
    volumes:
        - ./wordpress:/var/www/html
    environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=password
    links:
        - mysql
    restart: always

* Create directory nginx, db-data, logs/nignx and wordpress
* nginx for ngin configuration
* db-data for database
* wordpress for wordpress configuration
* open nginx/wordpress.conf
paste the below 

`` 
    
     server {
          listen 80;
          server_name wp_server; 
    root /var/www/html;
    index index.php;
 
    access_log /var/log/nginx/wp_server-access.log;
    error_log /var/log/nginx/wp_server-error.log;
 
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
 
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
      }
    }
``

* So the configuration is done. Run _docker-compose up -d_

Browse _http://your-server-ip_

 ![](https://res.cloudinary.com/cloudinsnap/image/upload/v1525106767/wpconfig_new.png)
* Configure wordpress