## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/154

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x


**Tier:** [ x ] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

The issue is that when the GET /health endpoint is hit, the logic in the `health_check` function (located in api/routes/health.py) incorrectly uses a raw SQL string of "SELECT 1" to check if the DB service is up. Since raw SQL strings are not accepted in SQLAlchemy 2.x, an error occurs, resulting in the endpoint falsely determining that the PostgreSQL service is down. This impacts the reliability of the GET /health endpoint.

A successful fix would prevent the use of raw SQL queries, and ensure that the GET /health endpoint accurately reports whether PostgreSQL is up or down, rather than reporting false negatives.

**Branch name:** fix/154-db-probe-raw-sql-error

**Setup confirmation:** [ x ] App runs locally at localhost:5173

**Cohort ledger:** [ x ] Issue added to cohort ledger

**Is This Issue Right for Me? Checklist:**
Part 1:
- I am able to understand the issue, locate the file where the issue stems from (api/routes/health.py), and can describe what a fixed app state should look  (a reliable GET /health endpoint).

Part 2:
- I chose a Tier 1 issue, since this is my first open-source contribution.

Part 3:
- I read through the `health_check` function in api/routes/health.py, and am able to see where the raw SQL string is used (line 31 - `await db.execute("SELECT 1")`).
- I have a general understanding of the surrounding code (checks status of PostgreSQL, Redis, and Vector DB via try/except blocks, updating the fields of a dictionary that eventually becomes the endpoint response).
- I have looked at a test file and saw how existing tests are structured.

Part 4:
- I have seem the issue comments and the ledger, and I am okay with the number of people working on it.
- I am confident I can implement, test, and submit a PR before the Week 9 deadline.
- There are no open blockers.