# 🛡️ Wazuh Custom Dashboards

> A collection of production-ready custom dashboards for Wazuh SIEM, designed to improve SOC visibility across alert monitoring, software inventory hygiene, and vulnerability/threat panorama.

## Available Dashboards

| Dashboard | Description | Index Pattern |
|-----------|-------------|---------------|
| [Alert Monitoring](./alert-monitoring/) | Real-time alert triage with severity breakdown | `wazuh-alerts-*` |
| [Software Hygiene](./software-hygiene/) | Software inventory visibility across all agents | `wazuh-states-inventory-packages-*` |
| [Threat & Vuln Panorama](./threat-vuln-panorama/) | CVE exposure overview by severity and status | `wazuh-alerts-*` or `wazuh-states-vulnerabilities-*` |

## Requirements

- Wazuh 4.x
- OpenSearch Dashboards 2.x
- Index patterns already configured in your stack

## How to Import

1. Go to **Stack Management** → **Saved Objects**
2. Click **Import**
3. Select the `.ndjson` file from the dashboard folder you want
4. Choose your conflict resolution preference (recommended: **Overwrite**)
5. Confirm import → navigate to **Dashboards**

## Contributing

Issues and PRs are welcome. If you adapt these dashboards to other Wazuh versions or add new visualizations, feel free to open a pull request.

## License

MIT — free to use, modify and distribute.