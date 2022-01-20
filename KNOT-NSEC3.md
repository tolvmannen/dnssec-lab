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

6. Perform a zone transfer (AXFR) and verify the zone uses `NSEC`:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

7. Reload the new configuration:
```bash
sudo knotc reload
```

8. Perform a zone transfer (AXFR) again and verify the zone now has switched to `NSEC3`:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

---
[Testing](testing.md)