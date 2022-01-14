## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.


1. Check the status of your keys:
```bash
sudo rndc dnssec -status labbX.examples.nu
```

2. Initiate a KSK rollover:
```bash
sudo rndc dnssec -rollover -key <KEY ID> labbX.examples.nu
```

3. Check the status of your keys again, to see that a new KSK has been generated:
```bash
sudo rndc dnssec -status labbX.examples.nu
```

4. Verify that the dnskey record is published:
```bash
dig @127.0.0.1 labbX.examples.nu dnskey +multi
```

5. generate a DS record for the new key
```bash
sudo dnssec-dsfromkey -2 /var/cache/bind/KlabbX.examples.nu.+013+<KEY ID>.key
```

6. Ask your teacher to update the DS in the parent zone.

7. Wait until the new DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

8. Tell BIND that the new DS is published in the parent zone:
```bash
sudo rndc dnssec -checkds -key <KEY ID> published labbX.examples.nu
```

9. Ask your teacher to remove the old the DS in the parent zone.

10. Wait until the old DS has been removed. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

11. Tell BIND that the old DS has been removed from the parent zone:
```bash
sudo rndc dnssec -checkds -key <KEY ID> withdrawn labbX.examples.nu
```

12. Wait for BIND to remove the old key from the zone. This should only take a few minutes. You can check periodically with:
```bash
sudo rndc dnssec -status labbX.examples.nu
```
or
```bash
dig @127.0.0.1 labbX.examples.nu dnskey +multi
```



---
Next Section: [Algorithm Rollover](BIND-Algorithm-Rollover.md)

[Testing](testing.md)