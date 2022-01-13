# Knot DNS setup.


#### Notes - Instructions written using

* Ubuntu 20.04
* Knot DNS, version 3.1.5

## Preparation

#### Change the servers hostname

1. To get rid of annoying error messages, add your hostname to the `hosts` file

```bash
sudo vi /etc/hosts
```

2. Add the following row, where Y.Y.Y.Y is your public IP address
```
Y.Y.Y.Y ns.labX.examples.nu
```

Note: If runnning on an AWS EC2, also add assigned hostname to /etc/hosts (one-liner for convenience)
```bash
echo $(hostname | sed s/'ip-'/''/ | sed s/-/./g | sed s/''$/' '/) $(hostname) > /tmp/hosts ; cat /etc/hosts >> /tmp/hosts ; sudo mv /tmp/hosts /etc/hosts
```


3. Change the hostname
```bash
sudo hostname ns.labX.examples.nu
```

4. Log out and back in to get an updated command prompt

#### Disable `systemd-resolved(8)` as it might interfer with Knot:

1. Disable and stop the service
```bash
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
```

2. Replace the symlink `/etc/resolv.conf` 
```bash
sudo rm /etc/resolv.conf
```

3. Add a (new) default system resolver.
```bash
sudo vi /etc/resolv.conf
```

```
nameserver 89.32.32.32
```


## Install Knot
```bash
sudo add-apt-repository ppa:cz.nic-labs/knot-dns-latest -y
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


1. Create zone file

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


2. Add configuration in /etc/knot/knot.conf

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

3. Add zone statement
```
zone:
  - domain: labX.examples.nu
    journal-content: all
    zonefile-load: difference-no-serial
    acl: acl_localhost
```

4. Save and exit

5. Check the configuration
```bash
sudo knotc conf-check
```

6. Verify that the zone can be loaded
```bash
sudo knotc zone-check labX.examples.nu
```

7. Reload Knot
```bash
sudo knotc reload
```

8. Verify that the server answers correctly
```bash
dig @127.0.0.1 labX.examples.nu soa
dig @127.0.0.1 labX.examples.nu ns
```


---
Next Section: [Knot DNSSEC lab](KNOT-dnssec.md)
