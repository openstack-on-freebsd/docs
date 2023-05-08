Launching Instances
===================

Creating Flavor
---------------

```bash
. ~/admin-openrc
```

```bash
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano
+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| description                | None    |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```

Creating Keypair
----------------

```bash
. ~/demo-openrc
```

```bash
$ ssh-keygen -q -N ""
Enter file in which to save the key (/usr/home/freebsd/.ssh/id_rsa):
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| created_at  | None                                            |
| fingerprint | 3d:2e:d6:a7:7d:d5:b2:5e:c3:9e:29:17:91:c3:98:4e |
| id          | mykey                                           |
| is_deleted  | None                                            |
| name        | mykey                                           |
| type        | ssh                                             |
| user_id     | 4c0c4007690a490f9325d27a10f6c03e                |
+-------------+-------------------------------------------------+
```

Creating Security Group Rules
-----------------------------

```bash
$ openstack security group rule create --proto icmp default
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2022-12-19T15:58:24Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 9a2b4157-68c6-495e-8f5f-0dba1dbedb00 |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | None                                 |
| port_range_min          | None                                 |
| project_id              | 1cb65b04c33e43dbb647dc3cdca78505     |
| protocol                | icmp                                 |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 31c09819-ecc4-4b94-87f2-325968d7e11a |
| tags                    | []                                   |
| updated_at              | 2022-12-19T15:58:24Z                 |
+-------------------------+--------------------------------------+
$ openstack security group rule create --proto tcp --dst-port 22 default
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| created_at              | 2022-12-19T16:00:19Z                 |
| description             |                                      |
| direction               | ingress                              |
| ether_type              | IPv4                                 |
| id                      | 28cf0d84-0c0f-4195-ae9a-3005c697c38d |
| name                    | None                                 |
| normalized_cidr         | 0.0.0.0/0                            |
| port_range_max          | 22                                   |
| port_range_min          | 22                                   |
| project_id              | 1cb65b04c33e43dbb647dc3cdca78505     |
| protocol                | tcp                                  |
| remote_address_group_id | None                                 |
| remote_group_id         | None                                 |
| remote_ip_prefix        | 0.0.0.0/0                            |
| revision_number         | 0                                    |
| security_group_id       | 31c09819-ecc4-4b94-87f2-325968d7e11a |
| tags                    | []                                   |
| updated_at              | 2022-12-19T16:00:19Z                 |
+-------------------------+--------------------------------------+
```

Launching Instances
-------------------

```bash
$ openstack flavor list
+----+---------+-----+------+-----------+-------+-----------+
| ID | Name    | RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+---------+-----+------+-----------+-------+-----------+
| 0  | m1.nano |  64 |    1 |         0 |     1 | True      |
+----+---------+-----+------+-----------+-------+-----------+
$ openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| c162e97c-b273-4250-b44e-79e01034a131 | cirros | active |
+--------------------------------------+--------+--------+
$ openstack network list
+--------------------------------------+-----------+--------------------------------------+
| ID                                   | Name      | Subnets                              |
+--------------------------------------+-----------+--------------------------------------+
| e077a70a-2726-4694-bdc7-2a92b7adb419 | provider1 | 9f5e4252-685a-4b19-aaa7-9da2636718d2 |
+--------------------------------------+-----------+--------------------------------------+
$ openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+------+
| ID                                   | Name    | Description            | Project                          | Tags |
+--------------------------------------+---------+------------------------+----------------------------------+------+
| 31c09819-ecc4-4b94-87f2-325968d7e11a | default | Default security group | 1cb65b04c33e43dbb647dc3cdca78505 | []   |
+--------------------------------------+---------+------------------------+----------------------------------+------+
```

