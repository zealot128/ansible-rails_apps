Rails app - Rvm, PG, Nginx, Passenger
========

Deploying a Rails-app will always be a unique situation. Covering all aspects with one role is very hard. Therefore, this role provides an easy way, when deploying to:

* Ruby via RVM, user isolated.
* Capistrano as a deployment (shared/current directories). This role will create a database.yml and optionally a .env or secrets.yml, which capistrano can symlink into the repos.
* Optional: Nginx Passenger
* Optional: secrets.yml/.env variable files
* Optional: Postgres database with automatic dumping of last 7 days to /backup/db/APPNAME

This role makes it easy to deploy several (low traffic) apps on one host. **It is not intended for use of a distributed large app!** We use it, to deploy 7 similar apps to one slightly big machine.

Requirements
------------

Some requirements, that are covered by other roles:

### 1. You need to create an user by yourself (e.g. 'app'), e.g:

```yaml
    - role: mivok0.users
      users:
      - username: app
        uid: 1010
        name: 'deploy user'
        groups: ['sudo']
        ssh_key:
          - '{{pubkeys.john}}'
          - '{{pubkeys.jim}}'
          - '{{pubkeys.deployer}}'
```

### 2. If you'd like to use Postgres, then you need to install postgres before, e.g. using this role:

```yaml
    - role: Ansibles.postgresql
      when: false
      postgresql_locale: 'de_DE.UTF-8'
      postgresql_ext_install_contrib: yes
      postgresql_ext_install_dev_headers: true
      monit_protection: true
      postgresql_cluster_name: "main"
      postgresql_cluster_reset: false
      postgresql_encoding: 'UTF-8'
      postgresql_lc_messages: de_DE.UTF-8
      postgresql_lc_monetary: de_DE.UTF-8
      postgresql_lc_numeric: de_DE.UTF-8
      postgresql_lc_time: de_DE.UTF-8
      tags: [db,postgres]
      postgresql_users:
       - name: app
         pass: pass
         encrypted: no

```

### 3. If you'd like to use the provided nginx passenger configs, than you need passenger as well:

```yaml
    - role: abtris.nginx-passenger
```

Role Variables
--------------

There are two level of variables:

### 1. Global

```yaml
ruby_installation_method: rvm
rails_pg_collocation: 'en_US.UTF-8' # de_DE.UTF-8'

# curb/curl
rails_install_curb: true

# imagemagick and libmagick-wand-dev for Rmagick headers
rails_install_imagemagick: true

# only required, if you are compiling the assets on the target host and not
# using therubyracer
rails_install_nodejs_runtime: true

# put a /etc/nginx/sites-available/ file and active it
#
rails_nginx_site: true

# creates deployment key for app user (user of first defined app, app[0].user)
# will pause and show you the key, so you can copy/paste into your CI server
create_app_deployment_key: true

# wkhtmltopdf from source & xvfb
rails_install_wkhtmltopdf: true
# if you install wkhtmltopdf
wkhtmltopdf_url: http://downloads.sourceforge.net/project/wkhtmltopdf/0.12.2-dev/wkhtmltox-0.12.2-6a13a51_linux-precise-amd64.deb


```

### 2. an array of app-objects, like:



```yaml
  apps:
  - user: 'app'
    name: 'blog-app'
    domain: 'www.example.com example.com'
    path: '/var/www/blog'
    ruby: '/home/app/.rvm/wrappers/ruby-2.1.1/ruby'
    # path to local ssl cert/key, which will be used (can be left out)
    ssl_cert: ssl/cert.pem
    ssl_key: ssl/key.pem
    # created database, grants access and initiates backup cronjob
    pg_database: blog_production

    # logrotate weekly in $path/current/log/*
    logrotate: true

    # any kind of extra nginx stuff for the site conf
    nginx_extra: |
      rewrite ^/oldblog&(.*)$ /newblog?$1 permanent;

    # rails 4.1 secrets.yml
    secrets:
      secret_key_base: "asasdasd"
      twitter_token: "asdbasd"
      twitter_secret: "asdasdasdasd"

    # similar, but using dotenv
    env:
      SECRET_KEY_BASE: "asasdasd"
      TWITTER_TOKEN: "asdbasd"
      TWITTER_SECRET: "asdasdasdasd"
```






Example Playbook
-------------------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
  - hosts: servers
    # PARAMOUNT if you need to install rvm
    sudo: true
    roles:
    - role: rails_app
      ruby_version: '2.1.1'
      rails_install_wkhtmltopdf: true
      rails_install_imagemagick: true
      rails_install_nodejs_runtime: true
      apps:
      - user: 'app'
        name: 'blog-app'
        domain: 'www.example.com example.com'
        path: '/var/www/blog'
        ruby: '/home/app/.rvm/wrappers/ruby-2.1.1/ruby'
        ssl_cert: ssl/cert.pem
        ssl_key: ssl/key.pem
        pg_database: blog_production
        nginx_extra: |
          rewrite ^/oldblog&(.*)$ /newblog?$1 permanent;
        secrets:
          secret_key_base: "asasdasd"
          twitter_token: "asdbasd"
          twitter_secret: "asdasdasdasd"

```

Do not forget to add sudo if you want to use the rvm installation. Otherwise the rvm will be installed system wide, which might interfere.


License
-------

MIT


