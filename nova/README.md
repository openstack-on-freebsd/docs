Nova
====

Building
--------

```bash
git clone https://github.com/openstack-on-freebsd/nova.git
cd nova/
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
sudo pkg install libxml2 libxslt rust gcc
```

```bash
pip install . -r https://raw.githubusercontent.com/openstack-on-freebsd/docs/main/nova/nova-requirements.txt
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
web = /usr/local/libexec/novnc

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
enabled = true

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
enabled = true

[service_user]

[spice]

[upgrade_levels]

[vault]

[vendordata_dynamic_auth]

[vmware]

[vnc]
#enabled = true
enabled = false
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
rootwrap_config = etc/nova/rootwrap.conf

[libvirt]
virt_type = bhyve
connection_uri = bhyve:///system
images_type = raw
```

Add additional directories into `filters_path` and `exec_dirs` in `etc/nova/rootwrap.conf`:

```
[DEFAULT]
filters_path=/usr/home/freebsd/nova/etc/nova/rootwrap.d,/etc/nova/rootwrap.d,/usr/share/nova/rootwrap

exec_dirs=/usr/home/freebsd/nova/.venv/bin,/sbin,/usr/sbin,/bin,/usr/bin,/usr/local/sbin,/usr/local/bin
...
```

```bash
mkdir -p var/lib/nova var/lock/nova var/log/nova
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
export EVENTLET_HUB=poll # Especially for Python 3.9. Ref: https://github.com/eventlet/eventlet/issues/670#issuecomment-735488189
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

```
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_privsep/daemon.py.orig        2022-11-13 11:49:53.372599000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_privsep/daemon.py     2022-12-27 14:33:29.629325000 +0000
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
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_messaging/_drivers/impl_rabbit.py.orig        2022-11-13 11:49:53.930821000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_messaging/_drivers/impl_rabbit.py     2022-11-14 10:39:36.675110000 +0000
@@ -1077,8 +1077,10 @@
                       % (self.connection_id, str(e)))
         else:
             sock.settimeout(timeout)
-            # TCP_USER_TIMEOUT is not defined on Windows and Mac OS X
-            if sys.platform != 'win32' and sys.platform != 'darwin':
+            # HACK(starbops): add FreeBSD support
+            # TCP_USER_TIMEOUT is not defined on Windows, Mac OS X, and FreeBSD
+            if (sys.platform != 'win32' and sys.platform != 'darwin' and
+                   not sys.platform.startswith('freebsd')):
                 try:
                     timeout = timeout * 1000 if timeout is not None else 0
                     # NOTE(gdavoian): only integers and strings are allowed
