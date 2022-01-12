#  BIND DNS setup.

#### Notes - Instructions written using

* Ubuntu 20.04
* BIND 9.16.24


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


## Install BIND9
```bash
sudo add-apt-repository ppa:isc/bind
```
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade -y
```
```bash
sudo apt install bind9 bind9-dnsutils -y
```

#### For troubleshooting purposes, you might also want to install some additional packages
```bash
sudo apt-get install mlocate net-tools -y
sudo updatedb
```

## Publish a zone

1. Create zone file
```bash
sudo vi /var/cache/bind/labX.examples.nu
```
```
$ORIGIN labX.examples.nu.
$TTL 120
@       SOA     ns.labX.examples.nu. dns.examples.nu. 1618586094 14400 3600 1814400 120

@       NS      ns
ns     A       192.0.2.1
```

2. Add configuration
```bash
sudo vi /etc/bind/named.conf.local
```
```
zone "labX.examples.nu" {
    type master;
    file "labX.examples.nu";
    allow-transfer { 127.0.0.1; };
};
```

3. Verify that the configuration is valid
```bash
named-checkconf
```
	Valid config yields no output

4. Verify that the zone can be loaded
```bash
named-checkzone labX.examples.nu /var/cache/bind/labX.examples.nu
```

5. Reload BIND
```bash
sudo service bind9 reload
```

6. Verify that the server answers correctly
```bash
dig @127.0.0.1 labX.examples.nu soa
dig @127.0.0.1 labX.examples.nu ns
```

---
Next Section: [BIND DNSSEC lab](BIND-dnssec.md)
