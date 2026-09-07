# LibSwap Docker Submission

**Name:** Ujjain Sri Ganesh
**Student ID:** 226411987

LibSwap is a library catalogue with login, borrowing, reservations, returns, Express APIs, and MongoDB storage.

## Requirements

* Docker Desktop
* Docker Compose
* Port `3000` available

Node.js and MongoDB are **not required** on the host.

## Start the Application

Run these commands from the repository folder:

```bash
docker run --rm --user "$(id -u):$(id -g)" \
  -v "$PWD:/workspace" -w /workspace \
  node:22-bookworm-slim node scripts/docker-setup.js

docker compose --env-file .env.docker up --build -d --wait

docker compose --env-file .env.docker exec -T app npm run seed:demo
```

## Application URLs

Open:

* http://localhost:3000/
* http://localhost:3000/borrowBooks.html
* http://localhost:3000/returnBooks.html
* http://localhost:3000/api/student
* http://localhost:3000/api/health

The `/api/student` response is:

```json
{
  "name": "ujjainsriganesh",
  "studentId": "226411987"
}
```

## Demo Accounts

Read `DEMO_PASSWORD` from `.env.docker`.

```text
demo.reader@libswap.test
demo.reserver@libswap.test
```

The reader can borrow **The Hobbit**. The reserver can reserve it, then borrow it after the reader returns it.

## Tests

Run the tests with:

```bash
docker compose --env-file .env.docker exec -T \
  -e TEST_MONGODB_URI=mongodb://mongo:27017 app npm test
```

The tests cover:

* Login
* Borrowing
* Reservations
* Returns
* Queue priority
* Database persistence
* Page loading

## Persistence

To verify persistence:

```bash
docker compose --env-file .env.docker down
docker compose --env-file .env.docker up -d --wait
```

Loans and reservations remain because MongoDB uses the `mongo-data` volume.

> **Important:** Do not use `down -v`, as this removes the MongoDB volume and deletes the persisted data.

## Security

`.env`, `.env.docker`, passwords, JWT secrets, and Atlas credentials are excluded from Git.

The marker does not need private credentials because Docker creates local settings automatically.

## Fresh-Environment Testing

Fresh-environment testing details are available in:

* `evidence/FRESH_ENVIRONMENT.md`
* `evidence/fresh-environment.log`
* `.github/workflows/docker-smoke.yml`

## Stop the Application

```bash
docker compose --env-file .env.docker down
```
