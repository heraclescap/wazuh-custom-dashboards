# Software Hygiene Dashboard

> **Wazuh Dashboard** — Provides visibility into the software inventory across the endpoint fleet, identifies the most common packages, and tracks the number of active agents in real time.

---

## Purpose of the dashboard

Maintaining an up-to-date software inventory is a fundamental prerequisite for any vulnerability management and compliance initiative. An environment where you do not know what is installed on your machines is an environment that is impossible to defend.

This dashboard utilises the **Wazuh Syscollector module**, which periodically inventories the packages installed on each agent, to answer the following questions:

- What is the most common software across the fleet?
- How many agents are currently active and reporting data?
- Are there any agents with packages that do not comply with the organisation’s policy?

It is particularly useful for Vulnerability Management teams, CISO’s, and compliance officers who need an overview of the fleet’s status without having to search through the agent management interface one by one.

---

## Index Pattern used

```
wazuh-states-inventory-packages-*
```

Unlike alert dashboards, this one utilises Wazuh’s **state inventory** index (introduced with the Wazuh States module in version 4.5+). This index contains the data collected by each agent’s Syscollector module: package name, version, architecture, vendor, installation date, category and size.

Each record represents a **package installed on an agent at a given point in time**, enabling highly precise aggregation queries on software distribution.

---

## General overview of the dashboard

screen 30j

---

## Detailed visualisations

### 1. Top 10 Installed Software — Horizontal bar chart

**Visualisation type:** Horizontal Bar Chart

**Fields used:**
- Metric: `cardinality(agent.name)` — number of distinct agents on which the package is present (labelled ‘Agent’)
- Bucket/Dimension: `package.name` (top 10, sorted by descending cardinality) — labelled ‘Package’

**Operational role:**

This is the central visualisation of the dashboard. It answers the question: **‘Which are the ten most widespread packages in my environment, and on how many machines are they present?’**

The use of **cardinality on `agent.name`** rather than a simple count is a deliberate and important choice: if the same package appears multiple times in a single agent’s inventory (for example, because it has been scanned several times over the period), we still want to count that agent only once. Cardinality ensures that we obtain the **number of distinct machines** that have the package, not the number of times it has been indexed.

In practice, this graph allows us to:
- Identify universally present packages (kernel, libc, bash, etc.) that will serve as a baseline
- Spot anomalies: a proprietary package that should be on 5 machines but appears on 47 warrants investigation
- Prepare patching campaigns: if `openssl 1.1.1` is on 200 machines and a critical CVE is released, this graph immediately shows the extent of the exposure
- Detect unauthorised software (shadow IT) that may have been installed without going through official channels

The horizontal view is chosen because package names can be long — the horizontal bars allow for reading without truncation.

screen

---

### 2. Active Agents — Semi-circular gauge

**Visualisation type:** Gauge (Time Series Visual Builder — TSVB)

**Fields used:**
- Source: `wazuh-monitoring-*` index
- Metric: `cardinality(id)` — number of distinct agents with active status
- Filter: `status: active` (KQL filter applied to the dashboard)
- Time field: `timestamp`

**Operational role:**

This gauge displays, in near real-time, the **number of Wazuh agents currently active** in the infrastructure. It consumes the `wazuh-monitoring-*` index, which contains agent heartbeats and status changes.

The information it provides is critical on two levels:

**Immediate operational level**: If the gauge displays a figure significantly lower than the known baseline (e.g. 180 active agents instead of the usual 200), this may mean that agents have gone down, that machines are switched off, or that a connectivity issue is affecting part of the fleet. In all these cases, the inventory data is incomplete and the analyses on this dashboard do not reflect the full picture.

**Governance level**: This figure is useful for reporting: “100% of our endpoints are reporting telemetry data” is an important SOC coverage metric. If agents go down and are not restarted, coverage deteriorates silently.

The half-gauge format is chosen for its readability in a summary view — you can instantly see whether the figure is within the expected range or not.

screen

---

## Importing the dashboard

### Prerequisites

- Wazuh 4.5+ (so that the `wazuh-states-inventory-packages-*` index is available)
- Syscollector module enabled on the agents (`syscollector` in `ossec.conf`)
- `wazuh-states-inventory-packages-*` index pattern created in Stack Management
- `wazuh-monitoring-*` index available (created automatically by Wazuh)

### Check that Syscollector is collecting packages

On a Linux agent, check that collection is active:

```xml
<!-- In /var/ossec/etc/ossec.conf on the agent -->
<wodle name="syscollector">
  <disabled>no</disabled>
  <packages>yes</packages>
  ...
</wodle>
```

If `packages` is set to `no` or if the wodle is disabled, no data will be uploaded to the index.

### Import procedure

1. From the side menu, go to **Stack Management** → **Saved Objects**
2. Click **Import**
3. Select the `Software_Hygiene.ndjson` file
4. If there is a conflict, select **Overwrite**
5. Confirm the import
6. Navigate to **Dashboards** → search for **‘Software Hygiene’**

### Adjust the base URL if necessary

This dashboard contains `fieldFormatMap` entries with a hardcoded instance URL (fields `data.url`, `data.virustotal.permalink`, `data.vulnerability.reference`). Replace the URL with that of your instance in the `.ndjson` file before importing if you want the clickable links to work correctly.

---

## Recommended customisation

**Filter by agent or group**: If your environment is segmented (servers vs. workstations, production vs. development), add a filter on `agent.name` or `agent.id` to view data by scope.

**Add a view by version**: An aggregation on `package.name` + `package.version` allows you to identify obsolete versions of the same package (e.g. how many machines have OpenSSL 1.1.1 vs 3.x).

**Add a category filter**: The `package.category` field allows you to filter by specific package types (e.g. `security`, `admin`, `net`).

**Alert on inventory changes**: Combined with Wazuh Syscollector rules (`syscollector` groups in alerts), you can create alerts for newly installed or removed packages.

---

## Tested on

| Component | Version |
|-----------|---------|
| Wazuh | 4.5+ |
| OpenSearch Dashboards | 2.x |
| Index Pattern | `wazuh-states-inventory-packages-*` |
| Module | Syscollector |

---

## Licence

MIT — free to use, modify and redistribute.
