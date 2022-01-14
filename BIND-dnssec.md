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

> Note: Valid config yields no output

2. Add the policy to the zone statement

```
zone "labbX.examples.nu" {
    type master;
    file "labbX.examples.nu";
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
dig @127.0.0.1 labbX.examples.nu axfr
```

7. Reload BIND
```bash
sudo rndc reload
```

8. Perform another zone transfer (AXFR) and verify the zone is now signed:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

9. Also check that DNSSEC records are correctly served for this zone:
```bash
dig @127.0.0.1 labbX.examples.nu SOA +dnssec
```

## Publishing the DS RR

The zone is now signed and we have verified that DNSSEC is working. It is now time to publish the DS RR.

1. Wait until the KSK is ready to be published in the parent zone.

2. Use the KSK key file to generate a DS record
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabbX.examples.nu.+013+40096.key
```

> Note: If you are uncertain which as to file contains the KSK, you can either check the key status to get the key ID: 

```bash
sudo rndc dnssec -status labbX.examples.nu
```

or get the ID from the dnskeys in the zone:

```bash
dig @127.0.0.1 labbX.examples.nu dnskey +multi
```
	
> Note:  You have to use the flag +multi for dig to print the additional key information (KSK/ZSK and key ID)

3. Ask your teacher to update the DS in the parent zone.

4. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

5. Query a validating resolver to verify that you get a signed response
```bash
dig @1.1.1.1 labbX.examples.nu SOA +dnssec
```

---
Next Section: [Manual KSK Rollover](BIND-Manual-KSK-Rollover.md)

[Testing](testing.md)




