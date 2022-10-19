Placement API
=============

Building
--------

```bash
git clone https://github.com/openstack/placement.git
cd placement/
git checkout tags/6.0.0 -b v6.0.0
```

```bash
sudo pkg install python
python -m venv .venv
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

### Configuration File

```bash
cp etc/placement/placement.conf.sample etc/placement/placement.conf
```

```
[DEFAULT]

[api]

auth_strategy = keystone

[cors]

[keystone_authtoken]

www_authenticate_uri = http://osf-keystone:5000/
auth_url = http://osf-keystone:5000/
memcached_servers = osf-keystone:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = password

[oslo_policy]

[placement]

[placement_database]

connection = mysql+pymysql://placement:password@localhost/placement

[profiler]

```

### Database

Setup MySQL server.

```bash
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

Create essential database and users with rightful privileges for `placement`.

```sql
CREATE DATABASE placement;
CREATE USER 'placement'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost';
CREATE USER 'placement'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%';
FLUSH PRIVILEGES;
```

### Initialization

Configure user and endpoints.

```bash
pip install python-openstackclient
```

```bash
. admin-openrc
```

```bash
openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://osf-nova:8778
openstack endpoint create --region RegionOne placement internal http://osf-nova:8778
openstack endpoint create --region RegionOne placement admin http://osf-nova:8778
```

```bash
placement-manage --config-file etc/placement.conf db sync
```

```bash
```

Running
-------

```
pip install uwsgi
```

```bash
OS_PLACEMENT_CONFIG_DIR=etc/placement uwsgi -M --http 0.0.0.0:8778 --wsgi-file $(which placement-api) --processes 2 --threads 10
```