```bash
$ wget https://ftp.tw.freebsd.org/pub/FreeBSD/releases/VM-IMAGES/13.2-RELEASE/amd64/Latest/FreeBSD-13.2-RELEASE-amd64.raw.xz
$ xz -d FreeBSD-13.2-RELEASE-amd64.raw.xz
$ . ~/admin-openrc.sh
$ glance image-create --name "freebsd-13.2" --file ~/FreeBSD-13.2-RELEASE-amd64.raw --disk-format raw --container-format bare --visibility=public
+------------------+----------------------------------------------------------------------------------+
| Property         | Value                                                                            |
+------------------+----------------------------------------------------------------------------------+
| checksum         | 2a2535c4bb168d0d0476112882ffae49                                                 |
| container_format | bare                                                                             |
| created_at       | 2023-05-08T17:21:32Z                                                             |
| disk_format      | raw                                                                              |
| id               | ce25fef0-858e-4192-8ea0-85077d4db73d                                             |
| min_disk         | 0                                                                                |
| min_ram          | 0                                                                                |
| name             | freebsd-13.2                                                                     |
| os_hash_algo     | sha512                                                                           |
| os_hash_value    | c8bb0068e45ace16b7f883882880ec9292f9f57380b2f30771846f83edad53686eb972482713fa84 |
|                  | 404c8beb04a6951e65ced7b3678d257e7d4a5e4dd4fa7fa2                                 |
| os_hidden        | False                                                                            |
| owner            | f40187f3d4514163bb251fdb080f25c8                                                 |
| protected        | False                                                                            |
| size             | 6476607488                                                                       |
| status           | active                                                                           |
| tags             | []                                                                               |
| updated_at       | 2023-05-08T17:22:11Z                                                             |
| virtual_size     | 6476607488                                                                       |
| visibility       | public                                                                           |
+------------------+----------------------------------------------------------------------------------+
```

```bash
openstack flavor create --id 2 --vcpus 1 --ram 2048 --disk 20 m1.small
sudo pkg install qemu-tools
```

`etc/nova/nova.conf`:

```
[DEFAULT]
rootwrap_config = etc/nova/rootwrap.conf

[libvirt]
virt_type = bhyve
```

```bash
$ openstack server create --flavor m1.small --image freebsd-13.2 --nic net-id=a13f5b2d-0aee-43f7-a51a-1b44f1d11a57 --security-group default --key-name mykey provider-instance
+-----------------------------+-----------------------------------------------------+
| Field                       | Value                                               |
+-----------------------------+-----------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                              |
| OS-EXT-AZ:availability_zone |                                                     |
| OS-EXT-STS:power_state      | NOSTATE                                             |
| OS-EXT-STS:task_state       | scheduling                                          |
| OS-EXT-STS:vm_state         | building                                            |
| OS-SRV-USG:launched_at      | None                                                |
| OS-SRV-USG:terminated_at    | None                                                |
| accessIPv4                  |                                                     |
| accessIPv6                  |                                                     |
| addresses                   |                                                     |
| adminPass                   | V7PrRLtktXg7                                        |
| config_drive                |                                                     |
| created                     | 2023-05-08T17:24:52Z                                |
| description                 | None                                                |
| flavor                      | m1.small (2)                                        |
| hostId                      |                                                     |
| id                          | a5c018ee-d4ed-4574-a414-777d8e011029                |
| image                       | freebsd-13.2 (ce25fef0-858e-4192-8ea0-85077d4db73d) |
| key_name                    | mykey                                               |
| locked                      | False                                               |
| name                        | provider-instance                                   |
| progress                    | 0                                                   |
| project_id                  | 81eaffe485ed41d3a440a3c62b1bab2d                    |
| properties                  |                                                     |
| security_groups             | name='8061bceb-fc38-4ae5-b685-c2676779e2a8'         |
| status                      | BUILD                                               |
| tags                        |                                                     |
| updated                     | 2023-05-08T17:24:51Z                                |
| user_id                     | 4059f0c2cbd34a7890126351e154b86b                    |
| volumes_attached            |                                                     |
+-----------------------------+-----------------------------------------------------+
```

Checking Instance Status
------------------------

