# Text-to-SQL over a Real Database

Junior Project · University of Sharjah

**Team:** Murtadha Hasan (U23106466) · Mohammed Ahmad (U23106325)

Ask a question in plain English and get the answer from a real SQLite database. A large language model (Google Gemini) turns the question into SQL, then the system checks the query and runs it.

## How it works

1. **Schema context** – the table definitions, foreign keys, 3 sample rows per table and the business rules are put into the prompt.
2. **Generate SQL** – Gemini returns a JSON object with the SQL and a one-sentence explanation.
3. **Safety check** – only a single `SELECT` (or `WITH … SELECT`) is allowed; write and admin keywords are blocked.
4. **Execute** – the query runs on an in-browser SQLite database (sql.js), locked with `PRAGMA query_only`, capped at 500 rows.
5. **Self-correct** – if SQLite returns an error, the error is sent back to the model, up to 2 times.

## Database

A sample online store (`harbor_goods.db`): `customers`, `products`, `orders`, `order_items`, `reviews` – about 1,900 rows, generated with a fixed seed so results are the same every time.

## Running it

Open the GitHub Pages link (or `index.html` in a browser), open **Gemini API key**, paste a key from [Google AI Studio](https://aistudio.google.com/apikey) and press **Save on this device**. The key stays in that browser only and is never part of this repository.

The example questions work without a key, using SQL saved with each example.

## Tech

HTML, CSS, JavaScript · sql.js (SQLite compiled to JavaScript) · Google Gemini API
