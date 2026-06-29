# 🚪 OrganiStation API Gateway

The **API Gateway** is a Node.js Express service serving as the single, unified entry point for all client requests entering the OrganiStation microservice cluster. It handles request logging, cross-origin resource sharing (CORS), JSON Web Token (JWT) verification, user role header injection, and dynamic reverse-proxying to backend services.

---

## ✨ Key Features

- **Centralized Authentication**: Validates all incoming API requests (except specified public routes) using JWT verification.
- **Header Injection & Forwarding**: Decodes validated JWT payloads and injects user identity into HTTP request headers (`X-User-Email`, `X-User-Role`, `X-User-Permissions`) before forwarding them to downstream services.
- **Forced Password Update Interceptor**: Restricts API access for users with the `must_change_password` flag active. They are only allowed to call `/api/auth/me`, `/api/auth/change-password`, or `/api/auth/logout` until their password is changed.
- **Dynamic Reverse-Proxying**: Routes requests to the appropriate backend microservice based on URL path matching using `http-proxy-middleware`.
- **Static Assets Host**: Serves the frontend static bundle from the `/public` folder when running in a unified container.
- **Robust Error Handling**: Captures downstream connection failures (e.g. `ECONNRESET`, `ETIMEDOUT`) and returns structured JSON responses rather than raw HTML to protect frontend API clients.

---

## 🛠️ Technology Stack

- **Runtime**: Node.js
- **Server Framework**: Express
- **Proxy Engine**: `http-proxy-middleware`
- **Token Handling**: `jsonwebtoken`
- **CORS Handling**: `cors`

---

## 📂 Proxy Routing Table

All requests sent to `/api/*` are captured by the gateway and routed based on the path suffix:

| Incoming Path Suffix | Target Service | Default Local URL | Path Rewrite Rule |
| :--- | :--- | :--- | :--- |
| `/api/auth/*` | **Auth Service** | `http://localhost:8001` | *None* |
| `/api/users/*` | **Auth Service** | `http://localhost:8001` | *None* |
| `/api/roles/*` | **Auth Service** | `http://localhost:8001` | *None* |
| `/api/permissions/*` | **Auth Service** | `http://localhost:8001` | *None* |
| `/api/ai/*` | **AI Service** | `http://localhost:8000` | `/api/ai/*` ➡️ `/api/*` |
| `/api/hr/*` | **HR Service** | `http://localhost:8002` | `/api/hr/*` ➡️ `/api/*` |
| `/api/projects/*` | **Project Service** | `http://localhost:8003` | `/api/projects/*` ➡️ `/api/*` |
| `/api/tickets/*` | **Project Service** | `http://localhost:8003` | `/api/tickets/*` ➡️ `/api/*` |
| `/api/finance/*` | **Finance Service** | `http://localhost:8004` | `/api/finance/*` ➡️ `/api/*` |

*Note: Ingestion operations in the AI Service are allowed up to a 3-minute timeout.*

---

## ⚙️ Configuration & Environment Variables

Create a `.env` file in the root of the `gateway` directory (you can copy `.env.example` as a template).

| Variable | Description | Default | Required |
| :--- | :--- | :--- | :--- |
| `PORT` | Service port | `3000` | No |
| `JWT_SECRET` | Secret key to verify JWT tokens | *Secret key* | Yes |
| `AUTH_SERVICE_URL` | Authentication service endpoint | `http://localhost:8001` | No |
| `AI_SERVICE_URL` | AI/RAG service endpoint | `http://localhost:8000` | No |
| `HR_SERVICE_URL` | HR service endpoint | `http://localhost:8002` | No |
| `PROJECT_SERVICE_URL` | Project management service endpoint | `http://localhost:8003` | No |
| `FINANCE_SERVICE_URL` | Finance service endpoint | `http://localhost:8004` | No |

---

## 💻 Local Development

### 1. Install Dependencies
```bash
npm install
```

### 2. Start Nodemon Dev Server
```bash
npm run dev
```
The gateway will start listening on `http://localhost:3000`.

---

## 🐳 Docker Deployment

To build and run the API gateway inside a Docker container:

```bash
# Build the Image
docker build -t organistation-gateway .

# Run the Container
docker run -d \
  -p 3000:3000 \
  --env-file .env \
  organistation-gateway
```
