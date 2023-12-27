# Documentation for The "OpenStack on FreeBSD" Project

The documentation provides in-depth guides for users to successfully deploy and operate OpenStack on FreeBSD. Covering installation, configuration, and troubleshooting, the documentation ensures a set of essential OpenStack components is up and running with basic functionality.

## Environment

- FreeBSD 14.0-STABLE
- Based on OpenStack Xena, specifically on the `stable/xena` branch of each projects
- Single-node all-in-one deployment

## Components

Components that are ported to FreeBSD and can be run with a limited feature set by applying patches and workarounds:

- [Keystone](keystone/README.md)
- [Glance](glance/README.md)
- [Placement](placement/README.md)
- [Neutron](neutron/README.md)
- [Nova](nova/README.md)

WIP:

- [Ironic](README.md)

## Future Works

### Neutron

- Implement the mechanism driver that utilizes FreeBSD native bridges
- Replace Linux namespaces and virtual ethernet with FreeBSD's `vnet` and `epair`
- Integrate `dnsmasq`

### Nova

- Implement a new `virt_type`: `bhyve`, for the `libvirt` compute driver
- Get rid of custom implementation - `socat-manager`

### Oslo

- Modify oslo.privsep to support FreeBSD's privilege model

### Misc

- Fixed version, do not rely on branches
- Use `--config-dir` instead of `--config-file`
- Move `var/lib`, `var/log`, `var/lock` directories under root (`/`)

## References

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