```bash
$ openstack server list
+--------------------------------------+-------------------+--------+--------------------------+--------------+----------+
| ID                                   | Name              | Status | Networks                 | Image        | Flavor   |
+--------------------------------------+-------------------+--------+--------------------------+--------------+----------+
| a5c018ee-d4ed-4574-a414-777d8e011029 | provider-instance | ACTIVE | provider1=192.168.48.203 | freebsd-13.2 | m1.small |
+--------------------------------------+-------------------+--------+--------------------------+--------------+----------+
$ $ openstack server show provider-instance
+-----------------------------+----------------------------------------------------------+
| Field                       | Value                                                    |
+-----------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                   |
| OS-EXT-AZ:availability_zone | nova                                                     |
| OS-EXT-STS:power_state      | Running                                                  |
| OS-EXT-STS:task_state       | None                                                     |
| OS-EXT-STS:vm_state         | active                                                   |
| OS-SRV-USG:launched_at      | 2023-05-08T17:25:49.000000                               |
| OS-SRV-USG:terminated_at    | None                                                     |
| accessIPv4                  |                                                          |
| accessIPv6                  |                                                          |
| addresses                   | provider1=192.168.48.203                                 |
| config_drive                |                                                          |
| created                     | 2023-05-08T17:24:51Z                                     |
| description                 | None                                                     |
| flavor                      | m1.small (2)                                             |
| hostId                      | 13cf4de6d83c51f12566b4149a614582db4b63029afdec369152c412 |
| id                          | a5c018ee-d4ed-4574-a414-777d8e011029                     |
| image                       | freebsd-13.2 (ce25fef0-858e-4192-8ea0-85077d4db73d)      |
| key_name                    | mykey                                                    |
| locked                      | False                                                    |
| name                        | provider-instance                                        |
| progress                    | 0                                                        |
| project_id                  | 81eaffe485ed41d3a440a3c62b1bab2d                         |
| properties                  |                                                          |
| security_groups             | name='default'                                           |
| status                      | ACTIVE                                                   |
| tags                        |                                                          |
| updated                     | 2023-05-08T17:25:49Z                                     |
| user_id                     | 4059f0c2cbd34a7890126351e154b86b                         |
| volumes_attached            |                                                          |
+-----------------------------+----------------------------------------------------------+
```

Connect to the console using `cu`. We need to set up the static IP address the same as Neutron allocated for the VM to get network connectivity, otherwise, all packets cannot pass the virtual switch.

```bash
$ sudo cu -l /dev/nmdm0B
Connected
Mon May  8 17:28:04 UTC 2023

FreeBSD/amd64 (freebsd) (ttyu0)

login: root
May  8 17:28:15 freebsd login[802]: ROOT LOGIN (root) ON ttyu0
FreeBSD 13.2-RELEASE releng/13.2-n254617-525ecfdad597 GENERIC

Welcome to FreeBSD!

Release Notes, Errata: https://www.FreeBSD.org/releases/
Security Advisories:   https://www.FreeBSD.org/security/
FreeBSD Handbook:      https://www.FreeBSD.org/handbook/
FreeBSD FAQ:           https://www.FreeBSD.org/faq/
Questions List:        https://www.FreeBSD.org/lists/questions/
FreeBSD Forums:        https://forums.FreeBSD.org/

Documents installed with the system are in the /usr/local/share/doc/freebsd/
directory, or can be installed later with:  pkg install en-freebsd-doc
For other languages, replace "en" with a language code like de or fr.

Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

To change this login announcement, see motd(5).
root@freebsd:~ # uname -a
FreeBSD freebsd 13.2-RELEASE FreeBSD 13.2-RELEASE releng/13.2-n254617-525ecfdad597 GENERIC amd64
root@freebsd:~ # ifconfig
vtnet0: flags=8863<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        options=80028<VLAN_MTU,JUMBO_MTU,LINKSTATE>
        ether fa:16:3e:b1:69:35
        inet6 fe80::f816:3eff:feb1:6935%vtnet0 prefixlen 64 scopeid 0x1
        inet 192.168.48.166 netmask 0xffffff00 broadcast 192.168.48.255
        media: Ethernet autoselect (10Gbase-T <full-duplex>)
        status: active
        nd6 options=23<PERFORMNUD,ACCEPT_RTADV,AUTO_LINKLOCAL>
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> metric 0 mtu 16384
        options=680003<RXCSUM,TXCSUM,LINKSTATE,RXCSUM_IPV6,TXCSUM_IPV6>
        inet6 ::1 prefixlen 128
        inet6 fe80::1%lo0 prefixlen 64 scopeid 0x2
        inet 127.0.0.1 netmask 0xff000000
        groups: lo
        nd6 options=23<PERFORMNUD,ACCEPT_RTADV,AUTO_LINKLOCAL>
root@freebsd:~ # ifconfig vtnet0 inet 192.168.48.203 netmask 255.255.255.0
root@freebsd:~ # May  8 17:28:49 freebsd dhclient[386]: My address (192.168.48.166) was deleted, dhclient exiting
May  8 17:28:49 freebsd dhclient[386]: connection closed
May  8 17:28:49 freebsd dhclient[386]: exiting.

root@freebsd:~ # route add default 192.168.48.1
add net default: gateway 192.168.48.1
root@freebsd:~ # ping -c 4 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=57 time=3.167 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=4.535 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=2.557 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=2.671 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 2.557/3.233/4.535/0.786 ms
```
