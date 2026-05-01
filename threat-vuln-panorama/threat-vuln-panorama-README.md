# Threat & Vulnerability Overview Dashboard

> **Wazuh Dashboard** — An overview of CVE exposure across the endpoint fleet, categorised by CVSS severity and distinguishing between active and resolved vulnerabilities.

---

## Purpose of the dashboard

Vulnerability management is one of the most critical use cases for a SIEM. This dashboard utilises the **Wazuh Vulnerability Detector module** to transform raw vulnerability scan data into an operational management view.

It answers the key questions of a VM (Vulnerability Management) programme:

1. How many CVEs have been detected in total, and how are they distributed by severity?
2. Of these CVEs, how many are still **active** (unresolved) and in which criticality category?
3. Where should remediation efforts be prioritised?

This dashboard is designed for use at two levels: by VM analysts seeking to prioritise patching, and by CISO or risk managers who need an overview of the organisation’s overall exposure for reporting purposes.

---

## Pattern Index used

```
wazuh-alerts-*
```

Vulnerability data is reported by Wazuh as alerts in the main index `wazuh-alerts-*`. The Vulnerability Detector module enriches these alerts with specific fields under the namespace `data.vulnerability.*`, including the CVE identifier, CVSS severity, status, affected package and NVD reference.

The dashboard filters target these fields to display only events related to vulnerabilities, excluding other types of alerts.

---

## General overview of the dashboard

screen 30j
---

## Detailed visualisations

### 1. Number of CVEs Detected by Severity — Metrics by severity

**Visualisation type:** Metric (multiple counters per group)

**Fields used:**
- Metric: `count()` — labelled ‘CVE’
- Grouping (bucket ‘group’): `data.vulnerability.severity` (top 5, sorted by descending count)

**Operational role:**

This visualisation displays a series of numerical counters, **one per severity level** detected in the environment. Typical values for the `data.vulnerability.severity` field according to the CVSS nomenclature are: `Critical`, `High`, `Medium`, `Low`, `None`.

The multi-metric display is intentional: it allows you to see the **full distribution of exposure** at a glance without having to scroll or click. Each counter represents the number of vulnerability alerts for that severity level over the selected period.

Points to note when analysing:

**Interpretation of the count**: A single CVE may appear multiple times in the count if Wazuh detects it on multiple agents or during multiple scans. This count therefore reflects the total number of detections, not the number of unique CVEs. To determine the number of unique CVEs, you should use a cardinality on `data.vulnerability.cve`.

**Prioritisation**: In a mature VM programme, the objective is to prioritise bringing the `Critical` count down to zero, followed by `High`. If the `Critical` count increases from one week to the next without corresponding remediation, this is a warning sign.

**Correlation with packages**: Reported CVEs are always associated with a specific package (`data.vulnerability.package.name` and `data.vulnerability.package.version`). By combining this dashboard with the Software Hygiene dashboard, you can quickly identify which widely used packages in the environment contain critical vulnerabilities.

The 45px font size chosen for the figures ensures readability even on a remote SOC display screen.

screen

---

### 2. Active CVEs by Severity — Active CVE metrics only

**Visualisation type:** Metric (multiple counters per group)

**Fields used:**
- KQL filter applied to the dashboard: `data.vulnerability.status: Active`
- Metric: `count()` — labelled ‘CVE’
- Grouping (bucket ‘group’): `data.vulnerability.severity` (top 4, sorted by descending count)

**Operational role:**

This visualisation is the **actionable complement** to the previous dashboard. It applies an additional filter to retain only alerts where `data.vulnerability.status` is `Active`, meaning that Wazuh considers the vulnerability to be unresolved on the endpoint.

The distinction between Panel 1 (all detected CVEs) and Panel 2 (active CVEs only) is fundamental to avoiding noise in the reporting:

**Panel 1 — All CVEs**: includes historical CVEs, CVEs that have been patched since the last scan, and CVEs that have changed status. This is the comprehensive view of what has been observed over the period.

**Panel 2 — Active CVEs**: retains only what is **still vulnerable now**. This is the view that requires action. A significant gap between the two panels is a positive sign: it means that patches have been deployed and the situation is improving.

