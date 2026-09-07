# SIT725 individual Docker submission — 226411987

## Scope

LibSwap's complete implemented application runs in Docker: Express serves the HTML/CSS/JavaScript catalogue, borrow/reserve and return pages, JWT authentication APIs and MongoDB-backed circulation. MongoDB runs in a second container with a named volume. The shared Atlas cluster is not required or modified by this setup.

The complete assessment brief requires an individual public repository, full end-to-end Docker deployment, a name and student ID endpoint, complete main README instructions, and one PDF containing screenshots and a personal reflection of no more than 500 words. The intended individual GitHub account is ujjain127. Publication and a fresh public clone still need verification.

## Start from a clean checkout

Prerequisite: Docker Desktop running with Docker Compose. Host Node is optional if you use the alternative setup below.

```sh
cd /Users/ujjains/LibSwap
npm run docker:setup
docker compose --env-file .env.docker up --build -d --wait
docker compose --env-file .env.docker exec -T app npm run seed:demo
```

If Node is not installed on the host, generate settings using the same Node image:

```sh
docker run --rm -v "$PWD:/workspace" -w /workspace node:22-bookworm-slim node scripts/docker-setup.js
```

Open http://localhost:3000/ for the catalogue, http://localhost:3000/borrowBooks.html to borrow/reserve, and http://localhost:3000/returnBooks.html to return. Identity: http://localhost:3000/api/student. Health: http://localhost:3000/api/health.

The setup command creates `.env.docker` once with random JWT and demo-password values. It does not overwrite existing settings. Sign in as `demo.reader@libswap.test` or `demo.reserver@libswap.test` using `DEMO_PASSWORD` from that private file. Demo seeding creates three books and two accounts; repeating it preserves existing accounts, passwords, loans and reservations. Keep this demo database separate from real team data.

If port 3000 is occupied, set APP_PORT=3001 in `.env.docker`, rerun the up command and use localhost:3001. Always pass `--env-file .env.docker`; the native `.env` contains unrelated Atlas settings.

## Design decisions

- **Node 22 / Debian slim:** compatible with the existing Mongoose 9 dependency; replaces the incompatible Node 18 base.
- **Locked dependencies:** npm ci installs package-lock.json exactly, with production dependencies only. No new application packages were added.
- **Non-root app:** the image runs under the built-in node user. An init process forwards signals and reaps children.
- **Two services:** the app reaches MongoDB at `mongo:27017` through Compose DNS. `localhost` inside the app would refer to the app container itself.
- **Database readiness:** MongoDB's health check pings the database; the app starts after it becomes healthy. `/api/health` verifies a live database connection, rather than only an HTTP process.
- **Persistence:** `mongo-data` mounts at `/data/db`, so records survive container restart and recreation.
- **Local exposure:** only app port 3000 is published, bound to the host loopback address. MongoDB has no published host port. This is a local assessment setup; MongoDB authentication/TLS and managed secret storage would be needed for a broader deployment.
- **Runtime configuration:** secrets are injected when containers start. `.dockerignore` excludes `.env` files, Git history, host dependencies and evidence. `.env.example` contains placeholders only.
- **Shutdown:** the app handles SIGTERM/SIGINT, stops accepting requests and closes its database connection.
- **Identity:** `/api/student` returns `{"name":"ujjainsriganesh","studentId":"226411987"}` and every implemented page displays the ID. This matches the identity fields required by the supplied brief.

## Verification commands

```sh
docker compose --env-file .env.docker config --quiet
docker compose --env-file .env.docker ps
docker compose --env-file .env.docker logs --tail=30 app
curl http://localhost:3000/api/health
curl http://localhost:3000/api/student
docker compose --env-file .env.docker exec -T -e TEST_MONGODB_URI=mongodb://mongo:27017 app npm test
```

The npm test suite uses its own uniquely named test database and removes only that database. It verifies registration/login, authentication failures, original-schema book records, simultaneous borrows, duplicate reservations, ownership checks, queue order, return cycles, private-field exclusions and served assets.

For a dedicated persistence demonstration, run this on a separate fresh project/port to avoid changing ongoing demo loans:

```sh
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check up --build -d --wait
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check exec -T app npm run seed:demo
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check exec -T app node scripts/docker-e2e.js before
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check down
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check up -d --wait
APP_PORT=3010 docker compose --env-file .env.docker -p libswap-hd-check exec -T app node scripts/docker-e2e.js after
```

The before phase borrows The Hobbit and reserves it for the second demo user. The after phase verifies both records survived container recreation, then returns, borrows in queue order, and returns the book. Do not use `down -v`: it deletes the database volume and defeats the persistence demonstration.

## Browser evidence to capture

1. Your individual GitHub repository showing the Dockerfile, Compose file and your commits.
2. Clean build output and `docker compose ps` with both services healthy.
3. `/api/student` displaying 226411987 and the visible ID in the application.
4. Reader signs in and borrows a book; it appears on My Returns.
5. Reserver signs in and reserves that unavailable book; queue position 1 is displayed.
6. Recreate containers without removing volumes; the loan and reservation remain.
7. Reader returns; reserver can borrow; reserver returns successfully.
8. Automated test output and persistence test output.

Capture real outputs from your run. Exclude .env contents, tokens and passwords from screenshots. Explain the actions in your own words and follow any exact screenshot/video limits in the full brief.

## Personal reflection prompts

Write the required reflection yourself using actual experiences. The reflection must be no more than 500 words.

- Why did you choose app + MongoDB containers instead of requiring the team's Atlas IP access?
- How did you discover and resolve Node/Mongoose compatibility and container database addressing?
- What did health checks and named volumes change about reliability?
- Which test proved that a reservation survives container recreation?
- What would you improve for a production deployment?

Do not claim independent work or unaided implementation beyond what actually occurred. Follow your unit's rules on acknowledging assistance.

## Submission status and Git

Docker changes are currently uncommitted alongside the borrow feature. Review and commit the feature first where practical, then create a Docker branch and commit the Docker changes separately. The student must submit an individual repository in their own GitHub account; repository ownership has not been verified.

Docker-specific files: Dockerfile, docker-compose.yml, .dockerignore, .env.example, scripts/docker-setup.js, scripts/seed-demo.js, scripts/docker-e2e.js, docs/DOCKER_SUBMISSION.md and evidence/DOCKER_VERIFICATION.md. Shared files also changed: server.js, package.json, public/catalogue.html, public/borrowBooks.html and public/returnBooks.html.

Before committing, inspect `git diff` and ensure the tracked `.env.example` contains no credentials. `.env` and `.env.docker` must remain ignored. Do not run or share `docker compose config` without `--quiet`, since expanded output includes secrets.
