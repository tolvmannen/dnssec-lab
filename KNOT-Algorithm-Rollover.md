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
sudo keymgr labbX.examples.nu list
```

7. Perform a zone transfer (AXFR) and note that the whole zone is now signed with *double signatures*:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

Knot will automatically phase out the old keys and signatures as it resigns the zone


8. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labbX.examples.nu ds
```

9. Ask your teacher to update the DS in the parent zone.

10. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```
11. We must manually tell the signer that the KSK has been submitted. 
```bash
sudo knotc zone-ksk-submitted labbX.examples.nu
```
    If the KSK is not yet ready to be submitted, you must wait a bit and try again later.
    
12. After the KSK has been submitted, wait for Knot to replace the keys and signatures. Check the key list and note that the old KSK and ZSK has been removed. 
```bash
sudo keymgr labbX.examples.nu list
```

```bash
dig @127.0.0.1 labbX.examples.nu axfr
```

13. Ask your teacher to remove the old DS from the parent zone.

14. Verify that the old DS has been removed
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

---
Next Section: [Signing with NSEC3](KNOT-NSEC3.md)

[Testing](testing.md)