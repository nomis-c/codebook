# Cypher: WITH, OPTIONAL MATCH and Subqueries

## Keep rows without a match (OPTIONAL MATCH)
```cypher
MATCH (t:Tracer)
OPTIONAL MATCH (t)-[:BINDS]->(p:Protein)
RETURN t.name, p.symbol AS target
```
- Like a SQL `LEFT JOIN` — tracers without a `BINDS` edge stay in the result with `target = null`
- With a plain `MATCH` they would disappear entirely

## Chain query steps (WITH)
```cypher
MATCH ...
WITH <vars>, <aggregates>
WHERE ...
MATCH ...
RETURN ...
```
- `WITH` ends one part of the query and passes on **only** the variables it names
- Can be used any number of times

### Filter after aggregation (like SQL HAVING)
```cypher
MATCH (t:Tracer)-[:STUDIED_IN]->(d:Disease)
WITH t, count(d) AS n
WHERE n > 1
RETURN t.name, n
```

### Find nodes with zero edges of a type
```cypher
MATCH (t:Tracer)
OPTIONAL MATCH (t)-[:BINDS]->(p:Protein)
WITH t, count(p) AS n_targets
WHERE n_targets = 0
RETURN t.name
```

### Top-N per group
```cypher
MATCH (d:Disease)<-[s:STUDIED_IN]-(t:Tracer)
WITH d, t, s
ORDER BY s.sensitivity DESC
WITH d, collect(t.name)[0..2] AS top2
RETURN d.name, top2
```

### Reuse an intermediate result
```cypher
MATCH (:Tracer {name: '[68Ga]Ga-DOTATATE'})-[:BINDS]->(p:Protein)
WITH p
MATCH (p)-[:ASSOCIATED_WITH]->(d:Disease)
RETURN p.symbol, collect(d.name) AS diseases
```

## Check if a pattern exists (EXISTS)
```cypher
MATCH (t:Tracer)
WHERE EXISTS { (t)-[:BINDS]->(:Protein {symbol: 'SSTR2'}) }
RETURN t.name
```
- Shorter version of the zero-edges example above: `WHERE NOT EXISTS { (t)-[:BINDS]->() }`

## Count a pattern (COUNT)
```cypher
MATCH (t:Tracer)
WHERE COUNT { (t)-[:STUDIED_IN]->() } >= 2
RETURN t.name, COUNT { (t)-[:STUDIED_IN]->() } AS n_diseases
```

## Collect from a subquery (COLLECT)
```cypher
MATCH (t:Tracer)
RETURN t.name, COLLECT { MATCH (t)-[:BINDS]->(p) RETURN p.symbol } AS targets
```

## Run a subquery per row (CALL)
```cypher
MATCH (d:Disease)
CALL (d) {
  MATCH (d)<-[s:STUDIED_IN]-(t:Tracer)
  RETURN t.name AS best
  ORDER BY s.sensitivity DESC
  LIMIT 1
}
RETURN d.name, best
```
- `CALL (d) { ... }` imports `d` into the subquery (5.23+)
- Older syntax: `CALL { WITH d ... }` (deprecated)
- Rows where the subquery returns nothing are dropped → `OPTIONAL CALL (d) { ... }` to keep them (5.24+)

## Build nested JSON (map projection)
```cypher
MATCH (t:Tracer)
RETURN t {
  .name, .nuclide,
  targets:  [(t)-[:BINDS]->(p:Protein) | p.symbol],
  diseases: [(t)-[s:STUDIED_IN]->(d:Disease) WHERE s.phase >= 3 | d.name]
} AS tracer
```
- `[pattern WHERE ... | expression]` = pattern comprehension, returns a list per row

## Combine results (UNION)
```cypher
MATCH (t:Tracer)-[:BINDS]->(p:Protein)   RETURN t.name AS tracer, p.symbol AS target
UNION
MATCH (t:Tracer)-[:REPORTS]->(p:Protein) RETURN t.name AS tracer, p.symbol AS target
```
- Column names must be identical in every part
- `UNION` removes duplicates, `UNION ALL` keeps them

# Notes
- Variables not listed in `WITH` are gone afterwards — `WITH *` passes everything on
- Aggregation in `WITH` groups by all non-aggregated variables listed there, same as in `RETURN`
- `size((t)-->())` was removed in Neo4j 5 → `COUNT { }`; `exists((a)-->(b))` still works, but `EXISTS { }` is the 5.x form
