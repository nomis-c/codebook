# LOAD CSV: Import CSV/TSV Files into Neo4j

## Where the file has to be
- `LOAD CSV` reads from the DBMS `import` folder only, referenced as `file:///<name>`
- Docker: mount it with `-v "$PWD/import:/import"`
- Neo4j Desktop: DBMS `...` menu → Open folder → Import

## Preview the file first
```cypher
LOAD CSV WITH HEADERS FROM 'file:///binds.tsv' AS row FIELDTERMINATOR '\t'
RETURN row LIMIT 5
```
- `row` is a map keyed by the header names → check spelling before writing anything

## Basic import
```cypher
LOAD CSV WITH HEADERS FROM 'file:///binds.tsv' AS row FIELDTERMINATOR '\t'
WITH row WHERE row.target IS NOT NULL AND row.target <> ''
MERGE (t:Tracer  {name: row.tracer})
MERGE (p:Protein {symbol: row.target})
MERGE (t)-[b:BINDS]->(p)
SET b.affinity_nM   = toFloatOrNull(row.affinity_nM),
    b.evidence_tier = toIntegerOrNull(row.evidence_tier),
    b.pmids         = split(row.pmids, ';')
```
- Drop `FIELDTERMINATOR '\t'` for comma-separated files
- Without `WITH HEADERS`, `row` is a list → `row[0]`, `row[1]`, ...

## Large files in batches
```cypher
:auto
LOAD CSV WITH HEADERS FROM 'file:///ppi.tsv' AS row FIELDTERMINATOR '\t'
CALL (row) {
  MATCH (a:Protein {symbol: row.a}), (b:Protein {symbol: row.b})
  MERGE (a)-[:INTERACTS_WITH]-(b)
} IN TRANSACTIONS OF 10000 ROWS
```
- Commits every 10000 rows instead of one huge transaction → no out-of-memory
- `:auto` prefix is only for Neo4j Browser — cypher-shell runs the query as is
- Python: needs an implicit transaction → `session.run(query)`, **not** `driver.execute_query()` / `execute_write()` (error: "can only be executed in an implicit transaction")

## Verification
```cypher
MATCH (n) RETURN labels(n), count(*)
MATCH ()-[r]->() RETURN type(r), count(*)
```

## Notes
- Every CSV value arrives as a **string** → always convert with `toFloatOrNull`, `toIntegerOrNull`, `toBoolean`
- Empty fields come in as `''` or `null` → filter with `WITH row WHERE ...` before `MERGE` (merging on `null` fails)
- Create unique constraints on the merge keys **before** importing (see [constraints_and_indexes.md](constraints_and_indexes.md))

Which import method when:

| Method                                       | When                                                     |
|----------------------------------------------|----------------------------------------------------------|
| `LOAD CSV`                                   | quick experiments, medium data                           |
| Python + `UNWIND $rows` in batches           | reproducible pipeline, validate in Python first → [python_driver.md](python_driver.md) |
| `neo4j-admin database import full`           | very large initial import into an **empty** DB, offline  |
