# Deploying Dify on Dokploy

This guide outlines the step-by-step workflow for deploying and hosting Dify on your Dokploy server (`dokploy.mediajungle.at`) using the secure domain `dify.mediajungle.at`.

The current setup utilizes **Docker Named Volumes** to avoid host permission/existence issues and relies on **Git-based Compose deployment** to ensure all Nginx and SSRF Proxy configurations are correctly pulled.

---

## Deployment Workflow

### Step 1: Push Your Changes to Git
Ensure your modified `docker-compose.yaml` (configured with Named Volumes) and your changes are committed and pushed to your git repository (e.g., GitHub, GitLab, or self-hosted Git):
```bash
git add docker/docker-compose.yaml docker/docker-compose-template.yaml docker/.env
git commit -m "Configure Dify with Docker Named Volumes and production environment settings"
git push origin main
```

---

### Step 2: Create a Compose Application in Dokploy
1. Log in to your Dokploy panel at `https://dokploy.mediajungle.at`.
2. Select your project (or create a new one).
3. Click on **Create Service** (or **Create Application**) and select **Compose**.
4. Give your application a name (e.g., `dify`).

---

### Step 3: Configure Git Integration
1. Inside the newly created Compose application, go to the **Sources** tab.
2. Choose **Git** as the provider.
3. Select your repository and the appropriate branch (e.g., `main`).
4. Set the **Compose Path** to `docker/docker-compose.yaml`.
5. Click **Save**.

*Note: Deploying via Git is essential because Nginx templates (`docker/nginx/...`) and SSRF proxy configuration files (`docker/ssrf_proxy/...`) are pulled directly from your repository. Host bind paths for these files will resolve correctly relative to the project directory cloned by Dokploy.*

---

### Step 4: Configure Environment Variables
1. Go to the **Environment** tab of the Compose app in Dokploy.
2. Open your local `docker/.env` file.
3. Copy all key-value pairs and paste them into Dokploy's bulk environment variables editor.
4. Verify the following critical production-ready variables are set correctly:
   - `SECRET_KEY=mediajungle_secure_dify_key_2026_xyz123` (No duplicate prefix)
   - `APP_WEB_URL=https://dify.mediajungle.at`
   - `APP_API_URL=https://dify.mediajungle.at`
   - `FILES_URL=https://dify.mediajungle.at`
   - `ENDPOINT_URL_TEMPLATE=https://dify.mediajungle.at/e/{hook_id}`
   - `NEXT_PUBLIC_SOCKET_URL=wss://dify.mediajungle.at` (Ensure `wss://` for secure WebSockets over HTTPS)
   - `NGINX_HTTPS_ENABLED=false` (SSL termination will be handled by Dokploy's Traefik proxy)
5. Save the environment settings.

---

### Step 5: Configure Domain and Traefik Routing
Dokploy uses Traefik as its reverse proxy, which automatically handles Let's Encrypt SSL certificates.
1. Navigate to the **Domains** tab within your Dokploy Compose application.
2. Click **Add Domain**.
3. Configure the domain settings:
   - **Domain**: `dify.mediajungle.at`
   - **Service**: Select `nginx` (Dify's main gateway service)
   - **Port**: `80` (The container's internal port Nginx listens to)
4. Enable **HTTPS / SSL** to ensure automatic certificate provisioning and secure routing.
5. Click **Save**.

---

### Step 6: Deploy Dify
1. Go to the main dashboard or **Deploy** tab of your Compose application in Dokploy.
2. Click **Deploy**.
3. Dokploy will pull your repository, provision the Docker Named Volumes (`dify_db_data`, `dify_redis_data`, etc.), launch all Dify microservices, and configure Traefik to securely route incoming traffic from `https://dify.mediajungle.at` to Dify's internal `nginx` service.

You can monitor the deployment logs and check the build status directly in the Dokploy panel. Once completed, Dify will be fully accessible and persistent under your domain!
