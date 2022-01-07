#  BIND DNS setup.

#### Notes - Instructions written using

* Ubuntu 20.04
* BIND 9.16.1-Ubuntu (Stable Release) <id:d497c32>


## Install BIND9
```bash
sudo apt-get update
```
```bash
sudo apt-get upgrade -y
```
```bash
sudo apt install bind9 bind9-dnsutils -y
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

6. Check that the server answers correctly
```bash
dig @127.0.0.1 labX.examples.nu soa
dig @127.0.0.1 labX.examples.nu ns
```

---
Next Section: [BIND DNSSEC lab](BIND-dnssec.md)