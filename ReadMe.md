# Feature Branch CI/CD Pipeline

This repository includes a GitHub Actions workflow for **linting**, **testing**, **building**, and **deploying** feature branches to a secure Azure environment.

---

## Workflow Overview

### Trigger Conditions
The workflow runs on:
- **Pushes** to branches matching:  
feature/**

markdown
Copy
Edit
- **Manual triggers** via `workflow_dispatch`.

---

## Environment Variables

| Variable                          | Description                                                                 |
|-----------------------------------|-----------------------------------------------------------------------------|
| `NODE_VERSION`                    | Node.js version to use for build/test.                                      |
| `AZURE_WEBAPP_PACKAGE_PATH`       | Root path for Azure deployment.                                             |
| `AZURE_CONTAINER_REGISTRY`        | Name of the Azure Container Registry (ACR).                                 |
| `CONTAINER_APP_NAME`              | Name of the containerized application.                                      |
| `AZURE_WEBAPP_PUBLISH_PROFILE`    | Azure publish profile (stored as GitHub secret).                            |
| `AZURE_REGISTRY_USERNAME`         | ACR username (secret).                                                      |
| `AZURE_REGISTRY_PASSWORD`         | ACR password (secret).                                                      |
| `AZURE_SUBNET_ID`                 | Azure subnet ID for private networking (secret).                            |
| `AZURE_RESOURCE_GROUP`            | Azure Resource Group (secret).                                              |

---

## Jobs

### 1. **Lint**
- **Purpose**: Ensures code style and syntax meet project standards.
- **Runs**:  
- Installs dependencies via `npm ci`.
- Executes `npm run lint:check`.
- **Condition**: Skips if commit message contains `ci skip`.

---

### 2. **Test**
- **Depends on**: `lint` job.
- **Purpose**: Runs unit tests to validate functionality.
- **Runs**:
- Installs dependencies via `npm install`.
- Runs `npm run test:unit -- --no-watch`.

---

### 3. **Build**
- **Depends on**: `lint` job.
- **Purpose**: Builds a Docker image for the application and pushes it to Azure Container Registry.
- **Environment variable**:  
- `DOCKER_FILE_EXT` is set to `.test` to use the `dockerfile.test`.
- **Steps**:
1. Logs into Azure Container Registry.
2. Builds and pushes the image with tag:  
   ```
   <ACR_NAME>.azurecr.io/<APP_NAME>:<GITHUB_SHA>
   ```
3. Uses `infra/docker/Dockerfile.test` for the build.

---

### 4. **Deploy Secured**
- **Depends on**: `build`, `test`.
- **Purpose**: Deploys container to **Azure Container Instances (ACI)** in a private subnet.
- **Steps**:
1. Logs in to Azure using the publish profile.
2. Prepares a container deployment YAML file from template:
   - Replaces placeholders in `infra/azure/container-template.yaml` with actual values using `envsubst`.
3. Deploys to ACI using:
   ```sh
   az container create --resource-group <resource-group> --file container-instances-vnet.yaml
   ```

---

## Dockerfile Explanation (`infra/docker/Dockerfile.test`)

### Stage 1: Build
- **Base image**: `node:18.20-alpine`
- Sets working directory to `/dist/src/app`
- Copies entire project into container
- Installs dependencies and runs `npm run build`

### Stage 2: Serve
- **Base image**: `nginx:latest`
- Copies the built app from the first stage into Nginx's web root
- Uses `nginx.test.conf` for server configuration
- Exposes port `80` for HTTP traffic

---

## CI/CD Flow Diagram

```mermaid
flowchart TD
  A[Push to feature/** branch] --> B[Lint Job]
  B --> C[Test Job]
  B --> D[Build Job]
  C --> E[Deploy Secured]
  D --> E
  E --> F[Azure Container Instances]
Skipping CI
To skip CI for a commit, include ci skip in your commit message:
 ```
```sh
git commit -m "Minor docs update [ci skip]"
 ```
Manual Trigger
You can trigger this workflow manually from the Actions tab in GitHub.


