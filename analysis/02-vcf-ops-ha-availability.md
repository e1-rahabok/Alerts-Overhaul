# VCF Operations node, cluster and HA availability

Source: VCF-Operations-Alerts-Export.xlsx (Alerts and Symptoms tabs).
Docs-dependent points are marked VERIFY. The 9.1 docs were not readable from the sandbox.

## 1. Current coverage

| Failure | Symptom in export | Alert in export | Status |
|---|---|---|---|
| VCF Ops node down, HA activated | nodedown EQ 1, IMMEDIATE, wait 1 | none | ORPHAN |
| VCF Ops node down, HA deactivated | nodedown EQ 2, CRITICAL, wait 1 | none | ORPHAN |
| Analytics processing down | UpStatus EQ 0, CRITICAL | none | ORPHAN |
| Cloud Proxy down | remotecollectordown EQ 1, IMMEDIATE | none | ORPHAN |
| Service(s) on node down | none | "One or more ... services on a node are down" (AUTO, no symptom) | UNFIREABLE from export |
| Node may be down, not reporting objects | none | "A node may be down and it is failing to report any Objects" (AUTO, no symptom) | UNFIREABLE from export |
| Postgres replication broken | ControllerSQL ReplicationState EQ 0, CRITICAL, wait 1 | "Postgres data replication is not working" (AUTO, symptom column blank) | Logic OK, linkage unclear, object mismatch |
| Services restarted or crashed | Watchdog Starts/Restarts, WARNING and CRITICAL (incl. vpostgres-repl, api, vcops-web, collector, casa, gemfire, analytics) | "...services on a node were restarted" | Covered |
| Fault Domain 1 or 2 unreachable | MESSAGE_EVENT CONTAINS text | "Fault Domain N is unreachable" (Administrative/Configuration) | Covered, fragile, misclassified |
| HA state | HAState EQ 1 (INFORMATION) | none | Only "activated" is checked |
| Witness node down | none | password and certificate alerts only | MISSING (VERIFY witness role) |
| HA VIP or external LB down | none | none | MISSING (VERIFY, see section 3) |
| NTP unreachable (cluster) | ntp unreachable GT 0, WARNING, wait 3 | "NTP Servers are not reachable" (Administrative/Availability) | Covered |

Overall:
- Node loss under HA has no alert.
- Service "down" alerts have no symptoms in the export.
- Classification puts HA events under Performance, Configuration and Administrative.

## 2. Classification (subtype)

Availability-type events filed under other subtypes:
- "services on a node are down": Administrative/Performance.
- "A node may be down ...": Administrative/Performance.
- "Fault Domain N is unreachable": Administrative/Configuration.

Availability subtype used for a non-availability event:
- "Cluster is disbalanced": Administrative/Availability. Balance is performance, not uptime.

VERIFY the subtype mapping in the UI first. The export author inferred it from numbers.

## 3. Proposed alert and symptom changes

### 3.1 Node down (highest priority)
- Bind `nodedown EQ 1` (HA activated) and `nodedown EQ 2` (HA deactivated) to a new Availability alert "VCF Operations node is down".
- Keep both tiers. Keep HA-activated at IMMEDIATE (redundancy lost), HA-deactivated at CRITICAL (data gap).
- Use wait 2 for HA-activated. A one-cycle reboot during patching should not fire.
- VERIFY the meaning of values 1 and 2 in the API. The export does not say.

### 3.2 HA redundancy lost (new)
- Add a symptom for "cluster member count below expected" or "HAState not in activated state".
- HAState only has an "activated" check today.
- Supporting evidence (snippet, VERIFY): KB 445857 reports HA stuck "activating" after adding nodes.
- Metric name for member count or HA state detail: VERIFY in API.

### 3.3 Services down (fix the empty alert)
- The alert "One or more ... services on a node are down" has no symptom.
- Add a per-service "down" symptom from the watchdog or service-status metric.
- Priority services: vpostgres, api, vcops-web, casa, collector, analytics, gemfire.
- Metric names: VERIFY in API. Current watchdog symptoms only count starts and restarts.

### 3.4 Postgres replication
- Fix the object mismatch. Symptom object is `VCF Operations Persistence`, alert object is `VCF Operations Node`.
- Confirm the symptom is linked in the UI. The export column is blank.
- Add a recommendation. Snippet: KB 398832 (reinitialize a Postgres replica) and KB 440877 (unhealthy Postgres cluster).
- Keep wait 1. A failover can create one 0 sample, so consider wait 2.

### 3.5 Cloud Proxy down
- Bind `remotecollectordown EQ 1` (IMMEDIATE) to the cloud proxy availability alert.
- Current alert uses "Collector is down" with wait 1.
- Add wait 2 to cover restart cycles.
- Salt Master and Httpd South alerts have no symptoms in the export. Bind or retire.

### 3.6 Analytics node processing down
- Bind `UpStatus EQ 0` (CRITICAL, "Node processing is down") to an Availability alert.
- Currently orphan.

### 3.7 Fault domain unreachable
- Move to Availability subtype.
- Message matching ("is unreachable") is fragile. Replace with a metric when one exists. VERIFY.

### 3.8 Witness node (VERIFY the 9.1 role first)
- Add "VCF Operations Witness Node unreachable" or "down" if the witness exists in 9.1.
- No availability metric is in the export for this object.

### 3.9 HA VIP or external load balancer (VERIFY)
- Snippet (KB 442121): VCF Ops has no built-in VIP. HA expects an external load balancer.
- The export has no alert for VIP or LB loss.
- If confirmed, add a probe alert on the VIP.

## 4. Alert-level settings
- Node and service alerts: wait 1 is fine for hard states.
- Probe-based checks: wait 2 to 3.
- Patch and reboot windows: maintenance or suppression rule. VERIFY the feature in 9.1.

## 5. Open items to confirm in the UI
1. Is "Node is down" bound to any alert in your instance?
2. Which symptom is bound to "One or more ... services on a node are down"?
3. Is the Postgres symptom linked to the Postgres alert?
4. Does the instance have a witness node in use?
5. Which metric exposes HA member count or HA state detail?
