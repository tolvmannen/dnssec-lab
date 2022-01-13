# BIND DNSSEC Lab

## Creating a Policy

We will create a KASP policy named "lab_p256". It uses ridiculously low values on the timing parameters, just so that key rollovers will go faster in this lab environment.

1. Open the BIND configuration file:
```bash
sudo vi /etc/bind/named.conf.local
```

2. Define a DNSSEC signing policy (KASP)

```
dnssec-policy "lab_p256" {
    keys {
         ksk lifetime unlimited algorithm ecdsa256;
         zsk lifetime PT30M algorithm ecdsa256;
    };

    // Key timings
    dnskey-ttl PT5M;
    publish-safety PT1M;
    retire-safety PT1M;
    purge-keys PT2H;

    // Signature timings
    signatures-refresh PT5M;
    signatures-validity PT15M;
    signatures-validity-dnskey PT15M;
    
    // Zone parameters
    max-zone-ttl PT5M;
    zone-propagation-delay PT5M;
    parent-ds-ttl PT2M;
    parent-propagation-delay 0;
};
```

3. Save and exit

4. Verify that the configuration is valid
```bash
named-checkconf
```


## Enable Zone Signing

In order to activate signing, configure the lab zone to use the policy `lab_p256`

1. Open the knot configuration file:
```bash
vi /etc/bind/named.conf.local
```

Note: Valid config yields no output

2. Add the policy to the zone statement

```
zone "labX.examples.nu" {
    type master;
    file "labX.examples.nu";
    allow-transfer { 127.0.0.1; };
    dnssec-policy lab_p256;
};
```

4. Save and exit

5. Verify that the configuration is valid
```bash
named-checkconf
```
	Valid config yields no output

6. Perform a zone transfer (AXFR) and verify the zone is not yet signed:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

7. Reload BIND
```bash
sudo rndc reload
```

8. Perform another zone transfer (AXFR) and verify the zone is now signed:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

9. Also check that DNSSEC records are correctly served for this zone:
```bash
dig @127.0.0.1 labX.examples.nu SOA +dnssec
```

## Publishing the DS RR

The zone is now signed and we have verified that DNSSEC is working. It is now time to publish the DS RR.

1. Wait until the KSK is ready to be published in the parent zone.

2. Use the KSK key file to generate a DS record
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabX.examples.nu.+013+40096.key
```

Note: If you are uncertain which as to file contains the KSK, you can either check the key status to get the key ID: 

```bash
sudo rndc dnssec -status labX.examples.nu
```

or get the ID from the dnskeys in the zone:

```bash
dig @127.0.0.1 labX.examples.nu dnskey +multi
```
	
Note that you have to use the flag +multi for dig to print the additional key information (KSK/ZSK and key ID)

3. Ask your teacher to update the DS in the parent zone.

4. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

5. Query a validating resolver to verify that you get a signed response
```bash
dig @n1.1.1.1 labX.examples.nu SOA +dnssec
```

To check the status of your keys, ues:
```bash
sudo rndc dnssec -status labX.examples.nu
```


## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.

Our KASP policy is configured to not perform KSK rollovers automatically, but we can still request one manually:

1. Check the status of your keys:
```bash
sudo rndc dnssec -status labX.examples.nu
```

2. Initiate a KSK rollover:
```bash
rndc dnssec -rollover -key 40096 labX.examples.nu
```

3. Check the status of your keys again, to see that a new KSK has been generated:
```bash
sudo rndc dnssec -status labX.examples.nu
```

4. Verify that the dnskey record is published:
```bash
dig @127.0.0.1 labX.examples.nu dnskey +multi
```

5. generate a DS record for the new key
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabX.examples.nu.+013+38587.key
```

6. Ask your teacher to update the DS in the parent zone.

7. Wait until the new DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

8. Tell BIND that the new DS is published in the parent zone:
```bash
sudo rndc dnssec -checkds -key 38587 published labX.examples.nu
```

9. Ask your teacher to remove the old the DS in the parent zone.

10. Wait until the old DS has been removed. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

11. Tell BIND that the old DS has been removed from the parent zone:
```bash
sudo rndc dnssec -checkds -key 40096 withdrawn labX.examples.nu
```



## Signing with NSEC3

1. Open the BIND configuration file:
```bash
sudo vi /etc/bind/named.conf.local
```

2. Edit the DNSSEC signing policy and add parameters for NSEC3

```
dnssec-policy "lab_p256" {
	...
    nsec3param iterations 0 optout no salt-length 8;
    ...
};
```

 	Guidance on recommended NSEC3 parameter settings can be found in [draft-hardaker-dnsop-nsec3-guidance-03](https://datatracker.ietf.org/doc/html/draft-hardaker-dnsop-nsec3-guidance-03). The default number of iterations in Knot is 10, but we choose to use the newer recommendation (0) from the draft.


3. Save and exit

4. Verify that the configuration is valid
```bash
named-checkconf
```

5. Reload BIND
```bash
sudo rndc reload
```

6. Perform a zone transfer (AXFR) and verify the zone now uses NSEC3:
```bash
dig @127.0.0.1 labX.examples.nu axfr
```

## Algorithm Rollover

Rolling the algorithm will by necessity also roll both KSK and ZSK. During the rollover all RRs will be signed by BOTH keys.

1. Open the BIND configuration file:
```bash
sudo vi /etc/bind/named.conf.local
```

2. Edit the DNSSEC signing policy and change algorithm for the KSK and ZSK (both must use the same algorithm)

```
dnssec-policy "lab_p256" {
    keys {
         ksk lifetime unlimited algorithm rsasha256;
         zsk lifetime PT30M algorithm rsasha256;
    };
    ...
};
```

3. Save and exit

4. Verify that the configuration is valid
```bash
named-checkconf
```

5. Reload BIND
```bash
sudo rndc reload
```

6. Verify with dig that new KSK and ZSK has been added to the zone 

```bash
dig @127.0.0.1 labX.examples.nu dnskey +multi
```

7. Generate a DS record for the new KSK 
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabXC.examples.nu.+008+18391.key
```

8. Ask your teacher to update the DS in the parent zone.

9. Wait until the new DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

10. Tell BIND that the new DS is published in the parent zone:
```bash
sudo rndc dnssec -checkds -key 18391 published labX.examples.nu
```

11. Ask your teacher to remove the old the DS in the parent zone.

12. Wait until the new DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

13. Tell BIND that the old DS has been removed from the parent zone:
```bash
sudo rndc dnssec -checkds -key 38587 withdrawn labX.examples.nu
```

---
Next Section: [Testing](testing.md)
