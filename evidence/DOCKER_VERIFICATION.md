# Docker verification — 7 September 2026

Student ID: 226411987. Tests performed on macOS/Apple Silicon using Docker Engine 29.5.3, Compose v5.1.4, Node v22.23.2 inside the app container, and mongo:8.0.

## Observed results

| Check | Result |
| --- | --- |
| Compose configuration validation | Passed |
| App image built with locked dependencies | Passed; npm audit at build reported 0 vulnerabilities in application packages |
| App and MongoDB health checks | Both healthy |
| Application runs as non-root | UID 1000 |
| `.env` and `.env.docker` absent from image | Confirmed |
| Public endpoint bound to host loopback | 127.0.0.1:3000 |
| MongoDB host port unpublished | Confirmed |
| Existing integration suite inside container | 1 suite test passed, 0 failures; multiple assertions cover registration, login, circulation, concurrency, validation and assets |
| `/api/student` | studentId = 226411987 |
| All three HTML pages contain identity | Passed |
| Docker HTTP test before recreation | Login, borrow, reserve and ownership check passed |
| Both containers removed and recreated without deleting volume | Passed |
| Loan and reservation persisted | Passed |
| FIFO borrow after return and final return | Passed |
| Catalogue loaded through browser from Docker | Confirmed: 3 demo books and student ID visible |

Representative actual outputs:

```text
PASS: identity, all pages, login, borrow, reserve and return ownership.
PASS: loan and reservation survive restart, FIFO priority, reserved borrowing and final return.
{"node":"v22.23.2","uid":1000,"envFileBakedIn":false,"dockerEnvBakedIn":false}
```

The verification used an isolated `libswap-hd-check` Compose project and named database volume. No shared Atlas records were accessed or changed. The npm audit statement concerns application packages at build time, not a container operating-system vulnerability scan. No cross-architecture run or public hosting deployment was performed. This is implementation evidence, not a replacement for the student's required screenshots, personal reflection or the full rubric.

## Final local-clone check

The root repository contains an unused empty file named `node`. It caused the base image entrypoint to invoke that empty file instead of server.js. Adding `/node` to .dockerignore resolved this. The final `libswap` Compose project was rebuilt from the actual clone, both services became healthy, demo seeding succeeded, and the in-container integration suite passed again. This stack is left running at localhost:3000.
