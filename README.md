# CodeDeployer - Auto Deployment Platform
This project is a lightweight platform that automates the deployment of frontend applications from Git repositories, inspired by **Vercel**. It streamlines the build and deployment process using AWS ECS, S3, Redis, and Express.
It supports:

* Git-based cloning and build
* Live deployment to a reverse proxy using subdomains
* Build logs via Redis and WebSockets
* AWS S3 for file storage
* AWS ECS Fargate for running dynamic build containers

---

## Features

* **API Server (Express)**:

  * Accepts Git repository URL and slug.
  * Triggers build container via AWS ECS.
  * Provides real-time build logs over WebSockets using Redis Pub/Sub.

* **Build Server (Dockerized)**:

  * Clones the project repo using a passed environment variable.
  * Builds the project (`npm install && npm run build`).
  * Uploads the `dist/` folder to AWS S3 under a namespaced path.

* **Reverse Proxy (Express + http-proxy)**:

  * Resolves incoming subdomain requests.
  * Serves content from `BASE_PATH/{slug}` using a proxy.

* **WebSocket Server (Socket.IO)**:

  * Exposes build logs to clients subscribed to project-specific Redis channels.

---

## Folder Structure

```
project-root/
├── api-server/        # Handles project creation and ECS task management
├── build-server/      # Docker image for building and uploading projects
├── reverse-proxy/     # Proxies requests to built projects
```

---

## Environment Variables

### API Server

* `ECS_REGION`
* `ECS_ACCESS_KEY`
* `ECS_SECRET_ACCESS_KEY`
* `CLUSTER` (ECS Cluster name)
* `TASK` (Task Definition ARN)
* `REDIS_HOST`, `REDIS_PORT`, `REDIS_USERNAME`, `REDIS_PASSWORD`

### Build Server (passed via ECS Task env overrides)

* `GIT_REPOSITORY__URL`
* `PROJECT_ID`
* `S3_REGION`, `S3_ACCESS_KEY`, `S3_SECRET_ACCESS_KEY`, `S3_BUCKET_NAME`
* `REDIS_*` same as above

### Reverse Proxy

* `BASE_PATH`: S3 public bucket URL prefix (e.g. `https://bucket-name.s3.amazonaws.com/__outputs`)

---

## Sample API Request

```bash
POST /project
Content-Type: application/json

{
  "gitURL": "https://github.com/user/repo.git",
  "slug": "my-project"
}
```

Returns:

```json
{
  "status": "queued",
  "data": {
    "projectSlug": "my-project",
    "url": "http://my-project.localhost:8000"
  }
}
```

---

## Setup

1. Deploy Redis instance
2. Configure AWS ECS Task with Docker image of `build-server`
3. Set up `reverse-proxy` to forward requests based on subdomain to S3 URL
4. Run all services:

   * API Server (port 9000)
   * WebSocket Server (port 9002)
   * Reverse Proxy (port 8000)

---

## Future Improvements

* Add DB layer for project metadata.
* Implement rate-limiting and authentication.
* Use CDN (CloudFront) for faster content delivery.
* Add retry logic and error monitoring (e.g., via Sentry).
