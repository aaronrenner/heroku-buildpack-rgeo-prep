# heroku-buildpack-rgeo-prep

This buildpack overwrites Heroku's default .bundle/config to set BUNDLE_BUILD__RGEO to Heroku's build directory.

## Setup

This solution closely follows this [blog article](https://devcenter.spacialdb.com/Heroku.html) by SpacialDB.


Create this .buildpacks file in the root of your project.

    https://github.com/peterkeen/heroku-buildpack-vendorbinaries.git
    https://github.com/aaronrenner/heroku-buildpack-rgeo-prep.git
    https://github.com/heroku/heroku-buildpack-ruby.git

Create this .vendor_urls file in the root of your project.

    https://s3.amazonaws.com/spacialdb/heroku/geos-3.4.2.tar.gz
    https://s3.amazonaws.com/spacialdb/heroku/proj-4.8.0.tar.gz

Add these files to git.

Now, set up your heroku configuration.

    heroku config:set BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git LD_LIBRARY_PATH=/app/lib

If you haven't already set up your heroku database for postgis, you need to run the following steps. You currently must have a production level database to enable postgis.

Since postgis uses different settings in the database.yml, you need to modify the DATABASE_URL variable. Run the following command and extract the nessecary components out of it:

    $ heroku config:get DATABASE_URL 
    postgres://<username>:<password>@<host>:<port>/<database>

With those variables, run the following command

    $ heroku config:set DATABASE_URL="postgis://<username>:<password>@<host>:<port>/<database>?postgis_extension=true&search_schema_path=public,postgis"

Enable PostGIS

    $ heroku pg:psql
    => CREATE EXTENSION postgis;

Deploy

    git push heroku master
    
Verify it worked

    heroku run bash
    ~> bundle exec rails c

    >> RGeo::Geos.supported?
    => true
    >> RGeo::CoordSys::Proj4.supported?
    => true

If both of these are true, you should be ready to go.

## Credits

This solution draws from many people's research including

* https://github.com/jcamenisch/heroku-buildpack-rgeo
* https://github.com/davekapp for help with troubleshooting the extconf.rb build process
* https://gist.github.com/perplexes/5357663
