Keystone
========

Prerequisites
-------------

```bash=
sudo pkg install python
```

Get the source of Keystone and checkout to branch `stable/xena`.

```bash=
git clone https://github.com/openstack/keystone.git
cd keystone
git checkout origin/stable/xena -b stable/xena
```

Then create virtual environment for testing:

```bash=
python3.8 -m venv .venv
source .venv/bin/activate
```

Upgrade `pip` to the latest version:

```bash=
python -m pip install --upgrade pip wheel
```

Rust is a build-time dependency, so install it using `pkg`.

```bash=
sudo pkg install rust
```

Build and Install Keystone Package
----------------------------------

It's time to install the dependencies and Keystone itself:

```bash=
pip install .
```

Config Generation
-----------------

> ~~NOTE: This section is not working as of now.~~
> Issue fixed. (2022/02/10)

```bash=
sudo pkg install libxml2 libxslt postgresql14-client
pip install lxml psycopg2
```

```bash=
pip install tox
tox -egenconfig
tox -egenpolicy
```

- https://mail.python.org/pipermail/python-ldap/2015q2/003546.html
- https://www.python-ldap.org/en/python-ldap-3.3.0/installing.html#setup-cfg

Could not install dependency `python-ldap`, error while building:

```
  ...
    cc -pthread -Wno-unused-result -Wsign-compare -Wunreachable-code -DNDEBUG -O2 -pipe -fstack-protector-strong -fno-strict-aliasing -fPIC -DHAVE_SASL -DHAVE_TLS -DHAVE_LIBLDAP_R -DHAVE_LIBLDAP_R -DLDAPMODULE_VERSION=3.3.1 -DLDAPMODULE_AUTHOR=python-ldap project -DLDAPMODULE_LICENSE=Python style -IModules -I/usr/home/imp/keystone/.tox/pep8/include -I/usr/local/include/python3.8 -c Modules/LDAPObject.c -o build/temp.freebsd-13.0-RELEASE-amd64-3.8/Modules/LDAPObject.o
    In file included from Modules/LDAPObject.c:3:
    Modules/common.h:15:10: fatal error: 'lber.h' file not found
    #include <lber.h>
             ^~~~~~~~
    1 error generated.
    error: command 'cc' failed with exit status 1
  ...
```

Specifying library and header directories in `setup.cfg` of `python-ldap` would work:

```bash=
library_dirs = /usr/local/lib
include_dirs = /usr/local/include
```

However, this isn't what we want because we have to download the sources and modify the configuration. Another way to do this is to prepend environment variables while installing `python-ldap` using `pip`:

```bash=
LDFLAGS=-L/usr/local/lib CPPFLAGS=-I/usr/local/include pip install python-ldap
```

Next step is to pass these envs to `tox`:

```bash=
LDFLAGS=-L/usr/local/lib CPPFLAGS=-I/usr/local/include tox -egenconfig
```

Phew, problem solved!

Database
--------

Setup MySQL server.

```bash=
sudo pkg install mysql80-server
sudo sysrc mysql_enable=yes
sudo service mysql-server start
sudo mysqladmin -u root password 'password'
```

Create essential database and users with rightful privileges for Keystone.

```sql=
CREATE DATABASE keystone;
CREATE USER 'keystone'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost';
CREATE USER 'keystone'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%';
FLUSH PRIVILEGES;
```

System User
-----------

Create a dedicated user `keystone` and group `keystone` for Keystone service.

```bash=
sudo pw user add -n keystone -c 'OpenStack Identity Service' -d /nonexistent -s /usr/sbin/nologin
```

Configuration
-------------

~~Since we're still unable to generate template config via `tox -egenconfig`, download `keystone.conf` sample file directly from [here](https://docs.openstack.org/keystone/xena/configuration/samples/keystone-conf.html) and put it under `etc/` of Keystone project's directory. Modify the essential fields:~~

The sample configuration file is under `etc/` directory of Keystone project as we generated it in the previous section. Modify essential fields:

```plain
[DEFAULT]

[application_credential]

[assignment]

[auth]

[cache]

[catalog]

[cors]

[credential]
key_repository = /usr/local/etc/keystone/credential-keys/

[database]
connection = mysql+pymysql://keystone:password@localhost/keystone

[domain_config]

[endpoint_filter]

[endpoint_policy]

[eventlet_server]

[federation]
sso_callback_template = /usr/local/etc/keystone/sso_callback_template.html

[fernet_receipts]
key_repository = /usr/local/etc/keystone/fernet-keys/

[fernet_tokens]
key_repository = /usr/local/etc/keystone/fernet-keys/

[healthcheck]

[identity]
domain_config_dir = /usr/local/etc/keystone/domains

[identity_mapping]

[jwt_tokens]
jws_public_key_repository = /usr/local/etc/keystone/jws-keys/public
jws_private_key_repository = /usr/local/etc/keystone/jws-keys/private

[ldap]

[memcache]

[oauth1]

[oslo_messaging_amqp]

[oslo_messaging_kafka]

[oslo_messaging_notifications]

[oslo_messaging_rabbit]

[oslo_middleware]

[oslo_policy]

[policy]

[profiler]

[receipt]

[resource]

[revoke]

[role]

[saml]
certfile = /usr/local/etc/keystone/ssl/certs/signing_cert.pem
keyfile = /usr/local/etc/keystone/ssl/private/signing_key.pem
idp_metadata_path = /usr/local/etc/keystone/saml2_idp_metadata.xml

[security_compliance]

[shadow_users]

[token]

provider = fernet

[tokenless_auth]

[totp]

[trust]

[unified_limit]

[wsgi]
```

