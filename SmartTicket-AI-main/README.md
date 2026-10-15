# Ticket Triage AI

Microservices backend that turns unstructured IT support tickets into structured triage data using **Spring Boot 3** and **Spring AI** (Google Gemini).

[![Java](https://img.shields.io/badge/Java-17-orange?style=flat&logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5-6DB33F?style=flat&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring AI](https://img.shields.io/badge/Spring%20AI-1.1-6DB33F?style=flat)](https://spring.io/projects/spring-ai)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white)](https://www.docker.com/)

---

## Overview

Submit a ticket title + description → receive structured JSON with **category**, **priority**, and a **draft reply** for support workflows.

Designed as a two-service architecture with an API gateway pattern for separation of concerns and containerized local deployment.

---

## Architecture

```
Client  →  Gateway (8080)  →  AI Service (8081)  →  Google Gemini API
              │                        │
              └── Spring WebClient ────┘
```

| Service | Port | Responsibility |
|---------|------|----------------|
| **gateway-service** | 8080 | Public REST API (`POST /api/triage`), routes to AI service via WebClient |
| **ticket-triage-ai** | 8081 | Spring AI integration, prompt engineering, JSON parsing |

---

## Tech Stack

- **Java 17** · **Spring Boot 3.5**
- **Spring AI** (Google GenAI / Gemini 2.5 Flash)
- **Spring WebClient** (async inter-service calls)
- **Docker & Docker Compose**
- **Jakarta Validation** · centralized exception handling

---

## API

### Gateway — `POST /api/triage`

**Request:**
```json
{
  "title": "VPN not connecting",
  "description": "User cannot connect to corporate VPN since morning. Error: timeout."
}
```

**Response:**
```json
{
  "category": "Network",
  "priority": "High",
  "draftReply": "Please verify your VPN client version and try reconnecting..."
}
```

### AI Service (internal) — `POST /triage`

Same request/response contract. Called by the gateway over the Docker network.

### AI Service — `GET /test?message=...`

Quick LLM connectivity check (development).

---

## Quick Start (Docker)

**Prerequisites:** Docker, Docker Compose, a [Google Gemini API key](https://aistudio.google.com/apikey)

```bash
# From repo root
export GEMINI_API_KEY=your_key_here   # Linux/Mac
# $env:GEMINI_API_KEY="your_key_here"  # PowerShell

docker-compose up --build
```

**Test:**
```bash
curl -X POST http://localhost:8080/api/triage \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Email down\",\"description\":\"Outlook not syncing since 9am for entire team.\"}"
```

---

## Local Development (without Docker)

**Terminal 1 — AI service:**
```bash
cd ticket-triage-ai
export GEMINI_API_KEY=your_key_here
./mvnw spring-boot:run
```

**Terminal 2 — Gateway:**
```bash
cd gateway-service
./mvnw spring-boot:run
```

Gateway defaults to `http://localhost:8081` for the AI service.

---

## Project Structure

```
Ticket-Triage/
├── docker-compose.yml          # Orchestrates both services
├── gateway-service/            # API gateway (port 8080)
│   ├── src/.../client/         # WebClient → AI service
│   └── Dockerfile
└── ticket-triage-ai/           # Spring AI service (port 8081)
    ├── src/.../service/        # TriageService, LlmTestService
    └── Dockerfile
```

---

## Environment Variables

| Variable | Service | Description |
|----------|---------|-------------|
| `GEMINI_API_KEY` | ai-service | Google Gemini API key |
| `AI_SERVICE_URL` | gateway | AI service base URL (default `http://ai-service:8081` in Docker) |

---

## Author

**Chandra Shekhar Dan** — Software Engineering Associate @ Amdocs

- [LinkedIn](https://www.linkedin.com/in/chandra-shekhar-dan/)
- [GitHub](https://github.com/ShekharDan)
- [LeetCode](https://leetcode.com/u/shekhardanchandra/)
