---
title: JWT Authentication API
description: A REST API built in Go implementing JWT-based authentication and role-based access control. 
liveUrl: https://example.com
githubUrl: https://github.com/SantiagoBonillaGuevara/GoApi
image: {
url: "/lambda.webp",
alt:  "lambda thumbnail"
}
---

# 🔐 JWT Auth API — Go + AWS Lambda

REST authentication API with JWT built in Go, optimized for minimal cold start times on AWS Lambda.

---

## 📋 Description

This project implements a **JWT (JSON Web Tokens)**-based authentication system with role-based access control. It is designed to run as an **AWS Lambda** function, leveraging Go's one of the lowest cold start times available (~5–15 ms), making it ideal for serverless workloads with critical latency requirements.

---

## ⚡ Why Go on Lambda for Fast Cold Starts?

| Runtime       | Typical Cold Start |
|---------------|--------------------|
| **Go**        | ~5–15 ms           |
| Node.js       | ~100–200 ms        |
| Python        | ~100–300 ms        |
| Java          | ~500–1000 ms       |
| .NET          | ~200–500 ms        |

Go compiles to a **native binary** with no JVM or interpreter, eliminating initialization overhead. Combined with Lambda, the result is an extremely fast API from the very first invocation.

---

## 🏗️ Architecture

```
HTTP Client
    │
    ▼
AWS API Gateway
    │
    ▼
AWS Lambda (Go binary)
    │
    ├── POST /users/login   → Generates JWT
    ├── GET  /users/a       → Requires role A
    ├── GET  /users/b       → Requires role B
    └── GET  /users/c       → Requires role C
```

---

## 📁 Project Structure

```
.
├── main.go          # Entry point, handlers, middleware, and routes
├── go.mod           # Module and dependencies
├── go.sum           # Dependency hashes
└── README.md
```

---

## 🔧 Dependencies

| Package | Version | Usage |
|---------|---------|-------|
| `github.com/gorilla/mux` | v1.8.1 | HTTP Router |
| `github.com/dgrijalva/jwt-go` | v3.2.0 | JWT generation and validation |

> **Note:** For production, it is recommended to migrate from `dgrijalva/jwt-go` to [`golang-jwt/jwt`](https://github.com/golang-jwt/jwt), as the former is no longer actively maintained.

---

## 🚀 Installation and Local Execution

### Prerequisites

- Go 1.24.1+
- AWS CLI (for deployment)

### Clone and run

```bash
git clone <repo-url>
cd api-go

# Download dependencies
go mod tidy

# Run locally
go run main.go
# → Server running on :8080
```

---

## 📡 Endpoints

### `POST /users/login`

Authenticates the user and returns a JWT.

**Request:**
```json
{
  "name": "userA",
  "password": "userA"
}
```

**Response `200 OK`:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Errors:**
- `400 Bad Request` — invalid request body
- `401 Unauthorized` — incorrect credentials

---

### `GET /users/a` · `GET /users/b` · `GET /users/c`

Role-protected endpoints. Require the `Authorization: Bearer <token>` header.

**Available roles:**

| Endpoint    | Required Role |
|-------------|---------------|
| `/users/a`  | `A`           |
| `/users/b`  | `B`           |
| `/users/c`  | `C`           |

**Headers:**
```
Authorization: Bearer <JWT>
```

**Response `200 OK`:**
```
Only role A can access here
```

**Errors:**
- `401 Unauthorized` — missing or invalid token
- `403 Forbidden` — incorrect role

---

## 🔑 JWT Structure

```json
{
  "sub":   "userA",
  "roles": "A",
  "iss":   "api-go",
  "iat":   1710000000,
  "exp":   1710003600
}
```

The token expires **1 hour** after issuance. Signed with **HMAC-SHA256**.

---

## ☁️ Deployment on AWS Lambda

### 1. Build for Lambda (Linux AMD64)

```bash
GOOS=linux GOARCH=amd64 go build -o bootstrap main.go
zip function.zip bootstrap
```

> The binary is named `bootstrap` to use AWS's `provided.al2` runtime.

### 2. Create the Lambda Function

```bash
aws lambda create-function \
  --function-name jwt-api-go \
  --runtime provided.al2 \
  --role arn:aws:iam::<ACCOUNT_ID>:role/<LAMBDA_ROLE> \
  --handler bootstrap \
  --zip-file fileb://function.zip
```

### 3. Configure API Gateway

Connect AWS API Gateway (HTTP API or REST API) to the Lambda function to expose the endpoints publicly.

### 4. Recommended Environment Variables

| Variable | Description |
|----------|-------------|
| `SECRET_KEY` | Secret key for signing JWTs (do not hardcode in source) |

> ⚠️ In production, move `secretKey` to an environment variable or AWS Secrets Manager.

---

## 👥 Test Users

| User  | Password | Role |
|-------|----------|------|
| userA | userA    | A    |
| userB | userB    | B    |
| userC | userC    | C    |

> ⚠️ These users are hardcoded. In production, replace them with a database or an Identity Provider (Cognito, Auth0, etc.).

---

## 🔒 Security Considerations

- [ ] Rotate `SECRET_KEY` and externalize it (env var / Secrets Manager)
- [ ] Migrate to `golang-jwt/jwt` (maintained fork)
- [ ] Replace hardcoded users with a real data source
- [ ] Enable HTTPS (API Gateway handles this automatically)
- [ ] Consider refresh tokens for long-lived sessions
- [ ] Add rate limiting in API Gateway

---

## 📄 License

MIT