Then populate the Identity service database. But before that you have to install `pymysql` first.

```bash=
pip install pymysql
```

```bash=
keystone-manage --config-file etc/keystone.conf db_sync
```

Initialize fernet keys:

```bash=
sudo keystone-manage --config-file etc/keystone.conf fernet_setup \
  --keystone-user keystone \
  --keystone-group keystone
sudo keystone-manage --config-file etc/keystone.conf credential_setup \
  --keystone-user keystone \
  --keystone-group keystone
```

Bootstrap the identity service. The string provided with `--bootstrap-password` is administrator's password.

```bash=
sudo keystone-manage --config-file etc/keystone.conf bootstrap \
  --bootstrap-password password \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

Kickstart Service
-----------------

For production environment, please use Apache web server. But for now we use `uwsgi` for convenience.

```bash=
pip install uwsgi
```

```bash=
sudo uwsgi --http 0.0.0.0:5000 --wsgi-file $(which keystone-wsgi-public)
```

Verify Installation
-------------------

Install `python-openstackclient`

```bash=
pip install python-openstackclient
```

Provide API endpoint and credentials for OpenStack client tool:

```bash=
export OS_USERNAME=admin
export OS_PASSWORD=password
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_ID=default
export OS_PROJECT_DOMAIN_ID=default
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_URL=http://controller:5000/v3
```

Test if our newly setup Keystone is working or not by listing users on it.

```bash=
$ openstack user list
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| 56ce21af0ee8499f9021d0c981e5dec8 | admin |
+----------------------------------+-------+
```

### Feeding Sample Data (Optional)

For a more realistic environment, you can feed some fake data to Keystone:

```bash=
ADMIN_PASSWORD=password SERVICE_PASSWORD=password tools/sample_data.sh
```

Now that you can check `user`, `project`, etc.

```bash=
$ openstack user list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| cf724ed093a44658afc9ce82450a8131 | admin   |
| a700e206d08c47d1bc5107e288d87f31 | glance  |
| a66e0dfe94ef465ba584891145478b75 | nova    |
| 8d4da32f26174067b5717eb382004387 | cinder  |
| 56d63f2531a94cb8aaf80b15931d1b0c | swift   |
| 01776042ccde4823ba3fcbab7d687cd5 | neutron |
+----------------------------------+---------+
```

Testing
-------

### Unit/Functional Tests

Since Sqlite will be used during functional tests, we need to install dependencies first:

```bash=
sudo pkg install py38-sqlite3
```

```bash=
$ LDFLAGS=-L/usr/local/lib CPPFLAGS=-I/usr/local/include tox -epy37
...
<redacted>
...
======
Totals
======
Ran: 5874 tests in 701.6174 sec.
 - Passed: 4960
 - Skipped: 914
 - Expected Fail: 0
 - Unexpected Success: 0
 - Failed: 0
Sum of execute time for each test: 2550.9768 sec.

==============
Worker Balance
==============
 - Worker 0 (1267 tests) => 0:09:25.175616
 - Worker 1 (1429 tests) => 0:10:25.164709
 - Worker 2 (1589 tests) => 0:11:41.000346
 - Worker 3 (1589 tests) => 0:11:10.273038
_______________________________________________________ summary ________________________________________________________
  py37: commands succeeded
  congratulations :)
```

```bash=
$ LDFLAGS=-L/usr/local/lib CPPFLAGS=-I/usr/local/include tox -epep8
...
<redacted>
...
Run started:2022-02-10 08:51:56.001641

Test results:
        No issues identified.

Code scanned:
        Total lines of code: 41441
        Total lines skipped (#nosec): 74

Run metrics:
        Total issues (by severity):
                Undefined: 0.0
                Low: 0.0
                Medium: 0.0
                High: 0.0
        Total issues (by confidence):
                Undefined: 0.0
                Low: 0.0
                Medium: 0.0
                High: 0.0
Files skipped (0):
_______________________________________________________ summary ________________________________________________________
  pep8: commands succeeded
  congratulations :)

```

Go into Production
------------------

```bash=
sudo pkg install apache24
sudo sysrc apache24_enable=YES
```

Uncomment following two lines in `/usr/local/etc/apache24/httpd.conf` in order to enable uWSGI proxy module for Apache HTTP server:

```
LoadModule proxy_module libexec/apache24/mod_proxy.so
LoadModule proxy_uwsgi_module libexec/apache24/mod_proxy_uwsgi.so
```

Copy the Apache configuration file for Keystone to the location where Apache will read from and start the HTTP server.

```bash=
sudo cp httpd/uwsgi-keystone.conf /usr/local/etc/apache24/Includes/
sudo service apache24 start
```

Make necessary changes to both `httpd/keystone-uwsgi-admin.ini` and `httpd/keystone-uwsgi-public.ini`:

```
wsgi-file = /usr/home/<username>/keystone/.venv/bin/keystone-wsgi-public
```

Start two Kestone uWSGI applications, one for admin, the other for public:

```bash=
sudo uwsgi httpd/keystone-uwsgi-admin.ini
sudo uwsgi httpd/keystone-uwsgi-public.ini
```
