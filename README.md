[![Build Status](https://travis-ci.org/slated/ansible-nginx-role.svg?branch=master)](https://travis-ci.org/slated/ansible-nginx-role)

Role Name
=========

Install nginx, copy ssl key/crt if `nginx_use_letsencrypt` not set,
disable default site and deploy a site config file.

Requirements
------------

Ssl cert and key, or certbot role in play and `nginx_use_letsencrypt: yes`.

Role Variables
--------------

Template for site conf file:

    nginx_site_template: wsgi-site-conf.j2
    
Set site domain, must be a single non-wildcard DNS domain:

    nginx_server_name:

Additional wildcard or regexp server names to match:

    nginix_extra_server_names:

Ssl setup (if using certbot); just nginx_use_letsencrypt is usually
enough, but you can override:

    nginx_use_letsencrypt: yes
    nginx_ssl_dest_dir: /etc/ssl
    nginx_strong_dh_group: yes  # Strongly recomended in production. See weakdh.org.
    nginx_certbot_live_dir: '{{ certbot_live_dir }}'
    nginx_certbot_cert_filename: fullchain.pem
    nginx_certbot_privkey_filename: privkey.pem

Ssl setup (if not using certbot); note: `ssl_key` should be kept secret!

    nginx_use_letsencrypt: no
    nginx_ssl_dest_dir: /etc/ssl
    nginx_strong_dh_group: yes  # Strongly recomended in production. See weakdh.org.
    nginx_ssl_crt: |
      -----BEGIN CERTIFICATE-----
      ...
    nginx_ssl_key: |
      -----BEGIN PRIVATE KEY-----
      ...
      
Upstream wsgi upstream name (`{{ nginx_wsgi_app }}-wsgi`) and servers
(for wsgi-site-conf template). 

    nginx_wsgi_app: webapp
    nginx_wsgi_port: 10000
    nginx_wsgi_servers:
      - localhost
      - server

Static file service, if Nginx is serving Django collectstatic files directly:

    nginx_serve_static: no
    nginx_static_dir:
    nginx_serve_media: no
    nginx_media_dir:

Log file dir

    nginx_log_dir: /var/log/nginx

Htpass for restricted content; only deployed if nginx_restricted_auth set

    nginx_restricted_file_path: /etc/nginx/.htpasswd
    nginx_restricted_auth:

Set Nginx load balancing algorithm

    nginx_load_balancing_algorithm: ip_hash

Dependencies
------------

The _certbot_ role if `nginx_use_letsencrypt: yes`

Example Playbook
----------------

    - hosts: servers
      - import role:
        name: certbot
        vars:
          certbot_hostname: my.domain.com
      - import_role:
          name: nginx
        vars:
          nginx_server_name: my.domain.com
          nginx_use_letsencrypt: yes
          nginx_wsgi_servers:
            - { host: app1, port: 10000 }
            - { host: app2, port: 10000 }
