---
title: "Hosting a WordPress instance in Docker over HTTPS"
tags: [Caddy, Docker, Let's Encrypt, WordPress]
---

This is how I created a [WordPress](https://wordpress.org/) blog that is hosted in a [Docker](https://www.docker.com/) container. The instance is configured behind a [Caddy](https://caddyserver.com/) web server, which is used to provide automatic HTTPS via [Let's Encrypt](https://letsencrypt.org/).

1. Create a network to host the WordPress related images

    ```bash
    docker network create blog-net
    ```

1. Create a volume and container to host the database for the WordPress instance

    ```bash
    docker volume create blog-mysql-data
    docker run --name blog-mysql -d --net blog-net -v blog-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=rootPassword -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpressPassword -e MYSQL_DATABASE=wordpress mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    ```

1. Create a volume and container to host the WordPress instance

    ```bash
    docker volume create blog-wordpress-html
    docker run --name blog-wordpress --net blog-net -d -v blog-wordpress-html:/var/www/html -e WORDPRESS_DB_HOST=blog-mysql -e WORDPRESS_DB_USER=wordpress -e WORDPRESS_DB_PASSWORD=wordpressPassword -e WORDPRESS_DB_NAME=wordpress wordpress
    ```

1. Create a network to host the reverse proxy and connect the WordPress instance to it

    ```bash
    docker network create revproxy-net
    docker network connect revproxy-net blog-wordpress
    ```

1. Create a volume and container for the reverse proxy

    ```bash
    docker volume create revproxy-caddy-etc
    docker run --name revproxy-caddy --net revproxy-net -d -v revproxy-caddy-etc:/etc -p 80:80 -p 443:443 abiosoft/caddy
    ```

1. Open vi within the container to configure Caddy

    ```bash
    docker exec -it revproxy-caddy /bin/sh
    vi /etc/Caddyfile
    ```

1. Update the contents of the Caddyfile to reverse proxy the WordPress instance

    ```text
    example.com {
     # Redirect rule to redirect the root of example.com to example.com/blog.
     redir 301 {
      if {path} is /
       / /blog
     }

     # Proxy rule to proxy /blog to port 80 on the WordPress container.
     # The without line make /blog map to the route of the WordPress container.
     proxy /blog blog-wordpress:80 {
      transparent
      without /blog
     }

     # Redirect logs and errors to stdout for docker logs.
     log stdout
     errors stdout
    }
    ```

1. Leave and restart the Caddy container

    ```bash
    exit
    docker restart revproxy-caddy
    ```

1. Add vi to the WordPress container so the config can be updated

    ```bash
    docker exec -it blog-wordpress /bin/bash
    apt-get update && apt-get -y install vim
    ```

1. Open wp-config.php using vi and add rules to configure the WordPress instance at the /blog sub directory (see indented lines)

    ```php
    <?php
    /**
     * The base configuration for WordPress
     *
     * The wp-config.php creation script uses this file during the
     * installation. You don't have to use the web site, you can
     * copy this file to "wp-config.php" and fill in the values.
     *
     * This file contains the following configurations:
     *
     * * MySQL settings
     * * Secret keys
     * * Database table prefix
     * * ABSPATH
     *
     * @link https://codex.wordpress.org/Editing_wp-config.php
     *
     * @package WordPress
     */

        /* http://stackoverflow.com/questions/34090577/wordpress-nginx-proxy-and-subdirectory-wp-login-php-redirects-to-domain */
        $_SERVER['REQUEST_URI'] = str_replace("/wp-admin/", "/blog/wp-admin/", $_SERVER['REQUEST_URI']);
        define( 'WP_SITEURL', '/blog' );
        define( 'WP_HOME', '/blog' );

    // ** MySQL settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define('DB_NAME', 'wordpress');
    ```

1. Navigate to example.com/blog/wp-admin/install.php and install WordPress as normal.
