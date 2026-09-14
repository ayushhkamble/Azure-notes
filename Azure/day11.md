
# Day 11 — Azure Administration
### Topics: Introduction to Azure Monitor, Workbooks & Dashboard Hub | Setting Up Metrics & Alerts | Using Log Analytics

---

## 1. Introduction to Azure Monitor, Azure Workbooks, Azure Dashboard Hub

### What is Azure Monitor?
<cite index="118-1">Azure Monitor is Microsoft's unified observability service for collecting, analyzing, and acting on telemetry from cloud and hybrid environments — it enables you to understand the health, performance, and reliability of your Azure applications and infrastructure by bringing together metrics, logs, traces, and events into a single observability experience.</cite>

<cite index="118-1">This monitoring data is integrated into the Azure portal experience for each service</cite>, and <cite index="118-1">the Azure Monitor data platform also supports other services such as Defender for Cloud and Microsoft Sentinel.</cite>

### The Two Halves of the Data Platform
Azure Monitor's data is split into two complementary stores:

| Data Type | What It Is | Storage |
|---|---|---|
| **Metrics** | <cite index="124-1">Lightweight, numeric monitoring data capable of supporting near real-time scenarios</cite> | Time-series database |
| **Logs** | <cite index="119-1">Detailed event/record data, collected, transformed, and routed to tables in a Log Analytics workspace</cite> | Log Analytics workspace |

### Where You See This Data in the Portal
<cite index="120-1">Most Azure services share a common set of monitoring menu items: the Overview page shows resource details and current state, a Monitoring tab includes charts for key metrics, and an Activity log lets you view subscription-level operational entries for that resource.</cite> <cite index="120-1">Selecting any chart opens Metrics explorer for deeper analysis.</cite>

### Azure Workbooks
<cite index="126-1">Workbooks provide a flexible canvas for data analysis and the creation of rich visual reports within the Azure portal — they let you tap into multiple data sources from across Azure and combine them into unified interactive experiences.</cite>

**Key characteristics:**
- <cite index="126-1">Workbooks combine text, log queries, metrics, and parameters into rich interactive reports</cite>, making them great for **freeform exploration**
- <cite index="127-1">Multi-source integration — connect data from Azure Monitor, Log Analytics, Azure Metrics, Application Insights, and even external APIs</cite>
- <cite index="127-1">Support dynamic/configurable parameters, letting users apply filters like time ranges or specific resources directly within the workbook</cite>
- <cite index="127-1">Workbooks can be shared via links or exported as JSON files for reuse or version control</cite>
- <cite index="126-1">Accessible via Monitor → Workbooks in the Azure portal, or from the Workbooks tab within a Log Analytics workspace</cite>, and offer a **gallery** of saved workbooks and pre-built templates

### Azure Dashboards and Dashboard Hub
<cite index="140-1">Dashboards are a focused and organized view of your cloud resources in the Azure portal, used as a workspace to monitor resources and quickly launch tasks for day-to-day operations — for example, you can build custom dashboards based on projects, tasks, or user roles.</cite>

<cite index="140-1">All dashboards are private when created, and each user can create up to 100 private dashboards. If you publish and share a dashboard with others in your organization, the shared dashboard is implemented as an Azure resource in your subscription and doesn't count towards the private dashboard limit.</cite>

**Dashboard hub (newer experience):**
<cite index="138-1">Dashboard hub offers editing features such as tabs, a rich set of tiles with support for different data sources, and dashboard access in the latest version of the Azure mobile app. Currently, Dashboard hub can only be used to create and manage shared dashboards — these are implemented as Azure resources visible to all users with subscription-level access.</cite>

> ⚠️ <cite index="138-1">Private dashboards aren't currently supported in Dashboard hub — to create a private dashboard, or share it with only a limited set of users, use the classic Dashboard view instead.</cite>

### Workbooks vs. Dashboards — When to Use Which
<cite index="132-1">Microsoft Learn guidance is to treat workbooks as rich investigative views and dashboards as broad operational summaries — keep the two definitions intentionally different.</cite>

