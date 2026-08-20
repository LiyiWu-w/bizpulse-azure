# 3. Issue Diagnosis, Research, Resolution, and Sharing

This section documents major issues encountered during the BizPulse implementation, deployment, and AI enablement process.

## Issue 1: Demo and Operator Upload Flow Confusion

**Description:** Demo users and operator users appeared to share similar upload screens, which made it unclear whether a public demo upload should affect the official dashboard data.

**Expected behavior:** Formal analysis and dashboard updates should only occur through the Operator workflow.

**Actual behavior:** Demo upload behavior could be mistaken as a formal import path.

**Environment:** Browser UI, Azure-deployed app, operator/demo routes.

**Steps to reproduce:** Open demo or workspace upload page and upload a file as a viewer or operator.

**Diagnosis:** Demo Viewer upload and Operator import are two different flows. Demo Viewer upload is temporary and should not update the authoritative public release.

**Research process:** Reviewed project flow, route behavior, and assignment need for a clear usage guide.

**Resolution:** Documented the separation clearly:

```text
Demo Viewer upload = temporary demo cache
Operator import = official recognition, standardization, commit, calculate, publish flow
```

**Verification:** Operator workflow screenshots show the official six-stage upload/import process, and dashboard screenshots show the current published dataset.

## Issue 2: Duplicate Dataset Upload Handling

**Description:** Re-uploading the same dataset content caused confusion about whether a new version should be created.

**Expected behavior:** Identical content should not silently duplicate data.

**Actual behavior:** Repeated upload could surface duplicate/version behavior.

**Environment:** Operator Data Workspace import flow.

**Steps to reproduce:** Upload the same CSV content more than once, then prepare/commit.

**Diagnosis:** Dataset content should be treated using content hash and version lineage so duplicate content is not considered a new unique business dataset.

**Research process:** Compared import behavior with expected versioning model.

**Resolution:** Existing duplicate content is returned or handled without creating misleading new data.

**Verification:** Publish step screenshot shows `created: false`, indicating the system recognized existing content rather than blindly creating a new dataset.

## Issue 3: CSRF Validation Failed in Demo AI Chat

**Description:** AI requests could fail with `CSRF_VALIDATION_FAILED`.

**Expected behavior:** Valid operator/demo session should be able to send an AI request.

**Actual behavior:** Request was rejected before AI processing.

**Environment:** Browser with both operator and demo session state present.

**Steps to reproduce:** Use AI Decision Center while both operator and demo cookies/tokens exist.

**Diagnosis:** The request could select the wrong CSRF/session context.

**Research process:** Reviewed browser session behavior and AI endpoint request requirements.

**Resolution:** Adjusted session selection logic so demo routes use demo session/CSRF context and operator routes use operator session/CSRF context.

**Verification:** AI request successfully reached the backend and produced an answer.

## Issue 4: AI Channel Disabled

**Description:** AI request returned `AI_CHAT_CHANNEL_DISABLED`.

**Expected behavior:** AI should answer once the credential is configured and channel is enabled.

**Actual behavior:** Credential existed, but channel remained disabled.

**Environment:** Azure deployment, Admin AI Management, AI Decision Center.

**Steps to reproduce:** Ask a question in AI Decision Center before enabling Ordinary Login AI or Public Demo AI.

**Diagnosis:** The shared credential binding and channel availability are controlled separately.

**Research process:** Used Admin AI Management UI and backend behavior.

**Resolution:** Enabled the required AI channel in Administrator Console after validating the shared credential binding.

**Verification:** AI Decision Center returned an answered response for profit explanation.

## Issue 5: Key Vault Permission Issue

**Description:** AI credential rotation or validation reported secret access problems.

**Expected behavior:** Backend managed identity should access the OpenAI key stored in Key Vault.

**Actual behavior:** Key Vault access was unavailable until identity permissions were fixed.

**Environment:** Azure Key Vault, managed identity, Azure Container Apps.

**Steps to reproduce:** Configure AI with Key Vault but without sufficient Key Vault secret permissions for the managed identity.

**Diagnosis:** The application identity needed proper Key Vault secret access.

**Research process:** Checked Key Vault role assignments and Admin AI validation behavior.

**Resolution:** Granted the application managed identity appropriate Key Vault secret permissions.

**Verification:** Admin AI Management showed a verified shared credential fingerprint.

## Issue 6: AI Answer Merge Rejected

**Description:** Some AI prompts returned `answer_merge_rejected`.

**Expected behavior:** AI should answer supported questions using authoritative facts and evidence.

**Actual behavior:** The answer was rejected when it could not be safely bound to evidence.

**Environment:** AI Decision Center with current published dataset.

**Steps to reproduce:** Ask a question like `Prioritize next actions` when the available Action Cards/evidence are insufficient.

**Diagnosis:** This is expected guardrail behavior. BizPulse should reject unsupported AI explanations instead of inventing recommendations.

**Research process:** Compared successful and rejected AI prompts.

**Resolution:** Use evidence-supported prompts for demo, such as `Explain profit changes`.

**Verification:** `Explain profit changes` returned an answered response and authoritative facts were visible.

## Issue 7: macOS Local Install Risk from greenlet Hash

**Description:** `requirements.txt` originally had a `greenlet==3.5.5` hash that only covered Linux/AMD64.

**Expected behavior:** macOS local installation should work if README claims macOS support.

**Actual behavior:** macOS installation could fail under pip hash-checking mode.

**Environment:** macOS local development, Python 3.12, hashed requirements file.

**Steps to reproduce:** Run:

```bash
python -m pip install --require-hashes -r requirements.txt
```

on macOS when only the Linux/AMD64 greenlet hash exists.

**Diagnosis:** pip hash-checking requires a valid hash for the exact wheel selected on the local platform. macOS selects a different wheel than Linux/AMD64.

**Research process:** Downloaded the macOS `greenlet==3.5.5` wheel and generated its hash with `python -m pip hash`.

**Resolution:** Added the macOS cp312 universal2 greenlet hash to `requirements.txt`.

**Verification:** Ran:

```bash
python -m pip install --dry-run --require-hashes -r requirements.txt
```

and confirmed no greenlet hash error occurred.

## Issue 8: Azure Deployment Validation

**Description:** Needed to confirm that the deployed application, AI settings, and health check were all active in Azure.

**Expected behavior:** Deployment succeeds, health check returns 200, and AI is enabled.

**Actual behavior:** Needed validation after each deployment and AI configuration change.

**Environment:** Azure Container Apps, Azure Container Registry, Bicep deployment, Key Vault.

**Steps to reproduce:** Build/push image, deploy, then query health endpoint and UI.

**Diagnosis:** Cloud deployment success must be validated separately from local build success.

**Research process:** Used Azure CLI deployment output, container app logs, health endpoint, and UI smoke tests.

**Resolution:** Standardized the smoke test process after deployment.

**Verification:** `/health/ready` returned 200 and UI pages showed `Data ready` with dashboard metrics.
