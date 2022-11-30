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

### Controller Node

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

#### Setting up Database

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

#### Initialization

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

### Compute Nodes

We use the simplest possible architecture that only supports attaching instances to provider networks. No self-service networks, routers, or floating IP addresses. Only the admin or other privileged user can manage provider networks.

#### Open vSwitch

Install and configure Open vSwitch on compute nodes.

```bash
sudo pkg install openvswitch
```

```bash
sudo sysrc ovsdb_server_enable=yes
sudo sysrc ovs_vswitchd_enable=yes
sudo service ovsdb-server start
sudo service ovs-vswitchd start
```

#### Configuring

`etc/neutron.conf`:

```
[DEFAULT]
core_plugin = ml2
auth_strategy = keystone
transport_url = rabbit://openstack:password@rabbitmq
state_path = /usr/home/freebsd/neutron/var/lib/neutron

[agent]

[cache]

[cors]

[database]

[healthcheck]

[ironic]

[keystone_authtoken]

[nova]

[oslo_concurrency]

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

```bash
mkdir -p var/lib/neutron
```

`etc/neutron/plugins/ml2/openvswitch_agent.ini`:

```
[DEFAULT]

[agent]
minimize_polling = false

[network_log]

[ovs]
bridge_mappings = provider:br-provider
datapath_type = netdev

[securitygroup]
firewall_driver = openvswitch

```

`etc/dhcp_agent.ini`:

```
[DEFAULT]
interface_driver = openvswitch
enable_isolated_metadata = True
force_metadata = True

[agent]

[ovs]

```

`etc/metadata_agent.ini`:

```
[DEFAULT]
nova_metadata_host = nova
metadata_proxy_shared_secret = s3cret

[agent]

[cache]

```

Create provider bridge `br-provider`.

```bash
sudo ovs-vsctl add-br br-provider -- set bridge br-provider datapath_type=netdev
```

Add the provider network interface into the provider bridge.

```bash
sudo ovs-vsctl add-port br-provider vtnet1
```

Patching
--------

Source code patching specifically for FreeBSD.

### Controller Node

```
--- /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py.orig     2022-11-28 15:23:02.796536000 +0000
+++ /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py  2022-11-29 15:24:28.902891000 +0000
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

### Compute Nodes

```
--- /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_privsep/daemon.py.orig     2022-11-30 11:42:37.739778000 +0000
+++ /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_privsep/daemon.py  2022-11-30 11:48:20.825033000 +0000
@@ -67,7 +67,8 @@
 from oslo_privsep import capabilities
 from oslo_privsep import comm

-if platform.system() == 'Linux':
+# HACK(starbops): add FreeBSD support
+if platform.system() == 'Linux' or platform.system() == 'FreeBSD':
     import fcntl
     import grp
     import pwd
@@ -405,7 +406,8 @@
     def _drop_privs(self):
         try:
             # Keep current capabilities across setuid away from root.
-            capabilities.set_keepcaps(True)
+            # HACK(starbops): bypass Linux capabilities on FreeBSD
+            #capabilities.set_keepcaps(True)

             if self.group is not None:
                 try:
@@ -422,12 +424,15 @@
                 setgid(self.group)

         finally:
-            capabilities.set_keepcaps(False)
+            # HACK(starbops): bypass Linux capabilities on FreeBSD
+            #capabilities.set_keepcaps(False)
+            pass

         LOG.info('privsep process running with uid/gid: %(uid)s/%(gid)s',
                  {'uid': os.getuid(), 'gid': os.getgid()})

-        capabilities.drop_all_caps_except(self.caps, self.caps, [])
+        # HACK(starbops): bypass Linux capabilities on FreeBSD
+        #capabilities.drop_all_caps_except(self.caps, self.caps, [])

         def fmt_caps(capset):
             if not capset:
@@ -437,15 +442,16 @@
             fc.sort()
             return '|'.join(fc)

-        eff, prm, inh = capabilities.get_caps()
-        LOG.info(
-            'privsep process running with capabilities '
-            '(eff/prm/inh): %(eff)s/%(prm)s/%(inh)s',
-            {
-                'eff': fmt_caps(eff),
-                'prm': fmt_caps(prm),
-                'inh': fmt_caps(inh),
-            })
+        # HACK(starbops): bypass Linux capabilities on FreeBSD
+        #eff, prm, inh = capabilities.get_caps()
+        #LOG.info(
+        #    'privsep process running with capabilities '
+        #    '(eff/prm/inh): %(eff)s/%(prm)s/%(inh)s',
+        #    {
+        #        'eff': fmt_caps(eff),
+        #        'prm': fmt_caps(prm),
+        #        'inh': fmt_caps(inh),
+        #    })

     def _process_cmd(self, msgid, cmd, *args):
         """Executes the requested command in an execution thread.
```

```
--- /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py.orig     2022-11-29 15:38:25.869655000 +0000
+++ /usr/home/freebsd/neutron/.venv/lib/python3.8/site-packages/oslo_messaging/_drivers/impl_rabbit.py  2022-11-30 14:35:23.291480000 +0000
@@ -1077,8 +1077,10 @@
                       % (self.connection_id, str(e)))
         else:
             sock.settimeout(timeout)
+            # HACK(starbops): add FreeBSD support
             # TCP_USER_TIMEOUT is not defined on Windows and Mac OS X
-            if sys.platform != 'win32' and sys.platform != 'darwin':
+            if (sys.platform != 'win32' and sys.platform != 'darwin' and
+                    not sys.platform.startswith('freebsd')):
                 try:
                     timeout = timeout * 1000 if timeout is not None else 0
                     # NOTE(gdavoian): only integers and strings are allowed
```

Running
-------

### Controller Node

```bash
pip install python-memcached
```

```bash
neutron-server --config-file etc/neutron.conf --config-file etc/neutron/plugins/ml2/ml2_conf.ini
```

### Compute Nodes

```bash
neutron-openvswitch-agent --config-file etc/neutron.conf --config-file etc/neutron/plugins/ml2/openvswitch_agent.ini
```

```bash
neutron-dhcp-agent --config-file etc/neutron.conf --config-file etc/dhcp_agent.ini
```

```bash
neutron-metadata-agent --config-file etc/neutron.conf --config-file etc/metadata_agent.ini
```

Verifying
---------

```bash
. ~/admin-openrc
```

```bash
$ openstack network agent list
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| 5d6dd635-b4c1-4399-80fa-8ed67428e1fb | Open vSwitch agent | nova | None              | :-)   | UP    | neutron-openvswitch-agent |
| aa69830f-7525-4070-999c-c23d0c70f29d | Metadata agent     | nova | None              | :-)   | UP    | neutron-metadata-agent    |
| c77190d3-e527-4fcd-be2a-616c8d412ccc | DHCP agent         | nova | nova              | :-)   | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
```