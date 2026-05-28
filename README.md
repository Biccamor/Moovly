# MovieNight — AI-Powered Group Movie Recommendation Engine

MovieNight is a backend API that solves a surprisingly hard problem: finding a movie that an entire group actually wants to watch. It aggregates individual preferences, mood signals, and constraints from multiple users into a single intelligent recommendation session powered by semantic search and the Groq inference API.

---

## Architecture Overview

```
User Preferences → Vibe Aggregation → Prompt Construction
       → BGE-M3 Embedding → pgvector Hybrid Search
           → FlashRank Reranking → Groq LLM Decision → Response
```

The recommendation pipeline runs in six stages:

1. **Preference Aggregation** — Collects each user's vibes and maps them via `VIBE_MAP` to genre frequencies and descriptive keywords. Runtime constraints and era filters are merged across the group.
2. **Prompt Construction** — Builds a structured natural-language prompt from the aggregated group profile.
3. **Vector Embedding** — Encodes the prompt into a 1024-dimensional vector using `BGE-M3` via `FlagEmbedding`.
4. **Hybrid Search** — Queries PostgreSQL + pgvector using combined vector similarity and metadata filters.
5. **Reranking** — Applies `FlashRank` (`ms-marco-MiniLM-L-12-v2`) to reorder candidates against the prompt.
6. **LLM Decision** — Passes top candidates to **Groq API** (Llama 3 8B) for final selection with structured reasoning. Groq's inference speed eliminates the latency bottleneck of local model hosting.

---

## Tech Stack

| Layer            | Technology                               |
|------------------|------------------------------------------|
| Framework        | FastAPI (Python 3.11)                    |
| Database         | PostgreSQL 17 + pgvector                 |
| ORM              | SQLModel + SQLAlchemy                    |
| Embeddings       | FlagEmbedding (BGE-M3), FlashRank        |
| LLM              | Groq API (Llama 3 8B)                    |
| Auth             | JWT (PyJWT) + Argon2 password hashing    |
| Containerization | Docker + Docker Compose                  |

---

## Key Features

**Group-aware recommendation engine**
Users join a shared session via invite code. Each user submits their own vibe preferences, hard exclusions, and runtime constraints. The engine merges all inputs and reasons over them as a single group profile.

**Vibe system**
Instead of genre dropdowns, users express mood through semantic labels (`PIZZA_CHILL`, `DATE_NIGHT`, `MIND_BENDER`, etc.). Each vibe maps to weighted genre frequencies and descriptive keywords used to construct the embedding prompt.

**Hybrid semantic search**
Movie catalogue is pre-embedded at 1024 dimensions. Queries combine pgvector cosine similarity with metadata filters (era, runtime, content exclusions) for precision retrieval.

**Continuous Integration (GitHub Actions)**
Every push triggers an automated pipeline with three stages:
- **Linting** — enforces code style and formatting standards
- **Security scanning** — Bandit static analysis for common Python vulnerabilities
- **Regression testing** — Pytest suite with build caching for optimized execution times

**Cloud-ready deployment**
The application is containerized and structured for Continuous Deployment to **Google Cloud Platform (GCP)**. Docker Compose handles local orchestration; the same container setup is the deployment target for cloud-native hosting.

**Security**
- Argon2 password hashing (Password Hashing Competition winner)
- JWT access tokens with refresh token rotation
- Per-endpoint rate limiting (strict limits on heavy LLM routes)
- Non-root Docker container for runtime isolation

---

## Project Structure

```
movieNight/
├── main.py                        # App entry point, lifespan, middleware
├── compose.yaml                   # Docker Compose (API + PostgreSQL)
├── routers/
│   ├── auth_router.py             # Register, login, token refresh
│   ├── recommendation_router.py   # Session creation & AI recommendations
│   ├── metadata_router.py         # Available vibes and eras
│   └── preference_router.py       # Save user preferences
├── engine/
│   ├── recommendation_service.py  # Full recommendation pipeline orchestration
│   ├── vector.py                  # Embedding creation & hybrid search
│   ├── llm_decider.py             # LLM-powered final selection
│   └── prompts.py                 # Vibe mappings & prompt templates
├── database/
│   ├── main_db.py                 # Engine setup & session management
│   ├── database_setup.py          # SQLModel table definitions
│   └── get_movies.py              # Movie data ingestion
└── schemas/
    └── schemas.py                 # Pydantic request/response models
```

---

## Getting Started

### Prerequisites

- Docker and Docker Compose
- Groq API key (free tier available at [console.groq.com](https://console.groq.com))

### Setup

```bash
# Clone the repository
git clone https://github.com/your-username/movieNight.git
cd movieNight

# Configure environment
cp .env.example .env
# Edit .env with your values

# Start all services
docker compose up --build
```

**Environment variables:**
```env
DATABASE_URL=postgresql://user:password@db:5432/movienight
GROQ_API_KEY=your-groq-api-key
SECRET_KEY=your-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE=25
```

The Compose file starts two services:
- PostgreSQL 17 with pgvector — port `5433`
- MovieNight API — port `8010` with hot reload

API docs available at `http://localhost:8010/docs`.

---

## API Reference

### Authentication — `/auth`

| Method | Endpoint         | Description                  | Rate Limit |
|--------|------------------|------------------------------|------------|
| POST   | `/auth/register` | Create a new account         | 5/min      |
| POST   | `/auth/login`    | Login and receive JWT token  | 10/min     |
| POST   | `/auth/refresh`  | Rotate refresh token         | 5/min      |

### Recommendations — `/recommendation`

| Method | Endpoint                  | Description                           | Rate Limit |
|--------|---------------------------|---------------------------------------|------------|
| POST   | `/recommendation/session` | Create session and save preferences   | 20/min     |
| POST   | `/recommendation/{id}`    | Run AI recommendation pipeline (heavy)| 2/min      |

**Example session request:**
```json
{
  "invite_code": "XJ79B",
  "meeting_type": "EKIPA",
  "users": [
    {
      "user_id": "uuid-here",
      "user_name": "Alice",
      "personal_vibe": {
        "vibes": ["LAUGH_RIOT", "PIZZA_CHILL"],
        "hard_nos": ["GORE"],
        "max_runtime": 120,
        "allow_seen": false,
        "eras": ["90s", "2000s"]
      }
    }
  ]
}
```

### Metadata — `/metadata`

| Method | Endpoint                        | Description                        |
|--------|---------------------------------|------------------------------------|
| GET    | `/metadata/preferences-options` | Returns all available vibes & eras |

---

## Database Schema

| Table          | Description                                              |
|----------------|----------------------------------------------------------|
| `app_user`     | User accounts with hashed password and taste vector      |
| `movie`        | Movie catalogue with metadata and semantic embeddings    |
| `room_session` | Group session with aggregated preference state           |
| `rating`       | Per-user movie ratings (`-1` dislike, `0` seen, `1` like)|

---

## License

Proprietary software. See [LICENSE.md](./LICENSE.md) for details.  
© 2026 Fabian Bicca Moraes. All rights reserved.
