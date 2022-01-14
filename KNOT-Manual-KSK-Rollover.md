## Manual KSK Rollover

The KSK rollover is usually done at the end of its lifetime. But a key rollover can be forced before that by issuing the rollover command.

Our KASP policy is configured to not perform KSK rollovers automatically, but we can still request one manually:

1. Initiate a KSK rollover:
```bash
sudo knotc zone-key-rollover labbX.examples.nu ksk
```
2. Check that the new KSK has been generated:
```bash
sudo keymgr labbX.examples.nu list
```

3. Show the DS RRs that we are about to publish. Notice that they share the key tag with the KSK:
```bash
sudo keymgr labbX.examples.nu ds
```
4. Ask your teacher to update the DS in the parent zone.

5. Wait until the DS has been uploaded. Check the DS with the following command:
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```
6. As of now, we must manually tell the signer that the KSK has been submitted. 

```bash
sudo knotc zone-ksk-submitted labbX.examples.nu
```
    If the KSK is not yet ready to be submitted, you must wait a bit and try again later.
    
7. After the KSK has been submitted, check the key list and note that the old KSK has been removed.
```bash
sudo keymgr labbX.examples.nu list
```

8. Ask your teacher to remove the old DS from the parent zone.

9. Verify that the old DS has been removed
```bash
dig @ns1.examples.nu labbX.examples.nu DS
```

---
Next Section: [Algorithm Rollover](KNOT-Algorithm-Rollover.md)

[Testing](testing.md)