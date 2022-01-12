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
  - domain: labX.examples.nu
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
dig @127.0.0.1 labX.examples.nu axfr
```

6. Reload the new configuration:
```bash
sudo knotc reload
```

7. Perform another zone transfer (AXFR) and verify the zone is now signed:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

    You can also look at the signed zone file on disk:
```bash
sudo cat /var/lib/knot/labX.examples.nu
```


8. Also check that DNSSEC records are correctly served for this zone:
```bash
dig @127.0.0.1 labX.examples.nu SOA +dnssec
```


## Publishing the DS RR

The zone is now signed and we have verified that DNSSEC is working. It is now time to publish the DS RR.

1. Wait until the KSK is ready to be published in the parent zone. This is indicated by the timestamp `ready` being non-zero:
```bash
sudo keymgr labX.examples.nu list
```

2. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labX.examples.nu ds
```
3. Ask your teacher to update the DS in the parent zone.

4. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```
5. As of now, we must manually tell the signer that the KSK has been submitted. After the signer knows the DS is in place at the parent, the initial key usage period will commence.
```bash
sudo knotc zone-ksk-submitted labX.examples.nu
```

6. Verify that we can query the zone from the *resolver* machine.
   The AD-flag should be set:
```bash
dig @127.0.0.1 +dnssec www.labX.examples.nu
```


## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.

Our KASP policy is configured to not perform KSK rollovers automatically, but we can still request one manually:

1. Initiate a KSK rollover:
```bash
sudo knotc zone-key-rollover labX.examples.nu ksk
```
2. Check that the new KSK has been generated:
```bash
sudo keymgr labX.examples.nu list
```

3. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labX.examples.nu ds
```
4. Ask your teacher to update the DS in the parent zone.

5. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```
6. As of now, we must manually tell the signer that the KSK has been submitted. 

```bash
sudo knotc zone-ksk-submitted labX.examples.nu
```
    If the KSK is not yet ready to be submitted, you must wait a bit and try again later.
    
7. After the KSK has been submitted, check the key list and note that the old KSK has been removed.
```bash
sudo keymgr labX.examples.nu list
```

8. Ask your teacher to remove the old DS from the parent zone.

9. Verify that the old DS has been removed
```bash
dig @ns1.examples.nu labX.examples.nu DS
```


## Signing with NSEC3

1. Open the knot configuration file:
```bash
sudo vi /etc/knot/knot.conf
```
2. Update the KASP policy to sign with NSEC3:
```
policy:
  - id: lab_p256
    nsec3: true
    nsec3-iterations: 0
```
   Guidance on recommended NSEC3 parameter settings can be found in [draft-hardaker-dnsop-nsec3-guidance-03](https://datatracker.ietf.org/doc/html/draft-hardaker-dnsop-nsec3-guidance-03). The default number of iterations in Knot is 10, but we choose to use the newer recommendation (0) from the draft.

4. Save and exit.

5. Verify that the configuration is valid:
```bash
sudo knotc conf-check
```
6. Reload the new configuration:
```bash
sudo knotc reload
```

7. Perform a zone transfer (AXFR) and verify the zone is now signed with NSEC3:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

## Algorithm Rollover

Rolling the algorithm will by necessity also roll both KSK and ZSK. During the rollover all RRs will be signed by BOTH keys.

1. Open the BIND configuration file:
```bash
sudo vi /etc/knot/knot.conf
```

2. Edit the DNSSEC signing policy and change algorithm for the KSK and ZSK (both must use the same algorithm)

```
policy:
  - id: lab_p256
    algorithm: RSASHA256
    ...
```

3. Save and exit

4. Verify that the configuration is valid
```bash
sudo knotc conf-check
```

5. Reload Knot
```bash
sudo knotc reload
```

6. Check that the new KSK has been generated and is ready to be published
```bash
sudo keymgr labX.examples.nu list
```

7. Perform a zone transfer (AXFR) and note that the whole zone is now signed with *double signatures*:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

Knot will automatically phase out the old keys and signatures as it resigns the zone


8. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labX.examples.nu ds
```

9. Ask your teacher to update the DS in the parent zone.

10. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```
11. We must manually tell the signer that the KSK has been submitted. 
```bash
sudo knotc zone-ksk-submitted labX.examples.nu
```
    If the KSK is not yet ready to be submitted, you must wait a bit and try again later.
    
12. After the KSK has been submitted, wait for Knot to replace the keys and signatures. Check the key list and note that the old KSK and ZSK has been removed. 
```bash
sudo keymgr labX.examples.nu list
```

```bash
dig @127.0.0.1 labX.examples.nu axfr
```

13. Ask your teacher to remove the old DS from the parent zone.

14. Verify that the old DS has been removed
```bash
dig @ns1.examples.nu labX.examples.nu DS
```


---
Next Section: [Testing](testing.md)