In practice, during a weekly VM update, Panel 2 is presented: “We currently have X active critical vulnerabilities, Y high-severity and Z medium-severity vulnerabilities across our infrastructure.” It is this figure that feeds into the attack surface reduction KPIs.

Top 4 instead of 5: the `data.vulnerability.severity` field may contain the value `None` for packages where a CVE is listed but no severity score has been assigned. Limiting the list to 4 prevents a non-actionable category from being displayed in the view of active critical CVEs.

A font size of 50px ensures readability on large screens.

screen

---

## Importing the dashboard

### Prerequisites

- Wazuh 4.x with the Vulnerability Detector module enabled
- At least one vulnerability scan performed (the data appears in `wazuh-alerts-*` with the rule group `vulnerability-detector`) or if you want you can directrly use the `wazuh-states-vulnerabilities-*` index
- `wazuh-alerts-*` or `wazuh-states-vulnerabilities-*` index pattern configured

### Enable the Vulnerability Detector (if not already done)

In the Wazuh manager configuration file (`/var/ossec/etc/ossec.conf`):

```xml
<vulnerability-detector>
  <enabled>yes</enabled>
  <interval>12h</interval>
  <run_on_start>yes</run_on_start>
  <provider name="canonical">
    <enabled>yes</enabled>
    <os>noble</os>
    <os>jammy</os>
    <os>focal</os>
    <update_interval>1h</update_interval>
  </provider>
</vulnerability-detector>
```

Adapt the `<os>` entries to your distributions. After activation, run an initial scan and wait for alerts to appear in `wazuh-alerts-*` with `rule.groups: vulnerability-detector`.

### Import procedure

1. From the side menu, go to **Stack Management** → **Saved Objects**
2. Click **Import**
3. Select the file `Threat_&_Vuln_Panorama.ndjson`
4. If there is a conflict, select **Overwrite**
5. Confirm the import
6. Navigate to **Dashboards** → search for **‘Threat & Vuln Panorama’**

### Adjust the base URL if necessary

This dashboard contains `fieldFormatMap` entries with a hardcoded instance URL (fields `data.url`, `data.virustotal.permalink`, `data.vulnerability.reference`). Replace the URL with that of your instance in the `.ndjson` file before importing if you want the clickable links to work correctly.

---

## Recommended customisation

**Add a filter by agent or agent group**: If you have separate environments (cloud, on-prem, DMZ), a filter on `agent.name` or on Wazuh groups allows you to view data by risk zone.

**Add a top 10 list of the most prevalent CVEs**: An aggregation on `data.vulnerability.cve` sorted by count allows you to see the CVEs affecting the most machines — ideal for prioritising patching.

**Add a top 10 list of vulnerable packages**: An aggregation on `data.vulnerability.package.name` + `data.vulnerability.package.version` with a `status: Active` filter provides a list of packages to patch as a priority.

**Create an alert for new critical CVEs**: In Wazuh, you can configure email alerts or webhooks triggered by rules in the `vulnerability-detector` group with a severity level of ≥ 12.

**Correlation with MITRE ATT&CK**: Certain CVEs are associated with MITRE techniques via the `rule.mitre.*` fields. An additional panel on these fields helps to enrich the tactical view.

---

## Key fields for further analysis

| Field | Description |
|-------|-------------|
| `data.vulnerability.cve` | CVE identifier (e.g. CVE-2024-1234) |
| `data.vulnerability.severity` | Severity: Critical, High, Medium, Low |
| `data.vulnerability.status` | Status: Active, Obsolete, etc. |
| `data.vulnerability.package.name` | Affected package |
| `data.vulnerability.package.version` | Vulnerable version installed |
| `data.vulnerability.cvss.cvss3.base_score` | CVSS v3 score |
| `data.vulnerability.reference` | Link to the NVD entry |
| `data.vulnerability.title` | Short description of the vulnerability |
| `agent.name` | Name of the affected endpoint |

---

## Tested on

| Component | Version |
|-----------|---------|
| Wazuh | 4.x |
| OpenSearch Dashboards | 2.x |
| Module | Vulnerability Detector |
| Index Pattern | `wazuh-alerts-*` or `wazuh-states-vulnerabilities-*` |

---

## Licence

MIT — free to use, modify and redistribute.
