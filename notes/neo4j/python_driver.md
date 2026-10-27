# neo4j Python Driver: Query and Load Data

## Install
```bash
uv add neo4j pandas
```

## Connect
```python
import neo4j
from neo4j import GraphDatabase

URI, AUTH = "bolt://localhost:7687", ("neo4j", "password123")

driver = GraphDatabase.driver(URI, auth=AUTH)
driver.verify_connectivity()
# ...
driver.close()
```
- Or as a context manager: `with GraphDatabase.driver(URI, auth=AUTH) as driver:`

## Run a query
```python
records, summary, keys = driver.execute_query(
    "MATCH (t:Tracer {nuclide: $nuc}) RETURN t.name AS name",
    nuc="F-18",
    database_="neo4j",
)
for r in records:
    print(r["name"])
```
- Query parameters are plain keyword args (`nuc=...`), driver options end with `_` (`database_=...`)

## Query straight into a DataFrame
```python
df = driver.execute_query(
    "MATCH (t:Tracer)-[s:STUDIED_IN]->(d:Disease) "
    "RETURN t.name AS tracer, d.name AS disease, s.sensitivity AS sens",
    database_="neo4j",
    result_transformer_=neo4j.Result.to_df,
)
```

## Write in batches (UNWIND)
```python
LOAD_BINDS = """
UNWIND $rows AS row
MERGE (t:Tracer  {name: row.tracer})
MERGE (p:Protein {symbol: row.target})
MERGE (t)-[b:BINDS]->(p)
SET b.affinity_nM = row.affinity_nM, b.evidence_tier = row.tier
"""

def chunks(rows, n=5000):
    for i in range(0, len(rows), n):
        yield rows[i:i + n]

def write_batch(tx, rows):
    tx.run(LOAD_BINDS, rows=rows)

with driver.session(database="neo4j") as session:
    for batch in chunks(rows):            # rows = list of dicts, e.g. df.to_dict("records")
        session.execute_write(write_batch, batch)
```
- One `UNWIND` with 5000 rows is orders of magnitude faster than 5000 single queries
- `execute_write` / `execute_read` retry automatically on transient errors

## Export to NetworkX
```python
import networkx as nx

edges = driver.execute_query(
    "MATCH (a:Protein)-[:INTERACTS_WITH]->(b:Protein) RETURN a.symbol AS a, b.symbol AS b",
    database_="neo4j",
    result_transformer_=neo4j.Result.to_df,
)
G = nx.from_pandas_edgelist(edges, "a", "b")      # undirected graph
```
- Matched as directed (`->`) on purpose so each undirected edge comes back only **once**

## Notes
- Always pass values as parameters (`$nuc`), never via f-strings → safe against injection + Neo4j can reuse the query plan
- Deprecation warnings are logged by the driver and end up in `summary.gql_status_objects` (`summary.notifications` is deprecated in driver 6) — fix them, they break on the next major version
- `CALL { ... } IN TRANSACTIONS` doesn't work with `execute_query` → use `session.run()`, see [import_csv_with_load_csv.md](import_csv_with_load_csv.md)
- Integration tests against a real DB: `uv add --dev testcontainers`, then `from testcontainers.community.neo4j import Neo4jContainer` → `Neo4jContainer("neo4j:5")`
