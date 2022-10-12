Notes for OpenStack on FreeBSD Project
======================================

Environment
-----------

- FreeBSD 13.1-RELEASE
- OpenStack Xena
- Hostname
    - `osf-keystone`
    - `osf-ironic`
    - `osf-nova`

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