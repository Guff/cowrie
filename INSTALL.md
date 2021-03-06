# Installing cowrie in six easy steps.

## Install with Python packages from your Linux Distribution

Install prerequisites on Debian based systems:

```
$ sudo apt-get install python-twisted python-crypto python-pyasn1 python-gmpy2 python-mysqldb python-zope.interface
```

Install prerequisites on RedHat based systems:

```
$ sudo yum install <tbd> <tbd> <tbd>
```

Install prerequisites on Alpine based systems:

```
$ sudo apk add python py-asn1 py-twisted py-zope-interface libffi-dev \
        py-cryptography py-pip py-six py-cffi py-idna py-ipaddress py-openssl
$ sudo pip install enum34
```

## Install with Python Virtual Environments

On Debian based systems:
```
$ sudo apt-get install virtualenv
```
On RedHat based systems:
```
$ sudo yum install virtualenv
```

Create a virtual environment

```
$ virtualenv v2
New python executable in ./cowrie/v2/bin/python
Installing setuptools, pip, wheel...done.
```

Activate the virtual environment and install packages

```
$ source v2/bin/activate
(v2) $ pip install twisted cryptography pyopenssl gmpy2
```

## Add a user

It's strongly recommended to install under a dedicated non-root user id:

```
$ sudo adduser --disabled-password cowrie
Adding user `cowrie' ...
Adding new group `cowrie' (1002) ...
Adding new user `cowrie' (1002) with group `cowrie' ...
Changing the user information for cowrie
Enter the new value, or press ENTER for the default
Full Name []:
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n]

$ sudo su - cowrie

$ git clone http://github.com/micheloosterhof/cowrie
Cloning into 'cowrie'...
remote: Counting objects: 2965, done.
remote: Compressing objects: 100% (1025/1025), done.
remote: Total 2965 (delta 1908), reused 2962 (delta 1905), pack-reused 0
Receiving objects: 100% (2965/2965), 3.41 MiB | 2.57 MiB/s, done.
Resolving deltas: 100% (1908/1908), done.
Checking connectivity... done.

$ cd cowrie

$ cp cowrie.cfg.dist cowrie.cfg

$ ./start.sh
Starting cowrie in the background...
```
When using Python Virtual Environments you can add the name of the venv as the first argument

```
$ ./start.sh v2
Starting cowrie in the background...
```

Cowrie runs by default on port 2222. This can be modified in the configuration file.
The following firewall rule will forward incoming traffic on port 22 to port 2222.

```
$ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222
```

Alternatively you can run authbind to listen as non-root on port 22 directly:

```
$ apt-get install authbind
$ touch /etc/authbind/byport/22
$ chown cowrie:cowrie /etc/authbind/byport/22
$ chmod 770 /etc/authbind/byport/22
```

* Edit start.sh and modify the AUTHBIND_ENABLED setting
* Change listen_port to 22 in cowrie.cfg

## Bugs and workarounds

* For some versions of Twisted you may receive the following error messagse:

```
....
  File "/usr/lib/python2.7/site-packages/Crypto/PublicKey/DSA.py", line 342, in _generate
      key = self._math.dsa_construct(obj.y, obj.g, obj.p, obj.q, obj.x)
      TypeError: must be long, not mpz
```

This is caused by Twisted incompatibilities. A workaround is to run:

```
$ cd cowrie/data
$ ssh-keygen -t dsa -b 1024 -f ssh_host_dsa_key
```

* Default file permissions

To make Cowrie logfiles public readable, change the ```--umask 0077``` option in start.sh into ```--umask 0022```