| Aspect | Dashboards | Workbooks |
|---|---|---|
| **Purpose** | At-a-glance operational summary | Deep investigative analysis / reporting |
| **Interactivity** | Static tiles, minimal filtering | <cite index="131-1">Rich, parameterized, narrative reports combining text, charts, grids, and live query results</cite> |
| **Best for** | Daily ops "single pane of glass" | Weekly reviews, capacity planning, root-cause investigation |
| **Sharing** | Publish as a resource, RBAC-controlled | Share via link or export as JSON |

📘 **Official Docs:**
- [Azure Monitor overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/overview)
- [Azure Workbooks overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
- [Create and manage dashboards in Dashboard hub – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-portal/dashboard-hub)

### 🧪 Practice Lab
1. In the Portal, go to **Monitor → Overview** and explore the summary tiles (Alerts, Service Health, etc.).
2. Open **Monitor → Workbooks** → browse the gallery → open a pre-built template (e.g., "Virtual Machine Insights" if you have a VM from Day 4/5).
3. Create a new blank workbook, add a **text** block and a **metrics** chart, then **Save** it.
4. Go to **Dashboard → + New dashboard → Blank dashboard**, pin a resource tile and a Markdown tile, then **Publish and share** it.
5. Try the newer **Dashboard hub** (search "Dashboard hub" in the Portal) and compare the tile options available there versus the classic Dashboard experience.

---

## 2. Setting Up Metrics and Alerts

### Metrics Explorer Recap
Metrics are numeric, time-series data (CPU %, network bytes, disk IOPS, etc.). <cite index="124-1">Platform and custom metrics are stored for 93 days, but you can only query up to 30 days' worth of data on any single chart</cite> — pan the chart to view older data within the retention window.

### How Alerts Work — Core Concepts
<cite index="142-1">An alert rule monitors your data and captures a signal indicating something is happening on the specified resource. The rule checks whether that signal meets the criteria of the condition — if it does, the alert is triggered, the associated action group initiates, and the alert's state updates.</cite>

<cite index="142-1">You can alert on any metric or log data source in the Azure Monitor data platform.</cite> <cite index="142-1">If you're monitoring more than one resource, the alert rule condition is evaluated separately for each resource, and alerts fire for each resource separately.</cite>

### The 3 Building Blocks of an Alert Rule
```
1. Scope (Resource)  →  What resource(s) to monitor
2. Condition (Signal) →  What metric/log/activity signal, and the threshold
3. Actions            →  What happens when the condition is met (Action Group)
```

### Alert Types

| Alert Type | Monitors |
|---|---|
| **Metric alerts** | Near real-time numeric data (CPU, memory, response time) |
| **Log search alerts** | Results of a scheduled KQL query against Log Analytics |
| **Activity log alerts** | Subscription-level events (e.g., "a VM was deleted") |
| **Resource health alerts** | Underlying platform health of a specific resource |

### Action Groups
<cite index="148-1">Action groups define the unique set of actions and users to be notified when an alert fires. For example, to notify three users by email for two different alert rules, you only need to create one action group and apply it to both rules.</cite>

<cite index="142-1">Action groups can include notification methods (email, SMS, push notifications), and automated actions such as Automation runbooks, Azure Functions, ITSM incidents, Logic Apps, secure webhooks, and Event Hubs.</cite>

<cite index="148-1">You can add up to five action groups to a single alert rule.</cite> <cite index="148-1">Action groups are a global service by default — if one region goes down, traffic is automatically routed and processed in another region, providing built-in disaster recovery.</cite>

### Alert States
<cite index="142-1">Alert conditions are set by the system: when an alert fires, its condition is "fired," and once the underlying condition clears, it's set to "resolved." Fired alert instances are read-only and can't be edited — configuration changes only apply to future alerts.</cite> <cite index="142-1">Separately, the User response (New, Acknowledged, or Closed) is set manually and doesn't change until the user updates it.</cite> <cite index="144-1">Alerts are stored for 30 days and deleted after that retention period.</cite>

### Creating a Metric Alert — CLI Example
```bash
# First, create an action group
az monitor action-group create \
  --resource-group rg-monitor-day11 \
  --name ag-admin-team \
  --short-name AdminTeam \
  --action email AdminEmail admin@contoso.com

# Then create a metric alert rule (example: high CPU on a VM)
az monitor metrics alert create \
  --name Alert-HighCPU \
  --resource-group rg-monitor-day11 \
  --scopes <vm-resource-id> \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action ag-admin-team \
  --severity 2
```

### Alert Processing Rules (Advanced)
<cite index="143-1">As the number of alert rules grows, manually ensuring each has the right set of action groups gets harder — Alert processing rules let you specify that logic once, in a single rule, instead of setting it consistently across every alert rule. They also cover alert types that aren't generated by an alert rule at all, like Azure Backup alerts or VM Insights guest health alerts.</cite>

📘 **Official Docs:**
- [Overview of Azure Monitor alerts – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [Create and manage action groups – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups)

### 🧪 Practice Lab
1. Create a resource group and an action group using the CLI command above (replace with your own email).
2. Create a metric alert rule on any resource you have (VM, storage account, etc.) — e.g., trigger when CPU exceeds a threshold, or when a storage account's used capacity crosses a limit.
3. In the Portal, go to **Monitor → Alerts → Alert rules** and confirm your rule appears.
4. Explore **Monitor → Metrics**, select a resource, and manually create an alert directly from a metrics chart by clicking **+ New alert rule**.
5. Discuss: why is it better to create one reusable Action Group ("AdminTeam") rather than re-entering the same email address into every single alert rule?

---

## 3. Using Log Analytics

### What is a Log Analytics Workspace?
<cite index="152-1">A Log Analytics workspace is a data store into which you can collect any type of log data from all of your Azure and non-Azure resources and applications.</cite> <cite index="152-1">Azure Monitor Logs automatically creates the tables required to store monitoring data collected from your Azure environment, and you can create custom tables for data from non-Azure sources.</cite>

> 💡 <cite index="152-1">Microsoft Sentinel documentation refers to this same resource as a "Microsoft Sentinel workspace" when it's enabled for Sentinel — it's the identical Log Analytics workspace, just billed under Sentinel pricing when that feature is turned on.</cite>

### Getting Data Into a Workspace
<cite index="158-1">Resources don't automatically send logs to a workspace — you have to configure Diagnostic settings for each resource, or use Azure Policy to push that configuration at scale.</cite> <cite index="158-1">For the subscription-level Azure Activity Log, this is configured via Diagnostic settings for the subscription itself (Monitor → Activity log → Export Activity Logs), rather than per individual resource. For VMs specifically, logs and metrics flow through the Azure Monitor Agent, configured using a Data Collection Rule (DCR) that defines what to collect and where to send it.</cite>

> 🎯 **Teaching point:** This is a common beginner mistake — creating a Log Analytics workspace does **not** automatically start collecting data. You must explicitly configure a **diagnostic setting** on each resource (or subscription) pointing to that workspace.

### The Log Analytics Query Tool
<cite index="151-1">Log Analytics is the tool in the Azure portal used to edit and run log queries for analyzing data in Azure Monitor Logs.</cite> <cite index="151-1">When you open Logs from Azure Monitor or a Log Analytics workspace directly, you have access to all records in that workspace; when you open Logs from a specific resource, your data is limited to that resource's log data.</cite>

**Two modes:**
| Mode | Description |
|---|---|
| **Simple mode** | <cite index="151-1">Point-and-select interface, no KQL knowledge required</cite> |
| **KQL mode** | <cite index="154-1">Gives advanced users the full power of Kusto Query Language to derive deeper insights</cite>, with <cite index="151-1">IntelliSense for KQL commands and color-coded syntax</cite> |

### Kusto Query Language (KQL) Basics
<cite index="157-1">KQL uses a pipe-based syntax where each operator transforms the result of the previous one</cite> — <cite index="158-1">if you've used PowerShell's pipeline or Unix command chaining, the mental model is similar.</cite>

**Example query — recent delete operations from the Activity Log:**
```kql
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "delete"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
| order by TimeGenerated desc
```
<cite index="158-1">Reading this step by step: start with the `AzureActivity` table, filter to the last 24 hours, filter to operations containing "delete," select only the relevant columns, then sort by time descending.</cite>

### Tables Are Organized by Solution
<cite index="157-1">Tables in a workspace are grouped by solution — e.g., `AzureDiagnostics`, `AppRequests` (from Application Insights), `ContainerLog`, `SecurityEvent`, and more.</cite> <cite index="153-1">The schema browser in the Log Analytics interface lists all available tables in the current scope.</cite>

### Example Queries and Learning Resources
<cite index="151-1">When you start Log Analytics, a dialog with example queries appears, categorized by solution — browsing these is a good way to learn how to write your own queries</cite>, and <cite index="157-1">Microsoft provides hundreds of pre-built queries under the "Queries" tab covering common scenarios like VM performance, container logs, security events, and network diagnostics.</cite>

### Saving and Reusing Queries
<cite index="158-1">You can save queries to a personal or shared library within a Log Analytics workspace — shared queries are visible to everyone with access to the workspace, and building a library of your team's commonly used queries is worth doing.</cite> <cite index="158-1">You can also pin query results directly into a Workbook to combine multiple queries into a single dashboard-like view with time controls and parameterization</cite> — tying directly back to **Section 1**.

### Cost Considerations
<cite index="157-1">Querying data within the interactive retention period (default 30 days) has no query cost; querying archived data beyond that incurs a search job cost. Log ingestion itself is the primary cost driver overall.</cite> <cite index="158-1">Log Analytics charges for data ingestion and for retention beyond the free tier — typically 31 days for most tables, 90 days for certain security tables if Defender for Cloud is enabled.</cite>

📘 **Official Docs:**
- [Log Analytics workspace overview – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)
- [Overview of Log Analytics in Azure Monitor – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-overview)
- [Log queries in Azure Monitor – Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-query-overview)

### 🧪 Practice Lab
1. Create a Log Analytics workspace:
   ```bash
   az monitor log-analytics workspace create \
     --resource-group rg-monitor-day11 \
     --workspace-name law-day11-demo
   ```
2. Configure a **diagnostic setting** on a resource (e.g., your storage account from earlier days) to send logs to this workspace:
   ```bash
   az monitor diagnostic-settings create \
     --name diag-to-law \
     --resource <resource-id> \
     --workspace law-day11-demo \
     --metrics '[{"category": "AllMetrics", "enabled": true}]'
   ```
3. In the Portal, open your workspace → **Logs** → run the sample `AzureActivity` KQL query above (adjust the table name if no activity exists yet — try `AzureDiagnostics` or check the schema browser for available tables).
4. Browse the **Queries** gallery and load one example query relevant to a resource type you've deployed this course (VM, storage, network).
5. Save your query to the workspace's **shared** query library so a teammate could reuse it.
6. Discuss: why does Microsoft separate "ingestion cost" from "query cost," and why might a company want to carefully control which diagnostic logs get sent to a workspace?
7. Clean up:
   ```bash
   az group delete --name rg-monitor-day11 --yes
   ```

---

## Quick Recap Table

| Concept | One-Line Summary |
|---|---|
| **Azure Monitor** | Unified observability platform — collects metrics, logs, traces, and events from all Azure resources |
| **Metrics vs Logs** | Metrics = lightweight numeric time-series data; Logs = detailed event data queried via KQL |
| **Workbooks** | Rich, interactive, multi-source investigative reports — best for deep analysis |
| **Dashboards / Dashboard hub** | At-a-glance operational summaries; Dashboard hub is the newer shared-only experience with mobile support |
| **Alerts** | Scope + Condition + Action Group = an alert rule; Action Groups define who/what gets notified |
| **Log Analytics Workspace** | Central data store for log data — requires explicit diagnostic settings per resource to populate |
| **KQL** | Pipe-based query language for analyzing logs — filter, project, and sort data step by step |

> 🎯 **Key takeaway:** Effective Azure monitoring isn't automatic — you have to deliberately wire resources to a Log Analytics workspace via diagnostic settings, build meaningful alert rules with the right action groups, and choose the right visualization (Dashboard for daily glance, Workbook for deep investigation) for the audience that needs it.

--- 
