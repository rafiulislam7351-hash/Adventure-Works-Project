# Adventure Works — End-to-End Azure Data Engineering Project

![Architecture](adventure_works_architecture.png)

An end-to-end **Azure data engineering pipeline** built on the **Medallion Architecture** (Bronze → Silver → Gold) for the Adventure Works dataset, with automated monitoring and email alerting.

## Architecture Overview

Raw Adventure Works data lands in **Azure Data Lake Storage Gen2**, gets transformed in **Azure Databricks**, refined into a Gold layer served by **Azure Synapse Analytics**, and is finally ready for BI/reporting — while a **Function App + Logic App alert system** watches the pipeline and sends email notifications via the Outlook connector, monitored by **Application Insights**.

```
Source Files ──▶ ADLS Gen2 (source) ──▶ Databricks (transform) ──▶ Synapse (gold) ──▶ BI/Reports
                        │                                                        │
                        └────────── Alert System (Function App + Logic App) ─────┘
                                             │             │
                                     Outlook Email    Application Insights
```

## Tech Stack

| Layer | Service |
|---|---|
| Data Lake | Azure Data Lake Storage Gen2 (`advwprojectstorage`, HNS enabled) |
| Processing / Transform | Azure Databricks (Premium, Unity Catalog, No-Public-IP) |
| Warehouse / Serving | Azure Synapse Analytics (`goldlayeradvw`, dedicated + serverless SQL) |
| Alerting / Orchestration | Azure Function App + Logic Apps (`alertsystem`) |
| Notification | Outlook.com managed connector |
| Monitoring | Application Insights + Smart Detection |
| IaC | Azure ARM Templates (this repo's `template.json`) |
| CI/CD & Versioning | GitHub ↔ Synapse workspace integration |

## Project Structure

```
├── template.json                    # Full ARM template — all Azure resources as code
├── adventure_works_architecture.png # Architecture diagram
└── README.md
```

## Resource Group Resources (from ARM template)

- **Microsoft.Databricks/workspaces** — `datamanipulator` (Premium SKU, Unity Catalog, no public IP, ZRS storage)
- **Microsoft.Synapse/workspaces** — `goldlayeradvw` (West US 2, linked to `advwprojectstorage` `gold` filesystem, GitHub-connected)
- **Microsoft.Storage/storageAccounts** —
  - `advwprojectstorage` — ADLS Gen2 with `source`, `silver`, `gold`, `parameter` containers
  - `adventureworkprojec9fc1` — Function App storage (queues, tables, file share)
- **Microsoft.Web/sites** — `alertsystem` Function App (Workflow Standard WS1 plan)
- **Microsoft.Web/connections** — `outlook` connector for email alerts
- **microsoft.insights/components** — Application Insights with Smart Detection rules
- **microsoft.insights/actionGroups** — Smart Detection action group

## How It Works

1. **Ingest** — Adventure Works source files are uploaded to the `source` container in ADLS Gen2.
2. **Transform (Silver)** — Databricks notebooks clean, standardize and enrich the raw data, writing to the `silver` container.
3. **Serve (Gold)** — Curated business-ready datasets land in the `gold` container, registered to the Synapse workspace for SQL analytics.
4. **Monitor & Alert** — Application Insights collects telemetry; the alert system (Function/Logic App) detects failures or anomalies and emails stakeholders automatically via Outlook.

## Key Features

- Medallion lakehouse architecture (Bronze / Silver / Gold containers)
- Entire infrastructure reproducible from a single ARM template
- Synapse workspace directly integrated with this GitHub repository
- Enterprise-grade security: TLS 1.2+, no public IP on Databricks, private containers, HTTPS-only
- Automated failure detection with email notifications
- Smart Detection rules (slow response, memory leaks, exception spikes, security issues)

## Getting Started

1. Deploy `template.json` via Azure Portal ("Deploy a custom template") or Azure CLI:
   ```bash
   az deployment group create      --resource-group Adventure_Work_Project      --template-file template.json      --parameters vulnerabilityAssessments_Default_storageContainerPath=<your-storage-path>
   ```
2. Upload Adventure Works source data to the `source` container.
3. Run the Databricks transformation notebooks → Silver.
4. Load Gold tables in Synapse and connect your reporting tool.
5. Configure the Outlook connection in the Logic App to enable email alerts.

## License

MIT
