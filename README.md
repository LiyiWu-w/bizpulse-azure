# BizPulse Azure

BizPulse is a full-stack business decision workspace for small e-commerce operators. It supports CSV/XLSX upload, deterministic data processing, dashboard analytics, action recommendations, and an AI explanation layer.

## Project Links

- GitHub Repository: https://github.com/LiyiWu-w/bizpulse-azure
- Live Azure Application: https://bizpulseliyi-app.mangohill-937cf0e0.eastus.azurecontainerapps.io
- Final Documentation: [docs/README.md](docs/README.md)
- Source Code: [bizpulse/](bizpulse/)

## What BizPulse Does

BizPulse helps operators turn fragmented business files into traceable dashboard insights.

Main workflow:

1. Upload CSV or Excel files
2. Recognize file type and fields
3. Confirm field mapping
4. Run data quality checks
5. Commit an immutable dataset version
6. Run deterministic calculations
7. Publish the verified dataset
8. View dashboards, action cards, and AI explanations

## Technology Stack

- Frontend: JavaScript / browser-based UI
- Backend: Python FastAPI
- Database: PostgreSQL
- File storage: Azure Blob-compatible storage
- Deployment: Docker + Azure Container Apps
- Container registry: Azure Container Registry
- Secrets: Azure Key Vault + Managed Identity
- AI: OpenAI API through a backend gateway

## Documentation

The final project documentation required for submission is in the [`docs/`](docs/) folder.

It includes:

- Production support and testing scenarios
- System setup instructions
- Issue diagnosis and resolution
- End-user usage guide
- Architecture diagram
- Screenshot evidence

## Important Notes

- PostgreSQL is the database engine. Neon, if used, is only a hosted PostgreSQL provider.
- Docker is used to package the application into a container image for Azure.
- AI does not calculate business numbers. It only explains published data and evidence.
- Demo uploads are temporary and do not replace the official published dashboard dataset.
