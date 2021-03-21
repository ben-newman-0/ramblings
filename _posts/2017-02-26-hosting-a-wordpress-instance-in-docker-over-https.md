---
title: "Hosting a WordPress instance in Docker over HTTPS"
tags: [Caddy, Docker, Let's Encrypt, WordPress]
---

This is how I created a <a href="https://wordpress.org/">WordPress</a> blog that is hosted in <a href="https://www.docker.com/">Docker</a>. The instance is configured behind a <a href="https://caddyserver.com/">Caddy</a> web server, which is used to provide automatic HTTPS via <a href="https://letsencrypt.org/">Let's Encrypt</a>.
<ol>
 	<li>Create a network to host the WordPress related images
<pre>docker network create blog-net</pre>
</li>
 	<li>Create a volume and container to host the database for the WordPress instance
<pre>docker volume create blog-mysql-data
docker run --name blog-mysql -d --net blog-net -v blog-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=rootPassword -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=wordpressPassword -e MYSQL_DATABASE=wordpress mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci</pre>
</li>
 	<li>Create a volume and container to host the WordPress instance
<pre>docker volume create blog-wordpress-html
docker run --name blog-wordpress --net blog-net -d -v blog-wordpress-html:/var/www/html -e WORDPRESS_DB_HOST=blog-mysql -e WORDPRESS_DB_USER=wordpress -e WORDPRESS_DB_PASSWORD=wordpressPassword -e WORDPRESS_DB_NAME=wordpress wordpress</pre>
</li>
 	<li>Create a network to host the reverse proxy and connect the WordPress instance to it
<pre>docker network create revproxy-net
docker network connect revproxy-net blog-wordpress</pre>
</li>
 	<li>Create a volume and container for the reverse proxy
<pre>docker volume create revproxy-caddy-etc
docker run --name revproxy-caddy --net revproxy-net -d -v revproxy-caddy-etc:/etc -p 80:80 -p 443:443 abiosoft/caddy</pre>
</li>
 	<li>Open vi within the container to configure Caddy
<pre>docker exec -it revproxy-caddy /bin/sh
vi /etc/Caddyfile</pre>
</li>
 	<li>Update the contents of the Caddyfile to reverse proxy the WordPress instance
<pre>example.com {
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
}</pre>
</li>
 	<li>Leave and restart the Caddy container
<pre>exit
docker restart revproxy-caddy</pre>
</li>
 	<li>Add vi to the WordPress container so the config can be updated
<pre>docker exec -it blog-wordpress /bin/bash
apt-get update &amp;&amp; apt-get -y install vim</pre>
</li>
 	<li>Open wp-config.php using vi and add rules to configure the WordPress instance at the /blog sub directory (see indented lines)
<pre>&lt;?php
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
...</pre>
</li>
 	<li>Navigate to example.com/blog/wp-admin/install.php and install WordPress as normal.</li>
</ol>
