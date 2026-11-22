# Text-to-SQL over a Real Database

Junior Project · University of Sharjah

**Team:** Murtadha Hasan (U23106466) · Mohammad Ahmad (U23106325)

Ask a question in plain English and get the answer from a real SQLite database. A large language model (Google Gemini) turns the question into SQL, then the system checks the query and runs it.

This is a **feasibility prototype**. It is not the multi-component architecture proposed for the Senior Project (Orchestrator, Schema-Linking Agent, SQL Generator Agent, Read-Only Sandbox and Validator on Python and PostgreSQL).

## How it works

1. **Schema context** – the table definitions, foreign keys, 3 sample rows per table and the business rules are put into the prompt. In **Full** mode (the default) every table is sent. In **Linked** mode only the tables chosen by the schema linker are sent (see below).
2. **Generate SQL** – Gemini returns a JSON object with the SQL and a one-sentence explanation.
3. **Safety check** – only a single `SELECT` (or `WITH … SELECT`) is allowed; write and admin keywords are blocked (the `REPLACE()` string function is allowed; `REPLACE INTO` is not). A query that breaks this policy **stops the request** and is never sent back for repair.
4. **Execute** – the query runs on an in-browser SQLite database (sql.js), locked with `PRAGMA query_only`, capped at 500 rows. A query that runs but returns no rows is flagged (not repaired), because execution success is not a correct answer.
5. **Self-correct** – if SQLite returns an error, the failed SQL and the error are sent back to the model, up to 2 times. After that the request ends with an explicit failure.

## Schema linking (optional, added 22 Nov 2026 in build 3)

A rule-based schema linker, a first version of the keyword baseline in UC-02 of the report, picks the tables a question needs. It uses:

- table and column names found in the question (with simple plural handling);
- exact values from short text columns (for example `Berlin`, `Kitchen`, `Wall Clock`);
- a short list of business terms for this store (for example *revenue* or *ordered* → `orders`, `order_items`);
- the shortest **declared** foreign-key path between the linked tables. Undeclared joins are never invented; problems are reported as gaps. If nothing matches, the full schema is used.

The linker runs in both modes and its choice is highlighted in the schema panel and explained in the log. Only **Linked** mode actually narrows the prompt. In Linked mode, if a query fails with `no such table` or `no such column`, the linked set is **expanded by one foreign-key hop** before the repair attempt (an idea taken from AutoLink's schema exploration and expansion).

The **Linking** column in the evaluation panel checks, without calling the LLM, whether the linker picked every table used by the reference SQL (strict recall). Because the business-term list was written with this test set in view, recall on these questions is optimistic. Example of a miss on an unseen question: *"What is the best-rated item?"* does not link `products`.

## Database

A sample online store (`harbor_goods.db`): `customers`, `products`, `orders`, `order_items`, `reviews` – about 1,900 rows, generated with a fixed seed so results are the same every time. This is a small, clean schema; it corresponds to the *clean* condition of the planned evaluation, not to a messy real-world schema.

## Running it

Open the GitHub Pages link (or `index.html` in a browser), open **Gemini API key**, paste a key from [Google AI Studio](https://aistudio.google.com/apikey) and press **Save on this device**. The key stays in that browser only and is never part of this repository.

The built-in example questions use hand-written SQL (not the LLM) and work without a key. They are for demonstration only and are not part of the evaluation.

## Presentation demo

Three buttons run three live cases from the test set through the LLM: a correct answer (J3), a controlled repair (R1, a faulty query is injected and the model must fix it) and an unsupported request (U2, a delete request that must not run). When a request needs more than one attempt, every attempt is shown with its SQL and error. The demo uses the schema mode that is currently selected.

## Evaluation run

The **Evaluation run** panel at the bottom of the page runs a fixed test set through the LLM:

- **15 test questions:** 4 simple, 4 joins, 4 aggregation and 3 unsupported or ambiguous requests.
- **3 controlled repair tests:** a deliberately faulty query (wrong column, syntax error, unknown table) replaces the first LLM attempt, to test the repair loop. These are reported separately from the LLM test results.

Reference SQL and expected results were fixed before any LLM run. For each question the page records the generated SQL of every attempt, the execution outcome, whether the result matches the expected result, the number of repairs and the total response time (time lost to temporary Gemini problems is recorded separately and not counted). Execution success and result correctness are reported separately.

A result is **correct** when it has the same rows as the expected result: row order is ignored, numbers are rounded to 2 decimals, and extra columns are allowed. For the unsupported or ambiguous requests, the correct behaviour is to decline (no SQL) or stop on the safety policy. Each automatic verdict can be overridden by hand, and both values are exported.

If Gemini is temporarily unavailable (free-tier rate limit, "high demand" overload or a network error), the page waits 20 s and tries again, up to 3 times. If it still fails, the question is marked **Gemini error · not scored**: it is not counted as wrong, and **Re-run not scored** runs only those questions again.

Results are kept **separately for each schema mode**. The main evaluation is the **Full** mode run; the Linked mode run is an additional comparison.

1. Select **Full**, press **Run all 18**, then **Export CSV** or **Export JSON** (files are named `…-full.csv` / `…-full.json`).
2. Optionally select **Linked**, run again and export (`…-linked.csv` / `…-linked.json`).

The exports include the schema mode, the tables in the reference SQL, the linked tables, linking recall, the tables sent on the last attempt and whether the schema was expanded. Results stay in the browser until you press **Clear results (this mode)**. Results from build 2 are carried over into Full mode.

## Limitations

- **Small, clean schema:** five synthetic tables with readable names and declared keys, so the messy-schema question (RQ1) is not tested here.
- **Full-schema prompting by default:** the linked mode is a small, rule-based first step, not a general schema-linking method.
- **Missing relationship handling:** joins rely on declared foreign keys; undeclared relationships are neither detected nor suggested.
- **Wrong but executable queries:** the repair loop reacts only to SQL errors; a query that runs but answers the wrong question is caught only by comparison with the reference results.
- **Prototype safety check:** a keyword filter plus a read-only database, with no statement timeout.

## Tech

HTML, CSS, JavaScript · sql.js 1.10.3 (SQLite compiled to JavaScript) · Google Gemini API

Build: evaluation build 3.1 (optional schema linking; retries on Gemini overload), 23 Nov 2026
