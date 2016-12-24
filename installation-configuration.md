onadata installation

git clone https://github.com/onaio/onadata.git
mv onadata onadata-venv
cd onadata-venv
git checkout 394f06e483c88cdfc7aab31673310e99742c5c3e
git checkout -b classic_ona
./script/install/ubuntu

# postgres configuration
sudo su postgres -c "psql -c \"CREATE USER onadata_venv WITH PASSWORD 'onadata_venv';\""
sudo su postgres -c "psql -c \"CREATE DATABASE onadata_venv OWNER onadata_venv;\""
sudo su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis;\""
sudo su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis;\""
sudo su postgres -c "psql -d onadata_venv -c \"CREATE EXTENSION IF NOT EXISTS postgis_topology;\""

virtualenv .
source ./bin/activate
cp onadata/settings/default_settings.py onadata/settings/local_settings.py 

# Edit the local_settings.py file appropriately

# syncdb option has been deprecated, so change the line 
python2 manage.py syncdb --noinput
# to 
python2 manage.py migrate --run-syncdb

bin/pip install -r requirements/base.pip
make

# bin/python manage.py runserver --nothreading

cd docs
make html
cd ..

bin/python manage.py collectstatic --noinput
bin/python manage.py createsuperuser

(onadata-venv) root@ubuntu:/opt/onadata-venv# bin/python manage.py createsuperuser
Your environment is:"onadata.settings.common"
/opt/onadata-venv/local/lib/python2.7/site-packages/six.py:808: RemovedInDjango110Warning: SubfieldBase has been deprecated. Use Field.from_db_value instead.
  return meta(name, bases, d)

Username (leave blank to use 'root'): badili
Email address: info@badili.co.ke
Password: 
Password (again): 
Superuser created successfully.

#uwsgi
bin/pip install uwsgi
wget https://gist.githubusercontent.com/soloincc/43cd165b5c835eb6d6b7132e8ec933b4/raw/45de558b28b144f251491d1e3b785bcb2c2f1699/onadata.conf
mv onadata.conf /etc/init/onadata-venv.conf 
start onadata-venv
cat /var/log/onadata-venv.log

#celery
apt-get install rabbitmq-server
wget https://gist.githubusercontent.com/soloincc/6bfe3791f73afdccad8a1ddf07404729/raw/d0f78668cb83c2c957c015e89aaa1231dca44e29/celeryd-ona
mv celeryd-ona /etc/default/celeryd-ona-venv
cp /etc/default/celeryd-ona-venv /etc/init.d/
chmod +x /etc/init.d/celeryd-ona-venv
update-rc.d celeryd-ona-venv defaults
service celeryd-ona-venv start

#nginx
wget https://gist.githubusercontent.com/soloincc/ce791bc0f4bd4b1c8b59f6c321711f6e/raw/32febd14088284dd5bc3a94ebedfd572fad56070/nginx-onadata
mv nginx-onadata /etc/nginx/sites-available/onadata-venv
ln -s /etc/nginx/sites-available/onadata-venv /etc/nginx/sites-enabled/onadata-venv

# to change the port the server is running at, make the necessary changes in onadata/settings/common.py
# start the server bound at that poit
echo "bin/python manage.py runserver 127.0.0.1:8001 --nothreading" > runserver
chmod +x runserver