```

```
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/virt/libvirt/host.py.orig     2022-11-13 11:49:55.347054000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/virt/libvirt/host.py  2022-11-20 06:11:12.993985000 +0000
@@ -741,7 +741,30 @@

         :returns: set of online CPUs, raises libvirtError on error
         """
-        cpus, cpu_map, online = self.get_connection().getCPUMap()
+        # HACK(starbops): mock up getCPUMap() because it is not implemented on
+        # FreeBSD platform
+        #cpus, cpu_map, online = self.get_connection().getCPUMap()
+        cpus, cpu_map, online = (
+                16,
+                [
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True,
+                    True
+                ],
+                16)

         online_cpus = set()
         for cpu in range(cpus):
@@ -1165,7 +1188,8 @@

     @staticmethod
     def _get_avail_memory_kb():
-        with open('/proc/meminfo') as fp:
+        # HACK(starbops): replace with FreeBSD specific path
+        with open('/compat/linux/proc/meminfo') as fp:
             m = fp.read().split()
         idx1 = m.index('MemFree:')
         idx2 = m.index('Buffers:')
@@ -1498,10 +1522,12 @@

         # we don't use '/capabilities/host/cpu/topology' since libvirt doesn't
         # guarantee the accuracy of this information
-        for cell in self.get_capabilities().host.topology.cells:
-            if any(len(cpu.siblings) > 1 for cpu in cell.cpus if cpu.siblings):
-                self._has_hyperthreading = True
-                break
+        # HACK(starbops): FreeBSD support
+        self._has_hyperthreading = False
+        #for cell in self.get_capabilities().host.topology.cells:
+        #    if any(len(cpu.siblings) > 1 for cpu in cell.cpus if cpu.siblings):
+        #        self._has_hyperthreading = True
+        #        break

         return self._has_hyperthreading

```

```
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/virt/libvirt/driver.py.orig   2022-11-13 11:49:55.345720000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/virt/libvirt/driver.py        2022-11-14 10:43:57.675142000 +0000
@@ -425,7 +425,9 @@
         }
         super(LibvirtDriver, self).__init__(virtapi)

-        if not sys.platform.startswith('linux'):
+        # HACK(starbops): Make it works on FreeBSD platforms
+        if (not sys.platform.startswith('linux') and
+            not sys.platform.startswith('freebsd')):
             raise exception.InternalError(
                 _('The libvirt driver only works on Linux'))

```

```
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/conf/libvirt.py.orig  2022-11-13 11:49:54.784253000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/nova/conf/libvirt.py       2022-11-20 07:15:59.609152000 +0000
@@ -104,7 +104,7 @@
 """),
     cfg.StrOpt('virt_type',
                default='kvm',
-               choices=('kvm', 'lxc', 'qemu', 'parallels'),
+               choices=('bhyve', 'kvm', 'lxc', 'qemu', 'parallels'),
                help="""
 Describes the virtualization type (or so called domain type) libvirt should
 use.
```

```
--- /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_rootwrap/cmd.py.orig  2022-11-13 11:49:39.900492000 +0000
+++ /usr/home/freebsd/nova/.venv/lib/python3.9/site-packages/oslo_rootwrap/cmd.py       2022-12-27 15:31:20.180714000 +0000
@@ -99,7 +99,8 @@
         if (fd_limits[0] > sensible_fd_limit):
             # Close any fd beyond sensible_fd_limit prior adjusting our
             # rlimit to ensure all fds are closed
-            for fd_entry in os.listdir('/proc/self/fd'):
+            # HACK(starbops): FreeBSD support
+            for fd_entry in os.listdir('/compat/linux/proc/self/fd'):
                 # NOTE(dmllr): In a previous patch revision non-numeric
                 # dir entries were silently ignored which reviewers
                 # didn't like. Readd exception handling when it occurs.
```

Running
-------

```bash
sudo pkg install py39-sqlite3
```

```bash
pip install python-memcached python-binary-memcached
```

```bash
EVENTLET_HUB=poll nova-api --config-dir etc/nova
EVENTLET_HUB=poll nova-scheduler --config-dir etc/nova
EVENTLET_HUB=poll nova-conductor --config-dir etc/nova
```

### Compute

```bash
sudo sysrc linux_enable=yes
sudo service linux start
```

#### Setting up bhyve

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

#### libvirt

```bash
sudo pkg install pkgconf libvirt
sudo sysrc libvirtd_enable=yes
sudo service libvirtd start
pip install libvirt-python
```

#### nova-compute

```bash
mkdir -p var/lib/nova/instances
```

```bash
sudo EVENTLET_HUB=poll nova-compute --config-dir etc/nova
```

#### Register Cell

Run the following command with the admin credential on the control node:

```bash
$ . ~/admin-openrc
$ EVENTLET_HUB=poll nova-manage --config-file etc/nova/nova.conf cell_v2 discover_hosts --verbose
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': 345fe55a-1331-4d39-9b24-cf56f55c1528
Checking host mapping for compute host 'nova': bdf0e6b6-b775-4831-8667-f9bfc6b3f425
Creating host mapping for compute host 'nova': bdf0e6b6-b775-4831-8667-f9bfc6b3f425
Found 1 unmapped computes in cell: 345fe55a-1331-4d39-9b24-cf56f55c1528
```

It's likely you're hitting the max connection limit of MySQL here. Try the following to check if that happens and temporarily increase the cap (the default value is `100`):

```bash
> show variables like "max_connections";
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
> set global max_connections = 500;
```

To persist the change, add the line under `[mysqld]` section in the file `/usr/local/etc/mysql/my.cnf`:

```
[mysqld]
max_connections = 500
```

### nova-novncproxy & nova-serialproxy

```bash
sudo pkg install novnc websockify
```

```bash
nova-novncproxy --config-dir etc/nova
```

```bash
EVENTLET_HUB=poll nova-serialproxy --config-dir etc/nova
```

Deploy socat-manager
--------------------

Install the dependency:

```bash
sudo pkg install socat
```

Run the server (on each compute node):

```bash
git clone https://github.com/openstack-on-freebsd/socat-manager.git
cd socat-manager/
sudo python3.9 server.py
```

Install the bhyve hook script (on each compute node):

```bash
sudo mkdir /usr/local/etc/libvirt/hooks
sudo cp hooks/bhyve /usr/local/etc/libvirt/hooks/
sudo chmod 755 /usr/local/etc/libvirt/hooks/bhyve
sudo service libvirtd restart
```

Verifying
---------

```bash
. ~/admin-openrc
```

```bash
$ openstack compute service list
+--------------------------------------+----------------+------+----------+---------+-------+----------------------------+
| ID                                   | Binary         | Host | Zone     | Status  | State | Updated At                 |
+--------------------------------------+----------------+------+----------+---------+-------+----------------------------+
| b415b8bf-ede1-472b-8321-1548bc4fdc2b | nova-conductor | nova | internal | enabled | up    | 2022-11-20T08:20:36.000000 |
| f3e2e820-85a8-4b97-91c1-d19f5a34623a | nova-scheduler | nova | internal | enabled | up    | 2022-11-20T08:20:34.000000 |
| ce83b73e-0092-48f6-bd4f-684ce3f472c4 | nova-compute   | nova | nova     | enabled | up    | 2022-11-20T08:20:41.000000 |
+--------------------------------------+----------------+------+----------+---------+-------+----------------------------+
```

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