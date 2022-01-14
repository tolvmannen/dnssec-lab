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
dig @127.0.0.1 labbX.examples.nu axfr
```

---
[Testing](testing.md)