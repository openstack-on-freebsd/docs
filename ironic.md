Ironic
======


Prerequisites
-------------

```bash
sudo pkg install python
```

```bash
git clone https://github.com/openstack/ironic.git
cd ironic/
git checkout tags/18.2.1 -b v18.2.1
```

Then create virtual environment for testing:

```bash
python3.8 -m venv .venv
source .venv/bin/activate
```

Upgrade `pip` to the latest version:

```bash
python -m pip install --upgrade pip wheel
```

```bash
sudo pkg install rust
```

Build and Install Ironic Packages
---------------------------------

```bash
pip install .
```

Config Generation
-----------------

```bash
sudo pkg install postgresql14-client
```

```bash
pip install tox
tox -egenconfig
tox -egenpolicy
```

Database
--------

Setup MySQL server.

```bash
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

Create essential database for ironic.

```sql
CREATE DATABASE ironic;
```

Configuration
-------------

```bash
cp etc/ironic/ironic.conf.sample etc/ironic/ironic.conf.local

sed -i "" "s/#auth_strategy = keystone/auth_strategy = noauth/" etc/ironic/ironic.conf.local
sed -i "" "s/#enabled_hardware_types = .*/enabled_hardware_types = fake-hardware/" etc/ironic/ironic.conf.local
sed -i "" "s/#enabled_deploy_interfaces = .*/enabled_deploy_interfaces = fake/" etc/ironic/ironic.conf.local
sed -i "" "s/#enabled_boot_interfaces = .*/enabled_boot_interfaces = fake/" etc/ironic/ironic.conf.local
sed -i "" "s/#enabled_management_interfaces = .*/enabled_management_interfaces = fake,ipmitool/" etc/ironic/ironic.conf.local
sed -i "" "s/#enabled_power_interfaces = .*/enabled_power_interfaces = fake,ipmitool/" etc/ironic/ironic.conf.local
sed -i "" "s/#sync_power_state_interval = 60/sync_power_state_interval = 604800/" etc/ironic/ironic.conf.local
sed -i "" "s/#connection = .*/connection = mysql+pymysql://root:password@localhost/ironic" etc/ironic/ironic.conf.local
sed -i "" "s/#rpc_transport = oslo/rpc_transport = json-rpc/" etc/ironic/ironic.conf.local
```

```bash
pip install pymysql
```

```bash
ironic-dbsync --config-file etc/ironic/ironic.conf.local create_schema
```

Kickstart Service
-----------------

```bash
sudo pkg install ipmitool
```

```bash
ironic-api -d --config-file etc/ironic/ironic.conf.local
```

```bash
ironic-conductor -d --config-file etc/ironic/ironic.conf.local
```

Verify Installation
-------------------

```bash
pip install python-openstackclient python-ironicclient
```

```bash
export OS_AUTH_TYPE=none
export OS_ENDPOINT=http://controller:6385/
```

```bash
$ baremetal driver list          
+---------------------+---------------------------+          
| Supported driver(s) | Active host(s)            |          
+---------------------+---------------------------+          
| fake-hardware       | osf-1.internal.zespre.com |          
+---------------------+---------------------------+          
$ baremetal node list
<empty>
```
