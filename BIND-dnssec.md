# BIND DNSSEC Lab

## Creating a Policy

We will create a KASP policy named "lab_p256". It uses ridiculously low values on the timing parameters, just so that key rollovers will go faster in this lab environment.

1. Open the knot configuration file:
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
    // purge-keys PT4H; // BIND does not recognise this option, although it's documented //

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
	Valid config yields no output

## Enable Zone Signing

In order to activate signing, configure the lab zone to use the policy `lab_p256`

1. Open the knot configuration file:
```bash
vi /etc/bind/named.conf.local
```

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

	If you are uncertain which as to file contains the KSK, you can either check key files: 
```bash
sudo grep keyid /var/cache/bind/KlabX.examples.nu*.key
```
```
/var/cache/bind/KlabX.examples.nu.+013+29254.key:; This is a zone-signing key, keyid 29254, for labX.examples.nu.
/var/cache/bind/KlabX.examples.nu.+013+40096.key:; This is a key-signing key, keyid 40096, for labX.examples.nu.
```

or get the ID from the zone:
```bash
dig @127.0.0.1 labX.examples.nu dnskey +multi
```
```
;; ANSWER SECTION:
labX.examples.nu.	300 IN DNSKEY 257 3 13 (
				4vDai6IfUNflFU8NnN2fWi2V6BNbxmbdB2cjNdJVnXny
				TgG2Qb1r4eD2IIBiakO4pJBEiWEo2oKMhlVT0Frk0w==
				) ; KSK; alg = ECDSAP256SHA256 ; key id = 40096
labX.examples.nu.	300 IN DNSKEY 256 3 13 (
				2mbnvVprgoVPQfkxyeRlJ/rmz2e1NnXvsBOhYAONOtmS
				1IkzjdR9iv3bylw7RM2I0ZfLYhZyoJdeLVXSz9ECEg==
				) ; ZSK; alg = ECDSAP256SHA256 ; key id = 29254
```

3. Ask your teacher to update the DS in the parent zone.

4. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labX.examples.nu DS
```

5. Query a validating resolver to verify that you get a signed response
```bash
dig @n1.1.1.1 labX.examples.nu SOA +dnssec
```


## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.

Our KASP policy is configured to not perform KSK rollovers automatically, but we can still request one manually:

1. Initiate a KSK rollover:
```bash
rndc dnssec -rollover -key 40096 labX.examples.nu
```



