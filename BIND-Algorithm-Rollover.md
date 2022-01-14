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

6. Check the status of your keys, to see that a new KSK and a new ZSK with the new algorithm has been generated:


7. Verify with dig that new KSK and ZSK has been added to the zone 

```bash
dig @127.0.0.1 labbX.examples.nu dnskey +multi
```


8. Perform a zone transfer (AXFR) and note that the whole zone is now signed with *double signatures*:
```bash
dig @127.0.0.1 labbX.examples.nu axfr
```


9. Generate a DS record for the new KSK 
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabbX.examples.nu.+008+18391.key
```


10. Ask your teacher to update the DS in the parent zone.


11. Wait until the new DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```


12. Tell BIND that the new DS is published in the parent zone:
```bash
sudo rndc dnssec -checkds -key 18391 published labbX.examples.nu
```


13. Wait for BIND to phasse out the old key and signatures. This may take a few minutes. You can check periodically with:
```bash
sudo rndc dnssec -status labbX.examples.nu
```
or
```bash
dig @127.0.0.1 labbX.examples.nu dnskey +multi
```


14. Ask your teacher to remove the old the DS in the parent zone.


15. Wait until the new DS has been removed. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

16. Tell BIND that the old DS has been removed from the parent zone:
```bash
sudo rndc dnssec -checkds -key 38587 withdrawn labbX.examples.nu
```

---
Next Section: [Signing with NSEC3](BIND-NSEC3.md)

[Testing](testing.md)