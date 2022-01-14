# Knot DNSSEC Lab

## Creating a Policy

We will create a KASP policy named "lab_p256". It uses ridiculously low values on the timing parameters, just so that key rollovers will go faster in this lab environment.

1. Open the knot configuration file:
```bash
sudo vi /etc/knot/knot.conf
```

2. First we define a keystore "default" using the "pem" backend. This will keep a keys on disk.
```
keystore:
  - id: default
    backend: pem
```

3. Now we will define a DNSSEC signing policy (KASP):
```
policy:
  - id: lab_p256
    algorithm: ECDSAP256SHA256
    ksk-lifetime: 0
    zsk-lifetime: 30m
    keystore: default
    dnskey-ttl: 300
    rrsig-lifetime: 15m
    rrsig-refresh: 5m
    rrsig-pre-refresh: 1m
    propagation-delay: 0
```

4. Save and exit.


5. Verify that the configuration is valid:
```bash
sudo knotc conf-check
```

6. Reload the new configuration:
```bash
sudo knotc reload
```

## Create Template for Signed Zones

Before we can enable zone signing, we will create a new zone template that refers to our recently defined KASP policy.

1. Open the knot configuration file:
```bash
vi /etc/knot/knot.conf
```

2. Add the following entry to the `template` section:
```
template:
 - id: signed
   storage: "/var/lib/knot"
   file: "%s"
   serial-policy: unixtime
   journal-content: all
   zonefile-load: difference-no-serial
   semantic-checks: true
   dnssec-signing: on
   dnssec-policy: lab_p256
```

3. Save and exit.

4. Verify that the configuration is valid:
```bash
sudo knotc conf-check
```

5. Reload the new configuration:
```bash
sudo knotc reload
```

## Enable Zone Signing

In order to activate signing, configure the lab zone to use the `signed` template:

1. Open the knot configuration file:
```bash
vi /etc/knot/knot.conf
```

2. Add the template to the zone:
```
zone:
  - domain: labbX.examples.nu
    template: signed
    acl: [acl_localhost]
    ...
```

3. Save and exit.

4. Verify that the configuration is valid:
```bash
sudo knotc conf-check
```

5. Perform a zone transfer (AXFR) and verify the zone is not yet signed:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

6. Reload the new configuration:
```bash
sudo knotc reload
```

7. Perform another zone transfer (AXFR) and verify the zone is now signed:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

    You can also look at the signed zone file on disk:
```bash
sudo cat /var/lib/knot/labbX.examples.nu
```


8. Also check that DNSSEC records are correctly served for this zone:
```bash
dig @127.0.0.1 labbX.examples.nu SOA +dnssec
```


## Publishing the DS RR

The zone is now signed and we have verified that DNSSEC is working. It is now time to publish the DS RR.

1. Wait until the KSK is ready to be published in the parent zone. This is indicated by the timestamp `ready` being non-zero:
```bash
sudo keymgr labbX.examples.nu list
```

2. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labbX.examples.nu ds
```
3. Ask your teacher to update the DS in the parent zone.

4. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```
5. As of now, we must manually tell the signer that the KSK has been submitted. After the signer knows the DS is in place at the parent, the initial key usage period will commence.
```bash
sudo knotc zone-ksk-submitted labbX.examples.nu
```

6. Verify that we can query the zone from the *resolver* machine.
   The AD-flag should be set:
```bash
dig @127.0.0.1 +dnssec www.labbX.examples.nu
```



---
Next Section: [Manual KSK Rollover](KNOT-Manual-KSK-Rollover.md)

[Testing](testing.md)
