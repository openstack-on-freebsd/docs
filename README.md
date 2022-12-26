Notes for OpenStack on FreeBSD Project
======================================

Environment
-----------

- FreeBSD 13.1-RELEASE
- OpenStack Xena
- `/etc/hosts`
    ```
    192.168.88.243 osf-keystone
    192.168.88.244 osf-ironic
    192.168.88.245 osf-nova
    192.168.88.246 osf-glance
    192.168.88.249 osf-neutron
    ```

Components
----------

- [keystone](keystone.md)
- [ironic](ironic.md)
- [glance](glance.md)
- [placement](placement.md)
- [neutron](neutron.md)
- [nova](nova.md)

TODOs
-----

- Use `--config-dir` instead of `--config-file`
- Move "Patching" section before "Building" section
- Run services as non-root users
- Move `var/lib`, `var/log`, `var/lock` directories under root (`/`)
- New `virt_type`: `bhyve`
- Fixed version, do not rely on branches
- Add `nova-novncproxy`

References
----------

### Previous Works

- [OpenStack on FreeBSD/Xen Proof of Concept](http://empt1e.blogspot.com/2015/06/openstack-on-freebsdxen-proof-of-concept.html)

### Official Documentation

- [OpenStack Keystone - Install and configure](https://docs.openstack.org/keystone/xena/install/keystone-install-ubuntu.html#keystone-install-configure-ubuntu)
- [Setting Up Keystone](https://docs.openstack.org/keystone/latest/contributor/set-up-keystone.html)
- [OpenStack Keystone - Running tests](https://docs.openstack.org/keystone/latest/contributor/testing-keystone.html#)
- [Running Keystone in HTTPD](https://docs.openstack.org/keystone/ocata/apache-httpd.html)
- [OpenStack Ironic - Developer Quick-Start](https://docs.openstack.org/ironic/latest/contributor/dev-quickstart.html)

### Problem Solving

- [pg_config executable not found](https://stackoverflow.com/questions/11618898/pg-config-executable-not-found)
- [Installing python-ldap fails with lber.h file not found in ubuntu 17.10 even after installing devel packages](https://stackoverflow.com/questions/56506294/installing-python-ldap-fails-with-lber-h-file-not-found-in-ubuntu-17-10-even-aft)
- [python pip specify a library directory and an include directory](https://stackoverflow.com/questions/18783390/python-pip-specify-a-library-directory-and-an-include-directory)
- [tox configuration specification](https://tox.wiki/en/latest/config.html)
- [python-ldap does not support OpenLDAP 2.5+ yet](https://github.com/python-ldap/python-ldap/issues/445#issuecomment-983513451)
- [error in suds-jurko setup command: use_2to3 is invalid](https://github.com/andersinno/suds-jurko/issues/6)
- [libvirt and bhyve](https://people.freebsd.org/~rodrigc/libvirt/libvirt-bhyve.html)