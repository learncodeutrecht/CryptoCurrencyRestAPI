# CryptoCurrencyRestAPI

## docker-compose

https://docs.docker.com/compose/django/#connect-the-database

create project

```
docker-compose run web django-admin.py startproject CryptoCurrencyRestAPI .
```

run docker-compose

```
docker-compose up
```

and navigate to 127.0.0.1:8000

## deployment

Running on DigitalOcean droplet, adress: http://206.189.12.80/

## How was this Django project deployed?

This was done following this tutorial:
https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-ubuntu-16-04

### Summary (copied from tutorial, in case the link goes dead):

#### Install Packages from the Ubuntu Repositories

sudo apt-get update
sudo apt-get install python3-pip apache2 libapache2-mod-wsgi-py3
sudo pip3 install virtualenv
mkdir ~/CryptoCurrencyRestAPI
cd ~/CryptoCurrencyRestAPI
virtualenv venv
source venv/bin/activate

#### Inside virtual environment
pip install django

then cloned the current repo (CryptoCurrencyRestAPI repo) into ~/CryptoCurrencyRestAPI

Edited the Django settings.py file:
ALLOWED_HOSTS = ["206.189.12.80"]
Added this to bottom of settings.py file: STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

#### Complete Initial Project Setup

cd ~/myproject
./manage.py makemigrations
./manage.py migrate
./manage.py createsuperuser
./manage.py collectstatic
sudo ufw allow 8000
./manage.py runserver 0.0.0.0:8000

#### Configure Apache
sudo nano /etc/apache2/sites-available/000-default.conf

<VirtualHost *:80>
    . . . (left initial code away for simplicities sake)

    Alias /static /home/rik/CryptoCurrencyRestAPI/static
    <Directory /home/rik/CryptoCurrencyRestAPI/static>
        Require all granted
    </Directory>

    <Directory /home/rik/CryptoCurrencyRestAPI/CryptoCurrencyRestAPI>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>

    WSGIDaemonProcess CryptoCurrencyRestAPI python-home=/home/rik/CryptoCurrencyRestAPI/venv python-path=/home/rik/CryptoCurrencyRestAPI
    WSGIProcessGroup CryptoCurrencyRestAPI
    WSGIScriptAlias / /home/rik/CryptoCurrencyRestAPI/CryptoCurrencyRestAPI/wsgi.py

</VirtualHost>

#### Wrapping Up Some Permissions Issues
chmod 664 ~/CryptoCurrencyRestAPI/mydatabase
sudo chown :www-data ~/CryptoCurrencyRestAPI/dmydatabase
sudo chown :www-data ~/CryptoCurrencyRestAPI
sudo ufw delete allow 8000
sudo ufw allow 'Apache Full'
sudo apache2ctl configtest (last line should read: "Syntax OK")
sudo systemctl restart apache2
