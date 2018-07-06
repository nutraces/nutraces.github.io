---
layout: post
title: "How to setup wordpress using docker"
---


* Install docker ce or ee
* create docker-compose.yml as below

```
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: rootadmin
     networks:
       - back
     ports:
       - 23306:3306

   wordpress:
     depends_on:
       - db
     image: wordpress:4.8
     restart: always
     volumes:
       - ./html:/var/www/html
     environment:
       WORDPRESS_DB_HOST: "db:3306"
       WORDPRESS_DB_PASSWORD: "quehivemysqladmin"
       WORDPRESS_DB_USER: "userquehive"
       WORDPRESS_DB_NAME: "wpquehive"
       SERVER_NAME : localhost
     ports:
       - 80:80
     networks:
       - back

networks:
   back:

volumes:
  db_data: 
  html:
 ```
 
 ```

 * volumes - html : if you have existing wordpress , copy to here otherwise it will generate one during installation
 * volumes - db_data : it stores mysql data in the host machine , this is to avoid loss of data during container restart/redeploy
 
 ```
 
* Execute `docker-compose up -d` 

* Verify using `docker ps -a` and `docker logs <container id>`

* To import your existing wordpress database , you can provide remote access to for mysql user as follows

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION; 
SET PASSWORD FOR root@'%' = PASSWORD('password');
FLUSH PRIVILEGES;

GRANT ALL PRIVILEGES ON *.* TO 'otheruser'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION; 
SET PASSWORD FOR otheruser@'%' = PASSWORD('password');
FLUSH PRIVILEGES;

```

* Import the existing database

* Update migration script for localhost docker deployment as

```
UPDATE wp_options SET option_value = replace(option_value, 'http://example.com', 'http://127.0.0.1') WHERE option_name = 'home' OR option_name = 'siteurl';
UPDATE wp_posts SET guid = replace(guid, 'http://example.com','http://127.0.0.1');
UPDATE wp_posts SET post_content = replace(post_content, 'http://example.com', 'http://127.0.0.1');
UPDATE wp_postmeta SET meta_value = replace(meta_value,'http://example.com','http://127.0.0.1');
commit;
```

* Expose 80 port in the host machine to access the application

* Verify the application http://example.com or http://localhost

* Also, If you use nginx you can expose the wordpress ports to different and configure the same in nginx

 
