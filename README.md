## How to setup WordPress on Heroku with the Heroku Buildpack for PHP

This will set up a fresh WordPress install on Heroku with the [Heroku Buildpack for PHP](https://github.com/heroku/heroku-buildpack-php).

* `nginx` - Nginx for serving web content.
* `php` - PHP-FPM for process management.
* `wordpress` - Downloaded from the Github WordPress Repo.
* `MySQL` - ClearDB for the MySQL backend.
* `Sendgrid` - Sendgrid for the email backend.
* `MemCachier` - MemCachier for the memcached backend.
* `New Relic`- Monitoring

The following plugins will enable you to use Wordpress as a headless (API-based) CMS:

* S3 Uploads - Save all media uploads to a CDN on S3
* WP REST API v2 - Enable a JSON API
* Custom Post Type UI - Create custom post types through the WP admin

## Getting started

Use the Deploy to Heroku button, or use the old fashioned way described below.

<a href="https://heroku.com/deploy?template=https://github.com/ericecchi/headless-wordpress-heroku/tree/master">
  <img src="https://www.herokucdn.com/deploy/button.png" alt="Deploy">
</a>

Clone this repository into a new directory.

Create your Heroku app.

	heroku apps:create application-name --stack cedar --buildpack https://github.com/heroku/heroku-buildpack-php --region eu

`--region eu` is for deploying your app in the European region.

or on to add this buildpack to an existing app, run

	heroku config:set BUILDPACK_URL=https://github.com/heroku/heroku-buildpack-php


Before you push to Heroku make sure to add the following add-ons.

	heroku addons:add cleardb
	heroku addons:add sendgrid
	heroku addons:add memcachier
	heroku addons:add papertrail
	heroku addons:add newrelic


Define your AWS keys for the S3 Uploads plugin.

	heroku config:set S3_UPLOADS_BUCKET="bucket-name"
    heroku config:set S3_UPLOADS_KEY="abc123"
    heroku config:set S3_UPLOADS_SECRET="abc123"
    heroku config:set S3_UPLOADS_REGION="us-standard"


Some default configurations. WP_CACHE=true will enable Batcache with the Memcachier addon.

	heroku config:set DISABLE_WP_CRON=true
	heroku config:set WP_CACHE=true

Deploy your WordPress site to Heroku.

	git add .
	git commit -am "Initial commit"
	git push heroku master


## Overview
```
└── public                 # Heroku webroot
    ├── content            # The wp-content directory. Renamed to content to avoid confusion with wp-content - and it looks prettier
    │   ├── plugins        # Plugins
    │   ├── mu-plugins     # Required plugins
    │   └── themes         # Your custom themes
    │      
    └── wp                 # Where the actual WordPress install will be installed by Composer
```

## Upgrade WordPress

Update the version number for the WordPress package in composer.json, then run `composer update` and commit the changes in composer.json and composer.lock. Do not upgrade WordPress from the admin-interface as it will not survive a restart or dyno change.


## Setup local development

Make sure you have [Composer](https://getcomposer.org/) installed first, then run

	composer install

Create a local .env file.

	CLEARDB_DATABASE_URL=mysql://root:123abc@127.0.0.1/my_wordpress_heroku_database_name
	
or install the heroku config plugin from https://github.com/ddollar/heroku-config and pull your environment variables from Heroku.
The second option is to use the provided local-sample-config.php and rename it local-config.php. Update it with your local MySQL credentials, and you're good to go.

> NOTE: If you don't have a command-line mysql accessible and working, Mac/Homebrew users can `brew install mysql` and then follow the directions to have launchd start mysql at login. I believe the default username is root and the default password is blank.


## Known Issues

If you try to develop locally without syncing your external MemCachier envvars you might see a 500 error or a *You do not have sufficient permissions to access this page.* - message. Workaround is to simply remove object-cache.php and advanced-cache.php from the content dir while doing local dev. In a future release I'll try to have these files added on deploy with Composer.


## Sources

This would not have been possible without the work and resources provided by the following people:

* http://mattstauffer.co/blog/laravel-on-heroku-using-a-buildpack-locally-to-mimic-your-heroku-environment-nginx
* https://github.com/mchung/wordpress-on-heroku
