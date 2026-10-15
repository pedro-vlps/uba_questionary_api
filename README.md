# UBA Questionary API

> No momento o deploy desse projeto é inviável por questões financeiras e de usabilidade, o projeto funciona porém foi feito para estudantes de medicina da UBA (Universidad de Buenos Aires) e o uso não teria aplicabilidade real o suficiente para eu arcar com os custos então criei um projeto inspirado nesse porém para o público do ENEM ([https://github.com/pedro-vlps/enem_questionary_front](https://github.com/pedro-vlps/enem_questionary_front))

UBA Questionary is a study platform developed by [Pedro Vieira](https://github.com/pedro-vlps) for medical students at the Universidad de Buenos Aires. It generates anatomy and biology questions with AI, presents answer explanations, and records students' answers.

This repository contains the Python backend. The [React frontend](https://github.com/pedro-vlps/uba_questionary_front) completes the application.

## Engineering at a Glance

| Area | Implementation |
| --- | --- |
| API | Python 3.13, FastAPI, Uvicorn, Pydantic |
| Persistence | PostgreSQL 16, SQLAlchemy async sessions |
| AI generation | OpenAI Responses API with subject-specific prompts |
| Access control | JWT, institution membership, profiles, and question-package quotas |
| Billing | Stripe checkout and payment events that synchronize access |
| Tooling | Poetry, Docker Compose, Pytest, Coverage |

## Product Workflow

1. A student registers, signs in, and accesses the UBA study context.
2. Payment and package status determine access to question generation.
3. The student selects anatomy or biology and a topic.
4. The backend assembles a prompt with subtopic constraints and recent questions to reduce repetition, requests AI content, and persists the question.
5. The student submits an answer and can review previous answers through the application.

Account recovery, profile updates, student feedback, and administrative payment summaries support this workflow.

## Implementation Highlights

- **Domain-specific AI integration:** anatomy and biology prompts include topic, subtopic, answer-letter, diversity, and recent-question context.
- **Business-aware authorization:** protected routes combine authentication with institution and profile permissions.
- **Billing tied to access:** Stripe events synchronize subscription records and institution access; packages track question-generation usage.
- **Explicit API contracts:** Pydantic schemas define request and response data alongside SQLAlchemy models.
- **Automated verification:** dedicated suites cover routers, controllers, services, helpers, and permission behavior.

## Code Review Guide

| Start here | What to inspect |
| --- | --- |
| [Anatomy controller](src/controllers/ai_anatomy_controller.py) | Generation orchestration and persistence |
| [Anatomy AI service](src/services/ai_anatomy_service.py) | Prompt assembly and OpenAI integration |
| [Permission middleware](src/middleware/check_permissions.py) | Institution and profile access rules |
| [Subscription service](src/services/subscription_service.py) | Question usage and access state |
| [Stripe service](src/services/stripe_service.py) | Payment synchronization |
| [Infrastructure and permission tests](tests/test_infrastructure_and_permissions.py) | Boundary and failure cases |

## Architecture

The repository uses technical layers with domain-specific modules inside them:

```text
src/
  configs/        # settings and database connections
  controllers/    # request orchestration by domain
  databases/      # SQL schemas, migrations, and seeders
  helpers/        # prompt templates and permission definitions
  middleware/     # authentication and authorization
  models/         # relational models and generated route declarations
  routers/        # HTTP endpoints
  schemas/        # API data contracts
  services/       # domain operations and integrations
  utils/          # encryption, token, and input helpers
tests/            # automated test suites
docs/             # database diagram
```

Selected resource routes are generated through the author's [CRUD library](https://github.com/pedro-vlps/api_crud_generate_libary), declared in the models context.

## Local Setup

Requires Python 3.13, Poetry, PostgreSQL 16, and configured OpenAI and Stripe credentials for their respective workflows. Docker Compose is provided for the database and API.

Create a root `.env` using the settings contract in [configs.py](src/configs/configs.py):

```env
APP_ENV=DEV
POSTGRES_DB=uba_questionary
POSTGRES_USER=local_user
POSTGRES_PASSWORD=replace_with_local_password
POSTGRES_HOST=localhost
POSTGRES_PORT=5434
OPENAI_API_KEY=replace_with_openai_key
FERNET_KEY=replace_with_generated_fernet_key
JWT_SECRET_KEY=replace_with_random_secret
ALGORITHM=HS256
JWT_EXPIRATION_MINUTES=60
RESTRICT_STRIPE_AUTH_KEY=replace_with_stripe_test_key
PUBLIC_STRIPE_AUTH_KEY=replace_with_stripe_test_key
SECRET_STRIPE_AUTH_KEY=replace_with_stripe_test_key
WEBHOOK_STRIPE_SECRECT_KEY=replace_with_webhook_secret
DEFAULT_PRICE_ID=price_replace_me
PAYMENT_CURRENCY=ars
CHECKOUT_REDIRECT_URL=http://localhost:3000/app
PASSWORD_RESET_TOKEN_EXPIRATION_MINUTES=30
PASSWORD_RESET_INCLUDE_TOKEN_IN_RESPONSE=false
PASSWORD_RESET_URL=http://localhost:3000/reset-password
NICKNAME_RECOVERY_TOKEN_EXPIRATION_MINUTES=30
NICKNAME_RECOVERY_INCLUDE_TOKEN_IN_RESPONSE=false
NICKNAME_RECOVERY_URL=http://localhost:3000/recover-nickname
SMTP_ENABLED=false
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USE_TLS=false
SMTP_USE_SSL=false
SMTP_FROM_EMAIL=noreply@example.com
SMTP_FROM_NAME=UBA Questionary
SUPPORT_EMAIL=support@example.com
FRONTEND_ORIGINS=["http://localhost:3000"]
```

Generate a Fernet key after installing dependencies:

```bash
poetry install --no-root
poetry run python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

For a local API with the Compose database:

```bash
docker compose up -d db
poetry run uvicorn src.__main__:app --reload
```

Before starting Compose, create `src/databases/.cache` and ensure `PWD` resolves to the repository directory: the database volume uses that host path.

Apply [3.0_generate_db.sql](src/databases/scripts/3.0_generate_db.sql) to an empty database before using data-dependent routes. Review the separate [migrations](src/databases/scripts/migrations) and [seeders](src/databases/scripts/seeders) for your setup. Compose does not automatically initialize application tables.

To run both services in Compose, set `POSTGRES_HOST=db` and `POSTGRES_PORT=5432`, then run:

```bash
docker compose up --build
```

The API is served at `http://localhost:8000`. Swagger UI is at `/docs` and ReDoc at `/redoc`; both are disabled when `APP_ENV=P`.

Stripe workflows also require matching test-mode prices, webhook configuration, and the institution/profile records expected by the application. Email delivery requires enabling SMTP and supplying server credentials when applicable.

## Tests

```bash
poetry run pytest
poetry run task coverage
```

The suites use test doubles for database and external-service behavior. They do not establish live OpenAI, Stripe, or SMTP availability.

## Current Scope

No public deployment is maintained. This repository can be evaluated through its source, tests, and local setup.

The existing user-storage implementation uses Fernet encryption, including for passwords, and hashed lookup fields for email, nickname, and DNI. This is an implementation detail to review when evaluating the project, rather than a claim of production security.

The [ENEM project](https://github.com/pedro-vlps/enem_questionary_front) carries the study-platform idea forward for a broader Brazilian exam-preparation audience.
