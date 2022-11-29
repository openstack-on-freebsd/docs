Neutron
=======

Building
--------

```bash
git clone https://github.com/openstack/neutron.git
cd neutron/
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

```bash
pip install tox
sudo pkg install sqlite3 py38-sqlite3
```

```bash
tox -egenconfig
tox -egenpolicy
```

Configuring
-----------

`etc/neutron.conf`:

```
[DEFAULT]
auth_strategy = keystone
core_plugin = ml2
service_plugins =
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
transport_url = rabbit://openstack:password@rabbitmq

[agent]

[cache]

[cors]

[database]
connection = mysql+pymysql://neutron:password@localhost/neutron

[healthcheck]

[ironic]

[keystone_authtoken]
www_authenticate_uri = http://keystone:5000
auth_url = http://keystone:5000
memcached_servers = keystone:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = password

[nova]
auth_url = http://keystone:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = password

[oslo_concurrency]
lock_path = /usr/home/freebsd/neutron/var/lib/neutron/tmp

[oslo_messaging_amqp]

[oslo_messaging_kafka]

[oslo_messaging_notifications]

[oslo_messaging_rabbit]

[oslo_middleware]

[oslo_policy]

[oslo_reports]

[placement]

[privsep]

[quotas]

[ssl]

```

`etc/neutron/plugins/ml2/ml2_conf.ini`:

```
[DEFAULT]

[ml2]
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = openvswitch
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_geneve]

[ml2_type_gre]

[ml2_type_vlan]
network_vlan_ranges = provider

[ml2_type_vxlan]

[ovs_driver]

[securitygroup]

[sriov_driver]

```

```bash
mkdir -p var/lib/neutron
```

### Setting up Database

```bash
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

```sql
CREATE DATABASE neutron;
CREATE USER 'neutron'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost';
CREATE USER 'neutron'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%';
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
openstack user create --domain default --password-prompt neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://neutron:9696
openstack endpoint create --region RegionOne network internal http://neutron:9696
openstack endpoint create --region RegionOne network admin http://neutron:9696
```

```bash
pip install pymysql
```

```bash
neutron-db-manage --config-file etc/neutron.conf --config-file etc/neutron/plugins/ml2/ml2_conf.ini upgrade head
```

Patching
--------

Source code patching specifically for FreeBSD.

```
--- .venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py.orig       2022-11-28 15:23:02.796536000 +0000
+++ .venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py    2022-11-29 15:24:28.902891000 +0000
@@ -1077,8 +1077,10 @@
                       % (self.connection_id, str(e)))
         else:
             sock.settimeout(timeout)
-            # TCP_USER_TIMEOUT is not defined on Windows and Mac OS X
-            if sys.platform != 'win32' and sys.platform != 'darwin':
+            # HACK(starbops): add FreeBSD support
+            # TCP_USER_TIMEOUT is not defined on Windows, Mac OS X, and FreeBSD
+            if (sys.platform != 'win32' and sys.platform != 'darwin' and
+                    not sys.platform.startswith('freebsd')):
                 try:
                     timeout = timeout * 1000 if timeout is not None else 0
                     # NOTE(gdavoian): only integers and strings are allowed
```

Running
-------

```bash
neutron-server --config-file etc/neutron.conf --config-file etc/neutron/plugins/ml2/ml2_conf.ini
```

### Configuring Networking Options

We use the simplest possible architecture that only supports attaching instances to provider networks. No self-service networks, routers, or floating IP addresses. Only the admin or other privileged user can manage provider networks.

Running
-------

### Controller Node

### Compute Nodes

```bash
neutron-openvswitch-agent --config-file etc/neutron.conf --config-file etc/neutron/plugins/ml2/openvswitch_agent.ini
```

```bash
neutron-dhcp-agent --config-file etc/neutron.conf --config-file etc/dhcp_agent.ini
```

Open vSwitch
------------

Install and configure Open vSwitch on compute nodes.

### Installation

```bash
sudo pkg install openvswitch
```

```bash
sudo sysrc ovsdb_server_enable=yes
sudo sysrc ovs_vswitchd_enable=yes
sudo service ovsdb-server start
sudo service ovs-vswitchd start
```

### Pre-configuration

```bash
sudo ovs-vsctl add-br br-provider -- set bridge br-provider datapath_type=netdev
```