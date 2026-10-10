# API Testing with Postman and Newman

![API Tests](https://github.com/alfred-almaalouli/postman-api-tests/actions/workflows/api-tests.yml/badge.svg)

API tests for the [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html) demo API, written in **Postman** and run from the command line with **Newman**. The tests run in GitHub Actions on every push and every weekday morning, so the pipeline also works as a small API monitor.

## What is tested

| Folder | Requests | Checks |
|--------|----------|--------|
| 01 Health | `GET /ping` | API is reachable |
| 02 Authentication | `POST /auth` (valid and wrong password) | token is returned, wrong password gives no token |
| 03 Booking CRUD | list, create, read, update (PUT), partial update (PATCH), update without token, delete, read deleted | status codes, response body, JSON schema, response time, date logic, 403 without token, 404 after delete |
| 04 Negative tests | `GET /booking/999999999` | 404 for a booking that does not exist |
| 05 Data-driven | `POST /booking` for every row in [`data/guests.csv`](data/guests.csv) | each guest is saved with the right name and price |

## How the requests are chained

```
POST /auth  ──► token  ──────────────┐
POST /booking ──► bookingId ──┐      │
GET  /booking/{bookingId}  ◄──┤      │
PUT  /booking/{bookingId}  ◄──┼──────┤  (Cookie: token=...)
PATCH /booking/{bookingId} ◄──┼──────┤
DELETE /booking/{bookingId} ◄─┴──────┘
GET  /booking/{bookingId}  → 404
```

The token and the booking ID are saved as collection variables in the test scripts (`pm.collectionVariables.set`) and reused by the next requests.

## Run locally

Import both files from [`collection/`](collection) into Postman and run the collection, or use Newman:

```bash
npm install -g newman newman-reporter-htmlextra

# main flow
newman run collection/restful-booker.postman_collection.json \
  -e collection/restful-booker.postman_environment.json \
  --folder "01 Health" --folder "02 Authentication" \
  --folder "03 Booking CRUD" --folder "04 Negative tests" \
  -r cli,htmlextra

# data-driven tests
newman run collection/restful-booker.postman_collection.json \
  -e collection/restful-booker.postman_environment.json \
  --folder "05 Data-driven" -d data/guests.csv
```

The HTML reports from the pipeline can be downloaded under **Actions → latest run → Artifacts**.

## Tech stack

Postman · Newman · JavaScript (Chai assertions) · JSON Schema · GitHub Actions

## Author

Alfred Al Maalouli – [GitHub](https://github.com/alfred-almaalouli)
