

Note:
The PublishCDS metadata is usually set to the the time that the DNSKEY has been published, plus dnskey-ttl, zone-propagation-delay, and publish-safety. If it is the first key for the zone, the time to propagate the zone signatures is taken into account.
https://www.mail-archive.com/bind-users@lists.isc.org/msg30868.html


To publish CDS and CDNSKEY records (KSK):
```bash
sudo dnssec-settime -P sync -1h KlabX.examples.nu.+013+05412.private
```

#### Restart BIND
```bash
sudo service bind9 restart
```

#### Verify that the Keys (and CDS/CDNSKEY) is in the zone
```bash
 dig @localhost labX.examples.nu DNSKEY +multi
```
``` bash
;; ANSWER SECTION:
labX.examples.nu. 120 IN	DNSKEY 257 3 13 (
				NvvCwBO9w8aCW2N884uA1VhJlSkSMvXf4jsfDiIgV2gu
				25LqIL2KyitKwyH/rEAEiR5Po3MpGVvvW744fnhIhw==
				) ; KSK; alg = ECDSAP256SHA256 ; key id = 5412
labX.examples.nu. 120 IN	DNSKEY 256 3 13 (
				ca4SqKRUR0GWRy/C8lChaFWtYB+zO3nX+byozlS1fxDs
				qVHRSYImDzR6ZkgOihJMWRQ3pSrYeubtIuOstSw4hw==
				) ; ZSK; alg = ECDSAP256SHA256 ; key id = 34191
```
```bash
 dig @localhost labX.examples.nu CDS
```
```bash
;; ANSWER SECTION:
labX.examples.nu. 120	IN	CDS	5412 13 2 9DA5358E29E521BC72AA75CC54C7C863C5DA8BE2F1018566717EAF86 09FBE346
```
```bash
 dig @localhost labX.examples.nu CDNSKEY
```
```bash
;; ANSWER SECTION:
labX.examples.nu. 120	IN	CDNSKEY	257 3 13 NvvCwBO9w8aCW2N884uA1VhJlSkSMvXf4jsfDiIgV2gu25LqIL2KyitK wyH/rEAEiR5Po3MpGVvvW744fnhIhw==
```
