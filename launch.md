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
$ openstack server create --flavor m1.nano --image cirros --nic net-id=e077a70a-2726-4694-bdc7-2a92b7adb419 --security-group=default --key-name mykey provider-instance
+-----------------------------+-----------------------------------------------+
| Field                       | Value                                         |
+-----------------------------+-----------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                        |
| OS-EXT-AZ:availability_zone |                                               |
| OS-EXT-STS:power_state      | NOSTATE                                       |
| OS-EXT-STS:task_state       | scheduling                                    |
| OS-EXT-STS:vm_state         | building                                      |
| OS-SRV-USG:launched_at      | None                                          |
| OS-SRV-USG:terminated_at    | None                                          |
| accessIPv4                  |                                               |
| accessIPv6                  |                                               |
| addresses                   |                                               |
| adminPass                   | oGx7DZ3XGGMX                                  |
| config_drive                |                                               |
| created                     | 2022-12-19T16:04:42Z                          |
| flavor                      | m1.nano (0)                                   |
| hostId                      |                                               |
| id                          | 8720a5af-b5ce-4b68-a005-b88e5ecf0518          |
| image                       | cirros (c162e97c-b273-4250-b44e-79e01034a131) |
| key_name                    | mykey                                         |
| name                        | provider-instance                             |
| progress                    | 0                                             |
| project_id                  | 1cb65b04c33e43dbb647dc3cdca78505              |
| properties                  |                                               |
| security_groups             | name='31c09819-ecc4-4b94-87f2-325968d7e11a'   |
| status                      | BUILD                                         |
| updated                     | 2022-12-19T16:04:42Z                          |
| user_id                     | 4c0c4007690a490f9325d27a10f6c03e              |
| volumes_attached            |                                               |
+-----------------------------+-----------------------------------------------+
```

Checking Instance Status
------------------------

```bash
$ openstack server list
+--------------------------------------+-------------------+--------+----------+--------+---------+
| ID                                   | Name              | Status | Networks | Image  | Flavor  |
+--------------------------------------+-------------------+--------+----------+--------+---------+
| 8720a5af-b5ce-4b68-a005-b88e5ecf0518 | provider-instance | ERROR  |          | cirros | m1.nano |
+--------------------------------------+-------------------+--------+----------+--------+---------+
$ openstack server show provider-instance
+-----------------------------+------------------------------------------------------------------------------------------------------+
| Field                       | Value                                                                                                |
+-----------------------------+------------------------------------------------------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                                                               |
| OS-EXT-AZ:availability_zone |                                                                                                      |
| OS-EXT-STS:power_state      | NOSTATE                                                                                              |
| OS-EXT-STS:task_state       | None                                                                                                 |
| OS-EXT-STS:vm_state         | error                                                                                                |
| OS-SRV-USG:launched_at      | None                                                                                                 |
| OS-SRV-USG:terminated_at    | None                                                                                                 |
| accessIPv4                  |                                                                                                      |
| accessIPv6                  |                                                                                                      |
| addresses                   |                                                                                                      |
| config_drive                |                                                                                                      |
| created                     | 2022-12-19T16:04:42Z                                                                                 |
| fault                       | {'code': 400, 'created': '2022-12-19T16:04:44Z', 'message': "Host 'nova' is not mapped to any cell"} |
| flavor                      | m1.nano (0)                                                                                          |
| hostId                      |                                                                                                      |
| id                          | 8720a5af-b5ce-4b68-a005-b88e5ecf0518                                                                 |
| image                       | cirros (c162e97c-b273-4250-b44e-79e01034a131)                                                        |
| key_name                    | mykey                                                                                                |
| name                        | provider-instance                                                                                    |
| project_id                  | 1cb65b04c33e43dbb647dc3cdca78505                                                                     |
| properties                  |                                                                                                      |
| status                      | ERROR                                                                                                |
| updated                     | 2022-12-19T16:04:44Z                                                                                 |
| user_id                     | 4c0c4007690a490f9325d27a10f6c03e                                                                     |
| volumes_attached            |                                                                                                      |
+-----------------------------+------------------------------------------------------------------------------------------------------+
```