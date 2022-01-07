# Knot DNS setup.


#### Notes - Instructions written using

* Ubuntu 20.04
* Knot DNS, version 3.1.5


## Install Knot
```bash
sudo add-apt-repository ppa:cz.nic-labs/knot-dns-latest
```
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade
```
```bash
sudo apt-get install knot knot-dnsutils -y
```


#### Create zone file under /var/lib/knot/
```bash
vi /var/lib/knot/labX.examples.nu
```

Example:
```
$ORIGIN labX.examples.nu.
$TTL 120
@       SOA     ns.labX.examples.nu. hostmaster.examples.nu. 1618586094 14400 3600 1814400 120

@       NS      ns.labX.examples.nu.
ns      A       192.0.2.1
```


#### Add configuration in /etc/knot/knot.conf


```
server:
    rundir: "/run/knot"
    user: knot:knot
    listen: [ 0.0.0.0@53, ::@53 ]

log:
  - target: syslog
    any: info

database:
    storage: "/var/lib/knot"

acl:
  - id: acl_localhost
    address: 127.0.0.1
    action: transfer

template:
  - id: default
    storage: "/var/lib/knot"
    file: "%s"
```

Add zone statement
```
zone:
  - domain: labX.examples.nu
    journal-content: all
    zonefile-load: difference-no-serial
    acl: acl_localhost
```


## Optional

Generate a TSIG key. It will be used to limit access for AXFR and dynamic updates.
```
keymgr -t labX-tsig
```

Copy key to config.
NOTE: The key is is only written to stdout, not to a file. 
```
key:
  - id: labX-tsig
    algorithm: hmac-sha256
    secret: kygZePBiwMfLgcSv8DosScmXIo5e9P7nx/fKaNsaR7c=
```

Update ACL statement for limiting nsupdate/axfr. 
```
acl:
   - id: acl_localhost
     key: labX-tsig
     action: [ transfer ]
```


