# BizPulse Documentation

**Project:** BizPulse Local / BizPulse Azure  
**Documentation date:** 2026-08-19  
**Purpose:** This `/docs` folder supports production operations, setup, testing, troubleshooting, usage, and architecture documentation for the BizPulse full-stack application.

## Table of Contents

1. [Production Support and Testing Scenarios](01-production-support-and-testing.md)
2. [System Setup Instructions](02-system-setup.md)
3. [Issue Diagnosis, Research, Resolution, and Sharing](03-issue-diagnosis-and-resolution.md)
4. [System Usage Guide](04-system-usage-guide.md)
5. [Architecture Diagram](05-architecture-diagram.md)

## Screenshot Evidence

Screenshots used by this documentation are stored in [`screenshots/`](screenshots/). They show the operator import workflow, dashboards, calculation/publish state, and AI Decision Center.

## Important Project Notes

- BizPulse uses **PostgreSQL** as the database engine. Neon, if used, is a hosted PostgreSQL provider rather than a separate database type.
- The deployed application runs on **Azure Container Apps**.
- **Docker** is used to package the frontend, FastAPI backend, dependencies, and runtime into a container image.
- **AI is an explanation layer only.** Deterministic business calculations are performed by backend services, not by AI.
- Demo Viewer uploads are temporary demo artifacts and do not replace the current dashboard dataset. Formal data processing must be performed through the Operator workflow.
