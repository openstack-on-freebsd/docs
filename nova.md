Nova
====

Building
--------

```bash
git clone https://github.com/openstack/nova.git
cd nova/
git checkout origin/stable/xena -b stable/xena
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
sudo pkg install libxml2 libxslt rust 
```

```bash
pip install .
```

### Generating Default Configuration Files

Edit `test-requirements.txt` and comment out `oslo.vmware` dependency because it introduces another dependency, `suds-jurko`, which will cause dependency installation error while generating configuration files via `tox` later on.

```
#oslo.vmware>=3.6.0 # Apache-2.0
```

```bash
pip install tox
sudo pkg install postgresql14-client
```

```bash
tox -egenconfig
tox -egenpolicy
```

Configuring
-----------

```bash
cp etc/nova/nova.conf.sample etc/nova/nova.conf
cp etc/nova/policy.yaml.sample etc/nova/policy.yaml
```

`etc/nova/nova.conf`:

```
[DEFAULT]
my_ip = 10.0.2.2
transport_url = rabbit://openstack:password@rabbitmq:5672/
log_dir = /usr/home/freebsd/nova/var/log/nova
lock_path = /usr/home/freebsd/nova/var/lock/nova
state_path = /usr/home/freebsd/nova/var/lib/nova

[api]
auth_strategy = keystone

[api_database]
connection = mysql+pymysql://nova:password@localhost/nova_api

[barbican]

[barbican_service_user]

[cache]

[cinder]

[compute]

[conductor]

[console]

[consoleauth]

[cors]

[cyborg]

[database]
connection = mysql+pymysql://nova:password@localhost/nova

[devices]

[ephemeral_storage_encryption]

[filter_scheduler]

[glance]
api_servers = http://glance:9292

[guestfs]

[healthcheck]

[hyperv]

[image_cache]

[ironic]

[key_manager]

[keystone]

[keystone_authtoken]
www_authenticate_uri = http://keystone:5000/
auth_url = http://keystone:5000/
memcached_servers = keystone:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = password

[libvirt]

[metrics]

[mks]

[neutron]
auth_url = http://keystone:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = password
service_metadata_proxy = true
metadata_proxy_shared_secret = s3cret

[notifications]

[oslo_concurrency]
lock_path = /usr/home/freebsd/nova/var/lib/nova/tmp

[oslo_messaging_amqp]

[oslo_messaging_kafka]

[oslo_messaging_notifications]

[oslo_messaging_rabbit]

[oslo_middleware]

[oslo_policy]

[oslo_reports]

[pci]

[placement]
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://keystone:5000/v3
username = placement
password = password

[powervm]

[privsep]

[profiler]

[quota]

[rdp]

[remote_debug]

[scheduler]

[serial_console]

[service_user]

[spice]

[upgrade_levels]

[vault]

[vendordata_dynamic_auth]

[vmware]

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip
novncproxy_base_url = http://nova:6080/vnc_auto.html

[workarounds]

[wsgi]

[zvm]

```

`etc/nova/nova-compute.conf`:

```
[DEFAULT]
compute_driver = libvirt.LibvirtDriver

[libvirt]
virt_type = qemu
```

### Setting up Database

```bash
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

```sql
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
CREATE USER 'nova'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost';
CREATE USER 'nova'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%';
FLUSH PRIVILEGES;
QUIT
```

### Initialization

Configure user and endpoints.

```bash
pip install python-openstackclient
```

```bash
. ~/admin-openrc
```

```bash
openstack user create --domain default --password-prompt nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://nova:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://nova:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://nova:8774/v2.1
```

```bash
pip install pymysql
```

```bash
nova-manage --config-file etc/nova/nova.conf api_db sync
nova-manage --config-file etc/nova/nova.conf cell_v2 map_cell0
nova-manage --config-file etc/nova/nova.conf cell_v2 create_cell --name=cell1 --verbose
nova-manage --config-file etc/nova/nova.conf db sync
```

```bash
$ nova-manage --config-file etc/nova/nova.conf cell_v2 list_cells
Modules with known eventlet monkey patching issues were imported prior to eventlet monkey patching: urllib3. This warning can usually be ignored if the caller is only importing and not executing nova code.
+-------+--------------------------------------+----------------------------------------+------------------------------------------------+----------+
|  Name |                 UUID                 |             Transport URL              |              Database Connection               | Disabled |
+-------+--------------------------------------+----------------------------------------+------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                 none:/                 | mysql+pymysql://nova:****@localhost/nova_cell0 |  False   |
| cell1 | 345fe55a-1331-4d39-9b24-cf56f55c1528 | rabbit://openstack:****@rabbitmq:5672/ |    mysql+pymysql://nova:****@localhost/nova    |  False   |
+-------+--------------------------------------+----------------------------------------+------------------------------------------------+----------+
```

Patching
--------

Source code patching specifically for FreeBSD.

`$HOME/nova/.venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py`:

```python
            # HACK(starbops): add FreeBSD support
            # TCP_USER_TIMEOUT is not defined on Windows, Mac OS X, and FreeBSD
            if (sys.platform != 'win32' and sys.platform != 'darwin' and
                   not sys.platform.startswith('freebsd')):
```

`$HOME/nova/.venv/lib/python3.8/site-packages/nova/virt/libvirt/driver.py`:

```python
        # HACK(starbops): Make it works on FreeBSD platforms
        if (not sys.platform.startswith('linux') and
            not sys.platform.startswith('freebsd')):
```

Running
-------

```bash
sudo pkg install py38-sqlite3
```

```bash
nova-api --config-dir etc/nova
nova-scheduler --config-dir etc/nova
nova-conductor --config-dir etc/nova
```

### Compute

```bash
sudo kldload vmm
sudo kldload nmdm
sudo kldload if_bridge
sudo kldload if_tap
```

```bash
sudo sysrc kld_list="vmm nmdm if_tap if_bridge"
```

```bash
sudo sysctl net.link.tap.up_on_open=1
```

`/etc/sysctl.conf`:

```
net.link.tap.up_on_open=1
```

```bash
sudo pkg install pkgconf libvirt
sudo sysrc libvirtd_enable=yes
sudo service libvirtd start
pip install libvirt-python
```

```bash
sudo nova-compute --config-dir etc/nova
```

Verifying
---------

AMQP
----

```bash
sudo pkg install rabbitmq
sudo sysrc rabbitmq_enable=yes
sudo service rabbitmq start
```

### Enabling Dashboard

```bash
sudo rabbitmq-plugins enable rabbitmq_management
sudo service rabbitmq restart
```

```bash
sudo cp /var/db/rabbitmq/.erlang.cookie /root/
sudo chmod 400 /root/.erlang.cookie
```

```bash
sudo rabbitmqctl add_user admin password
sudo rabbitmqctl set_user_tags admin administrator
```

Go to your browser with `http://<rabbitmq-server-ip>:15672` to open the dashboard.

### Creating User for OpenStack

```bash
sudo rabbitmqctl add_user openstack password
sudo rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```