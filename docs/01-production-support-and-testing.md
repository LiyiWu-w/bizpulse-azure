# 1. Production Support and Testing Scenarios

## 1.1 Service Dependency Diagram

See the high-level architecture diagram in [05-architecture-diagram.md](05-architecture-diagram.md). The main dependency chain is:

```text
User Browser
→ Frontend UI
→ FastAPI Backend on Azure Container Apps
→ PostgreSQL + Azure Blob Storage
→ Dashboard / Action Inbox / AI Decision Center
```

The AI path is separate:

```text
AI Decision Center
→ FastAPI AI endpoint
→ AIChatService / OpenAI Gateway
→ Azure Key Vault through Managed Identity
→ OpenAI API
```

## 1.2 Production Components

| Component | Role | Notes |
|---|---|---|
| Browser frontend | Operator and demo user interface | Calls backend APIs using `fetch` and session/CSRF protection. |
| FastAPI backend | Main API layer | Handles import workflows, dataset versions, calculations, publishing, dashboards, and AI chat. |
| PostgreSQL | Relational database | Stores workflows, dataset versions, analysis results, release pointer, and audit/state records. |
| Azure Blob Storage | File/artifact storage | Stores raw upload staging objects, standardized artifacts, and exports. |
| Azure Container Apps | Runtime hosting | Runs the packaged Docker image in Azure. |
| Azure Container Registry | Image registry | Stores the pushed Docker image used by Container Apps. |
| Azure Key Vault | Secret storage | Stores OpenAI credential; accessed server-side through managed identity. |
| OpenAI API | AI explanation provider | Used only to explain existing published results and evidence. |

## 1.3 Monitoring and Health Checks

### Application health check

Use the public health endpoint:

```bash
curl -i https://bizpulseliyi-app.mangohill-937cf0e0.eastus.azurecontainerapps.io/health/ready
```

Expected result:

```text
HTTP/2 200
ready
```

### Azure Container App logs

```bash
export RESOURCE_GROUP="rg-bizpulse-liyi-test-eastus"
export APP_NAME="bizpulseliyi-app"

az containerapp logs show   --resource-group "$RESOURCE_GROUP"   --name "$APP_NAME"   --tail 100
```

### UI health indicators

The application UI should show:

- `Data ready` near the store scope selector.
- Current dashboard metrics in Today Overview.
- Current published data status as `complete` in Data Workspace.

## 1.4 Common Incidents and Recovery Steps

| Incident | Symptoms | Likely Cause | Recovery Steps | Verification |
|---|---|---|---|---|
| App unavailable | Browser cannot open app or health check fails | Container App stopped, failed revision, bad image, or cloud outage | Check Container App status and logs; redeploy last known good image; confirm environment variables and secrets | `/health/ready` returns 200 |
| Database connection loss | API errors, dashboard cannot load current data | PostgreSQL unavailable, connection string invalid, firewall/network issue | Check PostgreSQL server status and connection string; restart app revision if needed | Dashboard loads current data |
| Blob staging failure | Upload succeeds partly but preview/commit fails | Storage connection problem or artifact hash mismatch | Check Blob storage configuration; retry upload; verify storage object and SHA-256 | Preview loads standardized records |
| Recognition failure | File cannot move past Recognition | Unsupported file type, wrong columns, file size over limit | Use supported CSV/XLSX file; confirm required columns; keep file under 8 MB | Recognition displays adapter and source fields |
| Mapping failure | Mapping cannot be confirmed | Required canonical fields missing or invalid mapping | Review suggested mapping; use supported source schema | Mapping confirmation succeeds |
| Quality failure | Missing fields or conflicts reported | Invalid rows, missing required fields, conflicting business keys | Fix source data; re-upload; remove conflicting records | Quality status passed or conflicts 0 |
| Publish blocked | Prepared data cannot publish | Dataset version incomplete, calculation missing, or release eligibility failed | Run/retry calculations; ensure required domains are complete | Current published data is complete |
| CSRF validation failed | Write action or AI ask rejected | Session expired or wrong CSRF token | Refresh, log in again, use correct operator/demo session | Request succeeds |
| AI channel disabled | AI returns channel disabled | Admin channel toggle not enabled | Enable Ordinary Login AI and/or Public Demo AI in Admin Console | AI question returns answer |
| AI answer rejected | `answer_merge_rejected` shown | AI response could not be safely bound to authoritative evidence | Treat as guardrail behavior; ask evidence-supported question | Supported prompt returns answered response |

