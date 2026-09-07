# Borrow, reserve and return

Two new pages reuse `public/css/catalogue.css`: `/borrowBooks.html` for borrowing/reservations and `/returnBooks.html` for the signed-in user's returns. A shared script and small supplementary stylesheet keep both pages consistent. Catalogue navigation links to both pages. No new packages are required.

## Run with the team's MongoDB Atlas database

```sh
cd /Users/ujjains/LibSwap
npm ci
# Only if .env does not exist:
cp -n .env.example .env
# Edit .env locally: set the team's MONGODB_URI, JWT_SECRET and JWT_EXPIRES_IN.
npm start
```

Open http://localhost:3000/borrowBooks.html and sign in with an existing LibSwap account. Accounts can be created using the team's existing `/api/auth/register` API. Keep `.env` out of Git. The local clone did not have `.env` during implementation; shared Atlas connectivity has therefore not been tested. Existing Atlas records are preserved; circulation fields are added when used. There is no seed or reset step for Atlas.

## Behaviour and API

Authenticated endpoints use the existing JWT middleware and derive ownership from `req.user.userId`, never a client-supplied user ID:

- `GET /api/books/circulation`: books with the current user's loan state, queue position and permitted actions.
- `POST /api/books/:id/borrow`: borrow an available copy; a nonempty queue gives its first user priority.
- `POST /api/books/:id/return`: only the current borrower can return a book.
- `POST /api/books/:id/reserve`: reserve an unavailable book once; borrowers cannot reserve their own loan.

Send `Authorization: Bearer <login token>`. POST bodies are not needed. Responses use `message` consistently with the project. Invalid IDs return 400; missing/invalid authentication 401; missing books 404; circulation conflicts 409.

The existing Book model retains title, author, genre and available. It adds borrower (User reference), borrowedAt and reservations (ordered User references). These fields are excluded from the public catalogue API. Each update is atomic within the book document, preventing double borrowing and duplicate reservations without requiring a multi-document transaction. Existing records without the new fields work without migration.

Each Book document represents one copy. On return, available becomes true and the queue is retained; only its first user can borrow. The new page labels this as held for the first reservation. The existing catalogue's availability flag continues to mean physically available. Queue cancellation/expiry is not included. Older unavailable records without a borrower can be reserved, but need their true borrower reconciled by the team before an owner can return them. Login tokens are stored in sessionStorage for navigation between the pages and cleared on sign-out or a 401 response.

## Tests

Use a separate local test MongoDB, not the shared Atlas database:

```sh
# Terminal 1 (if MongoDB is not already running locally)
mkdir -p /tmp/libswap-test-db
mongod --dbpath /tmp/libswap-test-db --port 27028 --bind_ip 127.0.0.1
# Terminal 2
cd /Users/ujjains/LibSwap
TEST_MONGODB_URI=mongodb://127.0.0.1:27028 npm test
```

The suite uses a unique `libswap_test_*` database, creates test accounts through the registration API, logs in, tests real HTTP and MongoDB, and drops only that temporary database afterward. It does not use MONGODB_URI for tests. Checks include original book records, authentication, invalid/missing IDs, simultaneous borrows, duplicate reservations, ownership spoof attempts, FIFO queue persistence, return/borrow cycles, public API privacy and all page assets.

Manual browser flow: A signs in and borrows; B signs in and reserves; reload shows B's queue position; A returns from My Returns; B can then borrow from Borrow / Reserve and return from My Returns. Search should show an empty result for unmatched text; sign-out clears the list. The automated HTTP/MongoDB tests passed during implementation. Browser checks also passed for sign-in, borrowing, a second user reserving, reservation persistence after reload, navigation to the returns page, returning and the empty-loans state. The returns layout was visually checked at a narrow viewport.

## Exact feature files

Modified: `models/Book.js`, `routes/bookRoutes.js`, `server.js`, `package.json`, `public/catalogue.html`, `public/css/catalogue.css`, `public/js/catalogue.js`.

Added: `controllers/circulationController.js`, `public/borrowBooks.html`, `public/returnBooks.html`, `public/css/circulation.css`, `public/js/circulation.js`, `tests/circulation.test.js`, `tests/BORROW_RETURN.md`.

The catalogue script now inserts book text safely as text, rather than interpreting stored book titles as HTML. Existing Docker files and the empty `Pages/borrowBooks.html` are untouched; Express serves the new pages from `public`.

## Commit and merge after review

```sh
git switch borrowBooks
git diff --check
git add models/Book.js routes/bookRoutes.js server.js package.json public/catalogue.html public/css/catalogue.css public/js/catalogue.js controllers/circulationController.js public/borrowBooks.html public/returnBooks.html public/css/circulation.css public/js/circulation.js tests/circulation.test.js tests/BORROW_RETURN.md
git diff --cached
git commit -m "feat: add authenticated borrowing returns and reservations"
git push -u origin borrowBooks
```

Open a pull request into main for team review. For an agreed local merge:

```sh
git switch main
git pull --ff-only origin main
git merge --no-ff borrowBooks
TEST_MONGODB_URI=mongodb://127.0.0.1:27028 npm test
git push origin main
```

## Later HD Docker task

Use a compatible Node image (the existing Node 18 Dockerfile is below Mongoose 9's Node requirement), the existing committed lockfile and npm ci. Express already serves UI and API. Inject the Atlas URI/JWT secret at runtime; never bake .env into the image. Configure Atlas network access for the deployment, then demonstrate a clean container build plus localhost login/borrow/reserve/return and persistence after restart. A MongoDB container is optional when using Atlas. Add the required /api/student identity and other identity evidence once the complete assignment rubric is available; the supplied excerpt was truncated. Submit your individual repository, evidence and personal reflection. Docker configuration remains separate from this feature.
