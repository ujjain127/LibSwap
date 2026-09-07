# LibSwap - SIT725 individual Docker submission

Name: **ujjainsriganesh**

Student ID: **226411987**

LibSwap is a library catalogue and book borrowing application. This submission runs the complete implemented application in Docker: HTML/CSS/JavaScript pages, Express APIs, JWT authentication, and a MongoDB database with persistent storage.

## Requirements

- Docker Desktop installed and running, including Docker Compose.
- Internet access for the initial image and package downloads.
- Port 3000 available, or choose another port as described below.

You do not need Node.js or MongoDB installed on your computer, an Atlas account, or credentials from the author.

## Start from a source copy

Clone this repository using its GitHub **Code** URL, or download and extract its ZIP. Open a terminal in the folder containing this README, Dockerfile, and docker-compose.yml. Run all commands below from that folder.

### 1. Generate private local configuration

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/workspace" -w /workspace node:22-bookworm-slim node scripts/docker-setup.js
```

The command above is for macOS and Linux. On Windows PowerShell, use:

```powershell
docker run --rm -v "${PWD}:/workspace" -w /workspace node:22-bookworm-slim node scripts/docker-setup.js
```

Alternatively, with Node.js installed, run `npm run docker:setup`. The Linux/macOS command writes configuration under your user account so Compose can read it.

This creates `.env.docker` with a randomly generated JWT signing secret and demo account password. It preserves the file if it already exists. Open this file locally and note `DEMO_PASSWORD` for the browser login step. No configuration values need to be obtained from OnTrack or the author.

### 2. Build and start the app and database

```sh
docker compose --env-file .env.docker up --build -d --wait
```

Wait until both `app` and `mongo` are healthy. The first build may take several minutes.

### 3. Create demo accounts and books

```sh
docker compose --env-file .env.docker exec -T app npm run seed:demo
```

This adds three books and two users without overwriting existing passwords, loans or reservations.

### 4. Open and verify the application

- Catalogue: http://localhost:3000/
- Borrow and reserve: http://localhost:3000/borrowBooks.html
- Return your loans: http://localhost:3000/returnBooks.html
- Student identity: http://localhost:3000/api/student
- Database health: http://localhost:3000/api/health

The identity endpoint returns `{"name":"ujjainsriganesh","studentId":"226411987"}`.

A healthy database returns `{"status":"ok","database":"connected"}` from `/api/health`.

## Demonstrate working database features

Use the generated `DEMO_PASSWORD` from `.env.docker` for both accounts:

| Account | Email |
| --- | --- |
| Reader | demo.reader@libswap.test |
| Reserver | demo.reserver@libswap.test |

1. On Borrow / Reserve, sign in as the reader and borrow The Hobbit.
2. Open My Returns and confirm the book is listed as borrowed by you.
3. Sign out, open Borrow / Reserve and sign in as the reserver.
4. Reserve The Hobbit. Your reservation position should be 1.
5. Reload the page. The reservation should remain.
6. Sign in as the reader and return the book from My Returns.
7. Sign in as the reserver and borrow the returned book. The first reservation has priority.
8. Return it from My Returns. The loan list should become empty.

The existing registration API is also tested by the automated suite below. All circulation requests use authenticated user identity; another user cannot return your loan.

## Run automated tests

```sh
docker compose --env-file .env.docker exec -T -e TEST_MONGODB_URI=mongodb://mongo:27017 app npm test
```

The suite tests real HTTP registration, login and database operations, simultaneous borrowing, duplicate reservations, ownership checks, invalid requests, queue order and page assets. It creates and removes only its own uniquely named test database; demo application data is preserved.

## Verify persistence

After borrowing and reserving in steps 1-4 above, run:

```sh
docker compose --env-file .env.docker down
docker compose --env-file .env.docker up -d --wait
```

Sign in again and verify the loan and reservation still exist. MongoDB stores data in the Compose named volume `mongo-data`. Do not add `-v` to `down`: that deletes the database volume.

## Stop or restart

Stop containers while retaining data:

```sh
docker compose --env-file .env.docker down
```

Start again:

```sh
docker compose --env-file .env.docker up -d --wait
```

## Runtime configuration and secrets

`.env.docker` is generated locally and excluded from Git and the Docker image. The native-development `.env` is also excluded. `.env.example` contains placeholders only.

- `JWT_SECRET`: generated locally; used to sign login tokens.
- `DEMO_PASSWORD`: generated locally; used when first creating demo users.
- `APP_PORT`: defaults to 3000. Change it in `.env.docker` if necessary.
- `MONGODB_URI`: supplied by Compose as `mongodb://mongo:27017/libswap`. It connects to the Docker MongoDB service, not the shared Atlas cluster.

Always include `--env-file .env.docker` in Compose commands. No excluded author-specific secret is required to run this submission. Do not share `.env.docker`, full expanded Compose configuration or login tokens in screenshots.

## Troubleshooting

- **Docker daemon unavailable:** start Docker Desktop and wait for it to finish starting.
- **Port already allocated:** set `APP_PORT=3001` in `.env.docker`, rerun the build/start command and use localhost:3001 for all links.
- **Missing JWT_SECRET or DEMO_PASSWORD:** complete setup step 1 and include `--env-file .env.docker`.
- **No books / login fails:** complete demo seeding, use the exact email above and DEMO_PASSWORD from the same `.env.docker`. Re-seeding preserves existing passwords; changing DEMO_PASSWORD after the first seed does not reset users.
- **Unhealthy container:** inspect status and app logs using the commands below. Database readiness can take time on first startup.

```sh
docker compose --env-file .env.docker ps
docker compose --env-file .env.docker logs --tail=50 app
```

## Container design

The app uses Node 22 with locked dependencies (`npm ci`) and runs as a non-root user. Express serves UI and APIs together. Compose starts MongoDB first and waits for its health check; the app health check confirms database connectivity. The database port is not published to the host; the app port is bound to localhost. The named volume retains data across container recreation. This is a local assessment deployment, not a production security configuration.

Further design and verification details are in `docs/DOCKER_SUBMISSION.md` and `evidence/DOCKER_VERIFICATION.md`. All instructions required to start and verify the application are included above.

## Fresh environment verification

A source ZIP was extracted into a new folder without .env files, node_modules or Git metadata. Configuration was generated using Docker, followed by a build without application cache and a separate Compose project with a new database volume. The test record is in evidence/FRESH_ENVIRONMENT.md. This tests source-package reproducibility on the same Docker Desktop host; it is not a claim of testing on another physical machine or an independently provisioned VM.

The workflow `.github/workflows/docker-smoke.yml` is provided to repeat setup, build, integration and persistence checks on a fresh GitHub-hosted Ubuntu runner. After publishing the repository, check the Actions tab and retain a successful run URL. Until such a run succeeds, independent-runner testing remains unverified. A tutor can also clone the public repository on another machine and follow the startup steps above without any private credentials.
