# Knot DNS setup.


#### Notes - Instructions written using

* Ubuntu 20.04
* Knot DNS, version 3.1.5


## Change the servers hostname

1. To get rid of annoying error messages, add your hostname to the `hosts` file

```bash
sudo vi /etc/hosts
```

Add the following row, where Y.Y.Y.Y is your public IP address
```
Y.Y.Y.Y ns.labX.examples.nu
```

2. Change the hostname
```bash
sudo hostname ns.labX.examples.nu
```

3. Log out and back in to get an updated command prompt


## Install Knot
```bash
sudo add-apt-repository ppa:cz.nic-labs/knot-dns-latest
```
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade -y
```
```bash
sudo apt-get install knot knot-dnsutils -y
```

#### For troubleshooting purposes, you might also want to install some additional packages
```bash
sudo apt-get install mlocate net-tools -y
sudo updatedb
```

## Publlish a zone 


#### Create zone file under /var/lib/knot/
```bash
sudo vi /var/lib/knot/labX.examples.nu
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

```bash
sudo vi /etc/knot/knot.conf
```

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

Save and exit

Check the configuration
```bash
sudo knotc conf-check
```


Disable `systemd-resolved(8)` as it might interfer with Knot:
```bash
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

Replace the symlink `/etc/resolv.conf` 
```bash
sudo rm /etc/resolv.conf
```
```bash
sudo vi /etc/resolv.conf
```
Add a default system resolver
```
nameserver 9.9.9.9
```


## Optional

Generate a TSIG key. It will be used to limit access for AXFR and dynamic updates.
```bash
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

---
Next Section: [Knot DNSSEC lab](KNOT-dnssec.md)