## 1.5 Testing Scenarios and Results

### Manual End-to-End Tests

| Test Case | Expected Result | Actual Result | Status | Evidence |
|---|---|---|---|---|
| Upload CSV file | File is accepted for import workflow | CSV file appears as ready to upload | Pass | `screenshots/01-upload-files.png` |
| Recognize source | System identifies file role and fields | Recognition step becomes ready | Pass | `screenshots/02-recognition.png` |
| Confirm mapping | Suggested mapping is available and confirmable | Mapping displays `shopee_advertising_csv` and `canonical.import.v1` | Pass | `screenshots/03-mapping.png` |
| Quality check | Validation reports status and missing required fields | Status passed; missing required fields empty | Pass | `screenshots/04-quality.png` |
| Preview standardized data | Standardized rows are displayed | Preview table shows normalized advertising records | Pass | `screenshots/05-preview-top.png` |
| Prepare commit plan | Commit plan can be generated | Prepare commit plan button available | Pass | `screenshots/06-preview-commit-plan.png` |
| Commit immutable dataset version | New version is created without overwriting old data | Import quality summary shows rows read/kept and 0 conflicts | Pass | `screenshots/07-import-quality-commit.png` |
| Calculate results | Deterministic calculations run for prepared dataset | Calculation submitted and status updates | Pass | `screenshots/08-calculate-results.png` |
| Publish prepared data | Current published dataset becomes ready | Current data status is ready/complete | Pass | `screenshots/09-published-data-ready.png` |
| Today Overview | Published metrics display on dashboard | Net sales, orders, ad spend, profit, and stockout risk visible | Pass | `screenshots/10-business-overview.png` |
| Sales & Advertising | Sales/ad metrics and chart display | Gross sales, net sales, ad spend, and daily chart visible | Pass | `screenshots/11-sales-advertising.png` |
| Inventory & Replenishment | Inventory risk displays | P0 attention items and replenishment table visible | Pass | `screenshots/12-inventory-replenishment.png` |
| Profit & Cost | Profit metrics and cost chart display | Net revenue, contribution profit, operating profit visible | Pass | `screenshots/13-profit-cost.png` |
| AI Decision Center | AI explains published data and evidence | AI returns an answered profit explanation | Pass | `screenshots/14-ai-answer.png` |
| AI evidence traceability | AI has evidence-backed facts | Authoritative facts and Evidence links visible | Pass | `screenshots/15-ai-evidence.png` |

### Post-Deployment Smoke Tests

Run after every deployment:

```bash
curl -i https://bizpulseliyi-app.mangohill-937cf0e0.eastus.azurecontainerapps.io/health/ready
```

Then validate in browser:

1. Open the app URL.
2. Confirm Today Overview loads.
3. Confirm Data Workspace shows current published data as complete.
4. Confirm Sales & Advertising page loads current dataset metrics.
5. Confirm Inventory & Replenishment page loads current dataset metrics.
6. Confirm Profit & Cost page loads current dataset metrics.
7. Ask AI Decision Center: `Explain profit changes`.

### Unit, Integration, and End-to-End Test Coverage Summary

| Test Level | What it covers | Example validation |
|---|---|---|
| Unit tests | Adapter parsing, mapping validation, deterministic calculation functions | Invalid input is rejected; valid canonical fields are accepted |
| Integration tests | API + services + PostgreSQL + Blob-compatible storage | Import workflow stores files, records metadata, creates dataset version |
| End-to-end tests | Full user workflow through browser/API | Upload → recognition → mapping → quality → preview → commit → calculate → publish → dashboard |
| Smoke tests | Deployed cloud environment | Health endpoint, dashboard load, AI answer |

## 1.6 Support Notes

- Keep OpenAI API keys out of GitHub, browser code, screenshots, and terminal output.
- Use Azure Key Vault and Managed Identity for OpenAI credential access.
- Do not treat Demo Viewer uploads as official data imports.
- If AI refuses or rejects an answer, verify whether the request is unsupported or lacks evidence before changing code.
