# Classic ONA Installation and Configuration

Steps to install and configure classic ONA in a virtual environment

Clone the ONA repo and rename it

	cd /opt
	git clone https://github.com/onaio/onadata.git
	mv onadata onadata-venv
	cd onadata-venv

Checkout the classic ONA commit

	git checkout 394f06e483c88cdfc7aab31673310e99742c5c3e
	git checkout -b classic_ona

_*From here the configuration is more or less the steps as described in https://github.com/onaio/onadata/blob/master/install.md with a few modifications*_

Become root

	sudo su -
	cd /opt/onadata-venv

## Prepare the system for installation
	./script/install/ubuntu

## Database setup
Modify the username and password accordingly

	su postgres -c "psql -c \"CREATE USER onadata_venv WITH PASSWORD 'onadata_venv';\""
	su postgres -c "psql -c \"CREATE DATABASE onadata_venv OWNER onadata_venv;\""
	su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis;\""
	su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis;\""
	su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis_topology;\""

## Setup and start the virtual environment
	virtualenv .
	source ./bin/activate

Copy the default settings and edit the file appropriately with the credentials used above

	cp onadata/settings/default_settings.py onadata/settings/local_settings.py 

`syncdb` option has been deprecated, so in the file `Makefile` change the line 

	python2 manage.py syncdb --noinput
to 

	python2 manage.py migrate --run-syncdb

Install the requirements and ran make

	bin/pip install -r requirements/base.pip
	make

You may at this point start core with `bin/python manage.py runserver --nothreading` or continue with the setup.
## Compile the API docs
	cd docs
	make html
	cd ..

Copy static files to static dir

	bin/python manage.py collectstatic --noinput

Create the super user that will be used from the UI

	bin/python manage.py createsuperuser

## Setup the uwsgi init script.
The configuration file is missing in the git repo. Download a sample and edit it appropriately

	bin/pip install uwsgi
	wget https://gist.githubusercontent.com/soloincc/43cd165b5c835eb6d6b7132e8ec933b4/raw/45de558b28b144f251491d1e3b785bcb2c2f1699/onadata.conf
	# Edit the config file before moving it
	mv onadata.conf /etc/init/onadata-venv.conf 
	# Start the service
	start onadata-venv
	# Ensure it is running well
	cat /var/log/onadata-venv.log

## Setup celery service.
The config is missing from the git repo, download a sample and edit it appropriately

	apt-get install rabbitmq-server
	wget https://gist.githubusercontent.com/soloincc/6bfe3791f73afdccad8a1ddf07404729/raw/d0f78668cb83c2c957c015e89aaa1231dca44e29/celeryd-ona
	# Edit the config file before moving it
	mv celeryd-ona /etc/default/celeryd-ona-venv
	cp /etc/default/celeryd-ona-venv /etc/init.d/
	chmod +x /etc/init.d/celeryd-ona-venv
	# Create the startup scripts
	update-rc.d celeryd-ona-venv defaults
	# Start the celery service
	service celeryd-ona-venv start

## Setup nginx.
The nginx config file is also missing, download it and edit it appropriately

	apt-get install nginx
	wget https://gist.githubusercontent.com/soloincc/ce791bc0f4bd4b1c8b59f6c321711f6e/raw/32febd14088284dd5bc3a94ebedfd572fad56070/nginx-onadata
	# Edit the config file appropriately
	mv nginx-onadata /etc/nginx/sites-available/onadata-venv
	ln -s /etc/nginx/sites-available/onadata-venv /etc/nginx/sites-enabled/onadata-venv
 	# update and test /etc/nginx/sites-available/onadata-venv
    service nginx configtest
    # remove default nginx server config
    sudo unlink /etc/nginx/sites-enabled/default
    sudo service nginx restart

## Finished
Thats it. Classic ONA should be installed and configured well. To start it ran `bin/python manage.py runserver --nothreading`. If running it from a local machine, go to `http://127.0.0.1:8000` to access it. If running it from a remote server, ensure that nginx is configured well for proxy pass and your firewall is not blocking incoming connections.

## Extras
In order to ran the server using a different port other than the default 8000, edit and make the necessary changes in `/opt/onadata-venv/onadata/settings/common.py` and then start the server bound at that point `bin/python manage.py runserver 127.0.0.1:<port> --nothreading`