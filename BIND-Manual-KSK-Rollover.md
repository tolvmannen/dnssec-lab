## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.


1. Check the status of your keys:
```bash
sudo rndc dnssec -status labX.examples.nu
```

2. Initiate a KSK rollover:
```bash
sudo rndc dnssec -rollover -key 40096 labX.examples.nu
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

12. Wait for BIND to remove the old key from the zone. This should only take a few minutes. You can check periodically with:
```bash
sudo rndc dnssec -status labX.examples.nu
```
or
```bash
dig @127.0.0.1 labX.examples.nu dnskey +multi
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

 Note: Guidance on recommended NSEC3 parameter settings can be found in [draft-hardaker-dnsop-nsec3-guidance-03](https://datatracker.ietf.org/doc/html/draft-hardaker-dnsop-nsec3-guidance-03). The default number of iterations in Knot is 10, but we choose to use the newer recommendation (0) from the draft.


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

---
Next Section: [Algorithm Rollover](BIND-Algorithm-Rollover.md)

[Testing](testing.md)