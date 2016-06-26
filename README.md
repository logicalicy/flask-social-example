# Flask-Social Example Application

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

This application illustrates how to use [Flask-Social](http://packages.python.org/Flask-Social). It is developed for deployment at [Heroku](http://www.heroku.com).

This application is currently deployed at [http://flask-social-example.herokuapp.com](http://flask-social-example.herokuapp.com)

## Deploying to a Digital Ocean Droplet with Dokku

Choose an app name (or keep `flask-social-example`). This app name will be referred to as `<MY_APP_NAME>` below.

### Prepare Droplet

#### Install Droplet

Create droplet with Ubuntu Dokku 0.6.2 on 14.04

#### Add DNS Records

1. Add a domain (e.g. `<MY_APP_NAME>.example.com`)
2. Add CNAME records:

    ```
    www --> @
    * ---> @
    ```

### Configure Droplet

```
open <MY_DROPLET_IP_ADDRESS> # Complete Dokku setup, ensure hostname is set to <MY_APP_NAME>.example.com
ssh root@<MY_DROPLET_IP_ADDRESS>
dokku plugin:install https://github.com/dokku/dokku-postgres.git # Install dokku PostgreSQL plugin
dokku postgres:create <MY_APP_NAME>
Waiting for container to be ready
      Creating container database
      Securing connection to database
=====> Postgres container created: <MY_APP_NAME>
      DSN: postgres://postgres:XXX@dokku-postgres-<MY_APP_NAME>:5432/<MY_APP_NAME>
exit
```

Keep note of the DSN. This will be important to configure `app/config/app.yml`.

### Set Up Local Development Environment

```
mkdir ~/git && cd ~/git && git clone https://github.com/mattupstate/flask-social-example.git <MY_APP_NAME>
cd ~/git/<MY_APP_NAME>
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
echo "venv" >> .gitignore
brew upgrade postgresql
rm -rf /usr/local/var/postgres && initdb /usr/local/var/postgres -E utf8 # If re-initializing.
postgres -D /usr/local/var/postgres start
createdb `whoami`
psql postgres -c 'CREATE EXTENSION "adminpack";' # Extension pack for PgAdmin
# Install PgAdmin
mkdir -p ~/Library/LaunchAgents
ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist # Auto-launch postgresql
createdb <MY_APP_NAME>
python manage.py runserver
open localhost:5000
```

### Configure the Application

Replace in `app/config/app.yml`:

```
COMMON: &common
    ...
    SQLALCHEMY_DATABASE_URI: postgresql://<WHO_AM_I>:@localhost:5432/<MY_APP_NAME>
    ADMIN_CREDENTIALS: '<WHO_AM_I>,'
```

*Note:* `<WHO_AM_I>` is your username (i.e. output of `whoami`).

With details from _Configure Droplet_, replace in `app/config/app.yml`:

```
PRODUCTION: &production
    ...
    SQLALCHEMY_DATABASE_URI: postgres://postgres:XXX@dokku-postgres-<MY_APP_NAME>:5432/<MY_APP_NAME>
    ADMIN_CREDENTIALS: 'postgres,XXX'
```

Make sure to commit these changes:

```
git add . && git commit -m "My new config"
```

### Deploy the Application

```
cd ~/git/<MY_APP_NAME>
# Make changes
git add . && git commit -m "My new changes"
git remote add dokku dokku@<MY_APP_NAME>.example.com:<MY_APP_NAME>
git push dokku master
ssh root@<MY_DROPLET_IP_ADDRESS>
dokku postgres:link <MY_APP_NAME> <MY_APP_NAME>
exit
open <MY_APP_NAME>.example.com
```
