
.. meta::
   :description: A sample NGINX configuration for Matomo.

Matomo
=======

Recipe
------

You can see the complete information :github:`here <matomo-org/matomo-nginx>`.

This configuration file was initially provided by Seph.

.. code-block:: nginx

    server {
       ## This is to avoid the spurious if for sub-domain name rewriting.
       listen [::]:80;
       server_name www.stats.example.com;
       rewrite ^ $scheme://stats.example.com$request_uri? permanent;
    }

    server {
        listen [::]:80;
        limit_conn arbeit 10;
        server_name stats.example.com;

        # Parameterization using hostname of access and log filenames.
        access_log  /var/log/nginx/stats.example.com_access.log;
        error_log   /var/log/nginx/stats.example.com_error.log;

        # Disable all methods besides HEAD, GET and POST.
        if ($request_method !~ ^(GET|HEAD|POST)$ ) {
            return 444;
        }

        root  /var/www/sites/stats.example.com/;
        index  index.php index.html;
        
        # Support for favicon. Return a 204 (No Content) if the favicon
        # doesn't exist.
        location = /favicon.ico {
                 try_files /favicon.ico =204;
        }

        # Try all locations and relay to index.php as a fallback.
        location / {
                 try_files $uri /index.php;
        }

        # Relay all index.php requests to fastcgi.
        location ~* ^/(?:index|piwik)\.php$ {
                fastcgi_pass unix:/tmp/php-cgi/php-cgi.socket;
        }
        
        # Return a 404 for protected directories
        location ~ /(?:config|tmp|vendor)/ {
                  return 404;
        }

        # Any other attempt to access PHP files returns a 404.
        location ~* ^.*\.php$ {
                  return 404;
        }

        # Return a 404 for files and directories starting with a period. This includes directories used by version control systems
        location ~ /\. {
                 return 404;
        }
        
        # Return a 404 for package manager config files
        location ~ (?:composer.json|composer.lock|bower.json)$ {
                 return 404;
        }

        # Return a 404 for all text files.
        location ~* (?:README|LICENSE|LEGALNOTICE|\.txt|\.md)$ {
                 return 404;
        }

        # # The 404 is signaled through a static page.
        # error_page  404  /404.html;

        # ## All server error pages go to 50x.html at the document root.
        # error_page 500 502 503 504  /50x.html;
        # location = /50x.html {
        # 	root   /var/www/nginx-default;
        # }
    } # server

