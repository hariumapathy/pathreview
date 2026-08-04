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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/hariumapathy/pathreview/commit/42450e1bef452bb7ec0bbe9164a36aa168fcd022

**Reproduction summary:**

I reproduced the issue by running the application and then making a API request to the GET /health endpoint, observing both the API response and the terminal output.

See below for the full, detailed reproduction steps.

**Detailed Reproduction Steps:**
1. Get environment setup
	1. `docker compose -d`
	2. `make setup`
	3. `make run`
2. Looked at API docs: http://localhost:8000/docs
3. Used Postman to make a GET request to http://localhost:8000/health
	1. Got the following response back:
```
{
    "detail": {
        "status": "unhealthy",
        "dependencies": {
            "postgres": "unhealthy",
            "redis": "unhealthy",
            "vector_db": "healthy"
        },
        "safety_events_last_hour": 0,
        "timestamp": "2026-07-24T01:14:11.860279"
    }
}
```

- Note that `"postgres": "unhealthy"` appears in the response, indicating that the service is down.
4. In my VS Code terminal (Git Bash), I saw the following appear:
```
2026-07-23 21:14:11 [error    ] postgres_health_check_failed   error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')" request_id=09306c44-3308-41e8-9039-d977de26b95b
2026-07-23 21:14:12 [error    ] redis_health_check_failed      error="'Settings' object has no attribute 'redis_host'" request_id=09306c44-3308-41e8-9039-d977de26b95b
2026-07-23 21:14:12 [debug    ] vector_db_health_check_passed  request_id=09306c44-3308-41e8-9039-d977de26b95b
INFO:     127.0.0.1:62321 - "GET /health HTTP/1.1" 503 Service Unavailable
```
- Note that the error `Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')` is the encountered failure message that is reported in the original GitHub issue. This indicates that the bug/issue has been reproduced.

To further verify that the Postgres DB is up, we can see the healthy container by running `docker compose ps`. We can also run `docker exec -it pathreview-db-1 pg_isready`, which results in the output: `/var/run/postgresql:5432 - accepting connections`.

Therefore, it is confirmed that the response from the GET /health endpoint is incorrectly reporting the uptime status of the Postgres DB service.


**PLAN.md link:** https://github.com/hariumapathy/pathreview/blob/fix/154-db-probe-raw-sql-error/PLAN.md


**Blockers or open questions:**
Apart from the questions raised in PLAN.md, I have no further blockers.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
[What have you implemented so far? Which sub-tasks from PLAN.md are done?]

So I far I have implemented the fixes in `api/routes/health.py`, importing the `text()` wrapper method and nested it within the `db.execute()` method call. To commit my changes, I also had to add type hints to the `health_check` method, and add a comment `# noqa: B008` to the end of the method signature. This is done to bypass the ruff issue of calling a method within a default. However, the use of the `Depends()` method is a FastAPI convention, and not an improper coding practice.

Therefore, sub-tasks 1 and 2 are done from PLAN.md.

**Next steps:**
The next steps will be to write unit tests and ensure that no further issues or warnings are introduced into the codebase because of my changes.

**Blockers:**
Understanding how to use Mock and when to use it for unit tests is a new concept for me. With the use of online resources, Claude, and reading through existing unit tests, I am getting a better idea of how to write tests myself.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/808

**Branch:** fix/154-db-probe-raw-sql-error

**What you built:**
My fix addresses issue 154, adding the `text()` wrapper method in `api/routes/health.py` wherever a raw SQL string was passed to the `db.execute()` method. This fix prevents false negatives, which is when the health endpoint reports the Postgres service as down even though it actually is up.

**Tests added or updated:**
I created a new unit test file named `test_health.py`, with 7 tests that are relevant to the API health endpoint. They check various paths, such as when the Postgres service is down or up and whether the endpoint correctly reports that.

**Self-review confirmation:** [ x ] make check passes  [ x ] make test-unit passes
- Note: There were existing issues in make check and make test-unit that my changes did not worsen

**Draft PR feedback received from:** "none"
