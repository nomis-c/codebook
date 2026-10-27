# Neo4j 5: Old Syntax → New Syntax

Many tutorials and Stack Overflow answers use Neo4j 4.x syntax. **removed** = syntax error in 5.x, **deprecated** = still works with a warning.

| Old (4.x)                                           | New (5.x)                                                  | Status     |
|-----------------------------------------------------|------------------------------------------------------------|------------|
| `id(n)`                                             | `elementId(n)` (string) or your own ID property            | deprecated |
| `exists(n.prop)`                                    | `n.prop IS NOT NULL`                                       | removed    |
| `exists((a)-->(b))`                                 | `EXISTS { (a)-->(b) }`                                     | still works, `EXISTS { }` preferred |
| `size((a)-->(b))`                                   | `COUNT { (a)-->(b) }`                                      | removed    |
| `RETURN (a)-->(b)`                                  | `[(a)-->(b) \| b]`                                         | removed    |
| `CREATE CONSTRAINT ON (n:L) ASSERT n.x IS UNIQUE`   | `CREATE CONSTRAINT name FOR (n:L) REQUIRE n.x IS UNIQUE`   | removed    |
| `CREATE INDEX ON :L(x)`                             | `CREATE INDEX name FOR (n:L) ON (n.x)`                     | removed    |
| `DROP INDEX ON :L(x)`                               | `DROP INDEX name`                                          | removed    |
| `CALL db.indexes()` / `db.constraints()`            | `SHOW INDEXES` / `SHOW CONSTRAINTS`                        | removed    |
| `USING PERIODIC COMMIT 1000 LOAD CSV ...`           | `CALL { ... } IN TRANSACTIONS OF 1000 ROWS`                | removed    |
| `CALL { WITH x ... }`                               | `CALL (x) { ... }`                                         | deprecated (5.23+) |
| `distance(p1, p2)`                                  | `point.distance(p1, p2)`                                   | removed    |
| `neo4j-admin dump --database=neo4j --to=...`        | `neo4j-admin database dump neo4j --to-path=...`            | removed    |
| `neo4j-admin import ...`                            | `neo4j-admin database import full ...`                     | removed    |

### Example
```cypher
// OLD
MATCH (t:Tracer) RETURN t.name, size((t)-[:STUDIED_IN]->()) AS n
// NEW
MATCH (t:Tracer) RETURN t.name, COUNT { (t)-[:STUDIED_IN]->() } AS n
```

```cypher
// OLD
USING PERIODIC COMMIT 1000
LOAD CSV WITH HEADERS FROM 'file:///ppi.csv' AS row
MERGE (a:Protein {symbol: row.a})

// NEW
:auto
LOAD CSV WITH HEADERS FROM 'file:///ppi.csv' AS row
CALL (row) {
  MERGE (a:Protein {symbol: row.a})
} IN TRANSACTIONS OF 1000 ROWS
```

## Check your version
```cypher
CALL dbms.components() YIELD name, versions, edition
RETURN name, versions, edition
```

## Notes
- `elementId()` is a string like `4:7f3c...:6`, not a number — and like `id()` it gets reused after deletion and changes after dump/reimport → only for short-term lookups in one session
- Neo4j Browser shows a ⚠️ above the result for deprecated syntax; in Python it's in `summary.notifications`
