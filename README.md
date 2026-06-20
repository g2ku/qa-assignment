# QA Middle Assignment

QA test assignment covering API testing (Postman/Newman), manual UI test cases, SQL queries, and process diagrams.

## What's done

- **Postman collection** — CRUD coverage for `/booking` endpoint of [Restful Booker API](https://restful-booker.herokuapp.com), with positive and negative scenarios, boundary value tests for dates, and assertions on status code, response structure, and response time.
- **UI test cases** — 20 manual test cases covering the Login page and File Upload page on [the-internet.herokuapp.com](https://the-internet.herokuapp.com), using Equivalence Partitioning and Boundary Value Analysis.
- **Bug report** — 1 real bug found and documented during manual testing (Upload page returns Internal Server Error instead of a user-friendly message).
- **SQL queries** — 5 queries answering common QA data-validation questions (users without bookings, overlapping bookings, top rooms, stale confirmed bookings, booking counts by room type).
- **Diagrams** — booking process flow (happy path + 2 exceptions), sequence diagram for booking creation, and state transition map for the Booking entity.

## Project structure

```
/
├── README.md
├── postman/
│   ├── collection.json
│   └── environment.json
├── test-cases/
│   └── ui-test-cases.md
├── sql/
│   └── queries.sql
├── diagrams/
│   └── diagrams.md
```

## How to run the Postman collection (Newman)

```bash
npm install -g newman
newman run postman/collection.json -e postman/environment.json
```

Or import both files into the Postman app: **Import → postman/collection.json** and **Import → postman/environment.json**, then select the environment and run the collection.

## Found bugs / observations

- `POST /booking` accepts a booking where `checkout` date is earlier than `checkin` date — no server-side validation on date logic.
- The Upload page on the-internet.herokuapp.com returns a raw `Internal Server Error` when the Upload button is pressed without selecting a file, instead of a user-facing validation message.
- `POST /auth` with invalid credentials still returns HTTP 200 (with `{"reason": "Bad credentials"}`) instead of a 401/403 — inconsistent with REST conventions.

## What I'd improve with more time

- Add Playwright/Cypress E2E automation with Page Object Model and Gherkin scenarios (Task 3).
- Add a CI pipeline (GitHub Actions) to run the Postman collection and E2E tests automatically on every push.
- Add an Allure or Playwright HTML report for test results.
- Expand SQL section with a written MongoDB/NoSQL answer (document structure, indexing strategy, data validation approach).
- Add more boundary-value coverage for the `totalprice` field (negative numbers, zero, very large numbers).
