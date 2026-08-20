# 2. System Setup Instructions

## 2.1 Prerequisites

Install or prepare the following:

| Requirement | Purpose |
|---|---|
| Git | Clone and manage the repository |
| Python 3.12 | Backend runtime and dependency installation |
| Docker Desktop | Build and test the container image locally |
| Azure CLI | Deploy and inspect Azure resources |
| PostgreSQL-compatible database | Application database engine |
| Azure subscription | Container Apps, Container Registry, Blob Storage, Key Vault |
| OpenAI API key | AI explanation layer, stored in Key Vault |

Important terminology:

- **PostgreSQL** is the database engine.
- **Neon**, if used, is a hosted PostgreSQL provider, not a different database type.
- **Docker** packages the application runtime; it is not a database.
- **Azure** hosts the deployed application and cloud resources.

## 2.2 Clone the Repository

```bash
git clone https://github.com/LiyiWu-w/bizpulse-azure.git
cd bizpulse-azure
```

If working from the local capstone folder used during deployment:

```bash
cd /Users/admin/Desktop/NEWCaostone-azure/bizpulse
```

## 2.3 Python Environment Setup

Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
```

Install dependencies:

```bash
python -m pip install --require-hashes -r requirements.txt
```

For macOS local installation, the `greenlet==3.5.5` lock entry should include both Linux/AMD64 and macOS cp312 universal2 hashes.

Validation:

```bash
python -m pip check
python -m pip install --dry-run --require-hashes -r requirements.txt
```

## 2.4 Database Setup

BizPulse requires a PostgreSQL-compatible database. It can be hosted locally, in Neon, or in Azure Database for PostgreSQL.

### Local PostgreSQL example

```bash
createdb bizpulse_local
psql bizpulse_local
```

Use the repository migration process to create/update tables. If using Alembic in this branch:

```bash
alembic upgrade head
```

If the project uses a different migration entrypoint, use the migration command defined in the repository.

## 2.5 Environment Configuration

Create a local environment file such as `.env.local`. Do not commit real credentials.

Recommended configuration categories:

| Setting | Purpose |
|---|---|
| Database URL | Connect backend to PostgreSQL |
| Blob storage connection | Store raw uploads, standardized artifacts, and exports |
| Session/CSRF secret | Protect authenticated write actions |
| Environment name | local/staging/production behavior |
| AI enabled flag | Enables or disables AI feature at runtime/deployment |
| Key Vault URL | Location of OpenAI credential in Azure |
| Managed Identity identifiers | Allows backend to access Key Vault without exposing secret |

Security rule:

```text
Do not place OPENAI_API_KEY directly in frontend code, GitHub, screenshots, or public logs.
```

## 2.6 Local Run

The most reproducible local run path is Docker-based.

Build image:

```bash
docker build -t bizpulse-local .
```

Run image:

```bash
docker run --rm -p 8000:8000 --env-file .env.local bizpulse-local
```

Validation:

```bash
curl -i http://localhost:8000/health/ready
```

Expected result:

```text
HTTP/1.1 200 OK
ready
```

## 2.7 Azure Deployment

Set environment variables:

```bash
export RESOURCE_GROUP="rg-bizpulse-liyi-test-eastus"
export APP_NAME="bizpulseliyi-app"
export REGISTRY_NAME="bizpulseliyiacr08190141"
```

Build and push container image:

```bash
SOURCE_REVISION=$(git rev-parse --short=12 HEAD)
SOURCE_TREE_SHA=$(git rev-parse HEAD^{tree})
IMAGE_INPUT_SHA256=$(git ls-files -z | xargs -0 shasum -a 256 | shasum -a 256 | awk '{print $1}')
BUILD_CONTEXT_SHA256="$IMAGE_INPUT_SHA256"
IMAGE_TAG="${SOURCE_REVISION}-release-$(date +%Y%m%d%H%M%S)"

ACR_LOGIN_SERVER=$(az acr show   --resource-group "$RESOURCE_GROUP"   --name "$REGISTRY_NAME"   --query loginServer -o tsv)

az acr login --name "$REGISTRY_NAME"

docker buildx build   --no-cache   --platform linux/amd64   --build-arg SOURCE_REVISION="$SOURCE_REVISION"   --build-arg SOURCE_TREE_SHA="$SOURCE_TREE_SHA"   --build-arg IMAGE_INPUT_SHA256="$IMAGE_INPUT_SHA256"   --build-arg BUILD_CONTEXT_SHA256="$BUILD_CONTEXT_SHA256"   -t "$ACR_LOGIN_SERVER/bizpulse:$IMAGE_TAG"   --push .
```

Deploy through Azure Bicep parameters:

```bash
docker pull --platform linux/amd64 "$ACR_LOGIN_SERVER/bizpulse:$IMAGE_TAG"
export CONTAINER_IMAGE=$(docker image inspect   "$ACR_LOGIN_SERVER/bizpulse:$IMAGE_TAG"   --format '{{index .RepoDigests 0}}')

DEPLOYMENT_NAME="bizpulse-release-$(date +%Y%m%d%H%M%S)"

az deployment group create   --resource-group "$RESOURCE_GROUP"   --name "$DEPLOYMENT_NAME"   --template-file infra/main.bicep   --parameters .tmp/liyi-test.bicepparam
```

Important: `.tmp/liyi-test.bicepparam` is a local deployment parameter file and should not be committed.

## 2.8 OpenAI / Key Vault Setup

Create or use an Azure Key Vault and store the OpenAI key as a secret. The current deployment uses a shared validated credential binding for both Ordinary Login AI and Public Demo AI.

High-level requirements:

1. Key Vault exists.
2. OpenAI key is stored as a secret.
3. Application managed identity has permission to read the secret.
4. AI is enabled in deployment parameters.
5. Admin Console enables the appropriate AI channel.

Validation:

1. Open Admin AI Management.
2. Confirm shared credential shows verified fingerprint.
3. Confirm Ordinary Login AI / Public Demo AI channel is enabled as needed.
4. Ask `Explain profit changes` in AI Decision Center.

## 2.9 Final Setup Validation Checklist

- [ ] `/health/ready` returns 200.
- [ ] Operator workspace loads.
- [ ] Data Workspace displays current published data.
- [ ] Dashboard pages show current dataset metrics.
- [ ] Upload workflow can reach Recognition, Mapping, Quality, Preview, and Publish.
- [ ] AI Decision Center returns an answered response for `Explain profit changes`.
- [ ] No secret values are committed to GitHub.
