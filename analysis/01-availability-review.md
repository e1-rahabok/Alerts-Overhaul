# VCF 9.1 availability alerts review

Source: `VCF-Operations-Alerts-Export.xlsx` (Alerts 1,193 rows, Symptoms 1,951 rows).
Scope: availability only. Performance and security are out of scope.

## Source limits

- Broadcom TechDocs, KB and blogs are blocked from the sandbox (403 or DNS failure).
- 9.1 facts below come from search snippets. They are marked VERIFY.
- Confirm VERIFY items against the 9.1 PDF once it is available in the repo.

## Export caveats

- 189 alerts have a blank Symptom Definitions column. 22 of them are Availability alerts.
- 11 of the blank alerts have a same-named symptom, so linkage is unreliable.
- Alert Type and Subtype were inferred from numbers by the export author. Check the mapping in the UI.
- 322 symptom names are not used by any alert. 91 of them are availability-related.

## Part 1: Missing availability alerts

Ranked by impact.

1. VCF Operations node down (orphan symptoms `nodedown EQ 1` and `EQ 2`).
2. vCenter health service, appliance service and NTP down (orphans). Only a ping-based app-health alert covers vCenter.
3. NSX Manager service stopped (orphans for runtime state and monitor state). Cluster status symptom is also an orphan.
4. Identity Broker node loss. Only aggregate "unhealthy" and "not reachable" alerts exist. VERIFY 9.1 multi-node broker.
5. Cloud Proxy down (orphan `remotecollectordown`). Salt Master and Httpd South alerts have no symptoms.
6. Automation and Networks service down (orphans).
7. VCF Management Services control-plane or node health. VERIFY 9.1 control-plane count.
8. ESXi unexpected reboot or PSOD. No alert found. VERIFY whether 9.1 raises a built-in event.
9. Pod restart loops (orphan symptoms with wrong label on the WARNING tier).
10. Backup job failures for vCenter, SDDC Manager and NSX (orphans). Recoverability, not uptime.

## Part 2: Symptom logic improvements

1. License Server connectivity: `Communication failure with the license server` uses wait 60 cycles. At a 5-minute cycle that is about 5 hours. Suggest 2 to 3.
2. VCF services runtime backup: symptom is a last-run flag (`componentBackupSucceeded EQ 0`). Alert text says 24 hours. A stopped scheduler is never caught. Use time since last success.
3. Ping-only app health (10 symptoms, `PING_AVAILABILITY`, wait 1, cancel 1, CRITICAL). One dropped ICMP cycle fires CRITICAL. Suggest wait 2 to 3 plus a service or API check. Networks is INFORMATION with wait 2, which is inconsistent.
4. Identity Broker "not reachable" alert uses `vidb_readiness NOT_EQ Ready`. Readiness is not reachability.
5. Pod restart WARNING symptom is named "greater than 10" but threshold is 3.
6. Fault Domain and Fleet LCM alerts use message text matching. The Fleet LCM text refers to the 9.1 upgrade and will be stale after upgrade.
7. Postgres replication: object mismatch (symptom on Persistence, alert on Node). Recommendation is blank.
8. Wait and cancel: 143 of 153 availability alerts use wait 1, and 150 use cancel 1. Hard states suit wait 1. Probes need wait 2 to 3.
9. Host "lost connection to vCenter" includes an unresolved standby-mode symptom. Verify whether standby hosts trigger it.

## Part 3: Subtype misclassification (verify mapping first)

- Availability events typed as Performance: `Edge HA Member Status Down`, `Edge Tunnels Down`, `Edge Agent Down`, `Failure Domain Down`, `watchquagga`, `vcsyscmd`, `vc.procmon` and `Upload` process or service stopped, and `services on a node are down`.
- Typed as Configuration: `Fault Domain N is unreachable`.
- Typed as Hardware: `vSphere HA master missing`.
- Typed as Availability but not uptime: `Cluster is disbalanced`.

## Next step

Topic deep-dive: VCF Operations node and HA. See `analysis/02-vcf-ops-ha-availability.md`.
