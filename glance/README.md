Glance
======

Building
--------

```bash
git clone https://github.com/openstack/glance.git
cd glance/
#git checkout tags/23.0.0 -b v23.0.0
git checkout origin/stable/xena -b stable/xena
```

```bash
sudo pkg install python39
python3.9 -m venv .venv
source .venv/bin/activate
```

```bash
python -m pip install --upgrade pip wheel
```

```bash
sudo pkg install rust
```

```bash
pip install .
```

### Generating Default Configuration Files

```bash
pip install tox
sudo pkg install sqlite3 py39-sqlite3 postgresql14-client
```

```bash
tox -egenconfig
tox -egenpolicy
```

Configuring
-----------

```bash
cp etc/glance-api.conf etc/glance-api.conf.orig
cp etc/glance-api.conf.sample etc/glance-api.conf 
```

vim etc/glance-api.conf

```
[DEFAULT]

[barbican]

[barbican_service_user]

[cinder]

[cors]

[database]

connection = mysql+pymysql://glance:password@localhost/glance

[file]

[glance.store.http.store]

[glance.store.rbd.store]

[glance.store.s3.store]

[glance.store.swift.store]

[glance.store.vmware_datastore.store]

[glance_store]

stores = file,http

default_store = file

filesystem_store_datadir = /usr/home/freebsd/glance/var/lib/glance/images

[healthcheck]

[image_format]

[key_manager]

[keystone_authtoken]

www_authenticate_uri = http://keystone:5000
auth_url = http://keystone:5000
memcached_servers = keystone:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = password

[oslo_concurrency]

[oslo_messaging_amqp]

[oslo_messaging_kafka]

[oslo_messaging_notifications]

[oslo_messaging_rabbit]

[oslo_middleware]

[oslo_policy]

[oslo_reports]

[paste_deploy]

flavor = keystone

[profiler]

[store_type_location_strategy]

[task]

[taskflow_executor]

[vault]

[wsgi]

```

```bash
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

```sql
CREATE DATABASE glance;
CREATE USER 'glance'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost';
CREATE USER 'glance'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%';
FLUSH PRIVILEGES;
QUIT
```

```bash
pip install python-openstackclient
```

```bash
. ~/admin-openrc 
```

```bash
openstack user create --domain default --password-prompt glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
openstack endpoint create --region RegionOne image public http://glance:9292
openstack endpoint create --region RegionOne image internal http://glance:9292
openstack endpoint create --region RegionOne image admin http://glance:9292
```

```bash
pip install pymysql
```

```bash
sudo pw user add -n glance -c 'OpenStack Image Service' -d /nonexistent -s /usr/sbin/nologin
```

```bash
glance-manage --config-file etc/glance-api.conf db_sync
```

Running
-------

```bash
pip install python-memcached
```

```bash
glance-api --config-file etc/glance-api.conf
```

### Adding Images

```bash
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img -O ~/cirros-0.4.0-x86_64-disk.img
```

```bash
pip install python-glanceclient
```

```bash
. ~/admin-openrc
```

```bash
glance image-create --name "cirros" --file ~/cirros-0.4.0-x86_64-disk.img --disk-format qcow2 --container-format bare --visibility=public
```