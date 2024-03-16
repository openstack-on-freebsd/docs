Placement API
=============

Building
--------

```bash
git clone https://github.com/openstack/placement.git
cd placement/
git checkout origin/unmaintained/xena -b unmaintained/xena
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
pip install . -r https://raw.githubusercontent.com/openstack-on-freebsd/docs/main/placement/placement-requirements.txt
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

www_authenticate_uri = http://keystone:5000/
auth_url = http://keystone:5000/
memcached_servers = keystone:11211
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
openstack user create --domain default --password-prompt placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://placement:8778
openstack endpoint create --region RegionOne placement internal http://placement:8778
openstack endpoint create --region RegionOne placement admin http://placement:8778
```

```bash
pip install pymysql
```

```bash
placement-manage --config-file etc/placement/placement.conf db sync
```

Running
-------

```
pip install python-memcached uwsgi
```

```bash
OS_PLACEMENT_CONFIG_DIR=etc/placement uwsgi -M --http 0.0.0.0:8778 --wsgi-file $(which placement-api) --processes 2 --threads 10
```

Verifying
---------

```bash
pip install osc-placement
```

```bash
. admin-openrc
```

```bash
$ openstack --os-placement-api-version 1.2 resource class list --sort-column name
+----------------------------------------+
| name                                   |
+----------------------------------------+
| DISK_GB                                |
| FPGA                                   |
| IPV4_ADDRESS                           |
| MEMORY_MB                              |
| MEM_ENCRYPTION_CONTEXT                 |
| NET_BW_EGR_KILOBIT_PER_SEC             |
| NET_BW_IGR_KILOBIT_PER_SEC             |
| NET_PACKET_RATE_EGR_KILOPACKET_PER_SEC |
| NET_PACKET_RATE_IGR_KILOPACKET_PER_SEC |
| NET_PACKET_RATE_KILOPACKET_PER_SEC     |
| NUMA_CORE                              |
| NUMA_MEMORY_MB                         |
| NUMA_SOCKET                            |
| NUMA_THREAD                            |
| PCI_DEVICE                             |
| PCPU                                   |
| PGPU                                   |
| SRIOV_NET_VF                           |
| VCPU                                   |
| VGPU                                   |
| VGPU_DISPLAY_HEAD                      |
+----------------------------------------+
$ openstack --os-placement-api-version 1.6 trait list --sort-column name
+---------------------------------------+
| name                                  |
+---------------------------------------+
| COMPUTE_ACCELERATORS                  |
| COMPUTE_ARCH_AARCH64                  |
| COMPUTE_ARCH_MIPSEL                   |
| COMPUTE_ARCH_PPC64LE                  |
| COMPUTE_ARCH_RISCV64                  |
| COMPUTE_ARCH_S390X                    |
| COMPUTE_ARCH_X86_64                   |
| COMPUTE_CONFIG_DRIVE_REGENERATION     |
...
```