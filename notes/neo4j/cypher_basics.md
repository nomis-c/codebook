# Cypher Basics

Task-based quick reference for Neo4j 5. Examples use a small tracer graph:

```
(:Tracer)-[:BINDS]->(:Protein)-[:ASSOCIATED_WITH]->(:Disease)
(:Tracer)-[:STUDIED_IN]->(:Disease)
(:Protein)-[:INTERACTS_WITH]-(:Protein)
```

## Patterns

| Pattern                                | Meaning                          |
|----------------------------------------|----------------------------------|
| `(n)`                                  | any node                         |
| `(t:Tracer)`                           | node with label                  |
| `(t:Tracer:Therapeutic)`               | node with both labels            |
| `(n:Tracer\|Protein)`                  | node with one of the labels      |
| `(t:Tracer {name: '[18F]FDG'})`        | inline filter (equality only)    |
| `(a)-[:BINDS]->(b)`                    | directed edge                    |
| `(a)<-[:BINDS]-(b)`                    | other direction                  |
| `(a)-[:INTERACTS_WITH]-(b)`            | undirected (either direction)    |
| `(a)-[r:BINDS]->(b)`                   | edge bound to a variable         |
| `(a)-[:BINDS\|REPORTS]->(b)`           | one of several types             |
| `(a)-->(b)`                            | any outgoing edge                |

## Find Nodes

```cypher
MATCH (t:Tracer) RETURN t                                // whole nodes → graph view
MATCH (t:Tracer) RETURN t.name AS tracer, t.nuclide      // values → table, alias with AS
MATCH (t:Tracer {name: '[18F]FDG'}) RETURN t             // inline filter
MATCH (t:Tracer) RETURN DISTINCT t.nuclide               // unique values
```

## Filter

```cypher
MATCH (t:Tracer)
WHERE t.half_life_min >= 100 AND NOT t.approved
RETURN t.name
```

| Operator                                 | Example                                   |
|------------------------------------------|-------------------------------------------|
| `=` `<>` `<` `<=` `>` `>=`               | `t.half_life_min >= 100`                  |
| `AND` `OR` `NOT` `XOR`                   | `NOT t.approved`                          |
| `IN`                                     | `t.nuclide IN ['F-18', 'Ga-68']`          |
| `STARTS WITH` `ENDS WITH` `CONTAINS`     | `t.name CONTAINS 'DOTATATE'`              |
| `=~` (regex)                             | `t.name =~ '(?i).*dotatate.*'`            |
| `IS NULL` `IS NOT NULL`                  | `p.uniprot IS NULL`                       |
| pattern exists / doesn't                 | `WHERE NOT EXISTS { (t)-[:BINDS]->() }`   |

## Sort and Paginate

```cypher
MATCH (t:Tracer)
RETURN t.name, t.half_life_min
ORDER BY t.half_life_min DESC, t.name ASC      // second key = tie-breaker
SKIP 10 LIMIT 10                               // page 2
```

## Follow Relationships

```cypher
MATCH (t:Tracer)-[b:BINDS]->(p:Protein)                  // edge properties via variable
WHERE b.affinity_nM <= 1
RETURN t.name, p.symbol, b.affinity_nM

MATCH (t:Tracer)-[:BINDS]->(:Protein)-[:ASSOCIATED_WITH]->(d:Disease)   // multi-hop
RETURN t.name, d.name

MATCH (:Protein {symbol: 'HK2'})-[:INTERACTS_WITH]-(b:Protein)          // undirected: finds SLC2A1
RETURN b.symbol                                                          // although stored SLC2A1 -> HK2
```

## Aggregate

```cypher
MATCH (t:Tracer)-[s:STUDIED_IN]->(d:Disease)
RETURN d.name AS disease,                      // non-aggregated = grouping key
       count(t) AS n_tracers,
       round(avg(s.sensitivity), 3) AS mean_sens,
       collect(t.name) AS tracers
ORDER BY n_tracers DESC
```

| Function                    | Result                              |
|-----------------------------|-------------------------------------|
| `count(x)` / `count(*)`     | non-null values / rows              |
| `count(DISTINCT x)`         | distinct values                     |
| `sum(x)` `avg(x)`           | sum / mean                          |
| `min(x)` `max(x)`           | minimum / maximum                   |
| `collect(x)`                | list (`collect(DISTINCT x)` dedups) |
| `stDev(x)`                  | sample standard deviation           |
| `percentileCont(x, 0.5)`    | median                              |

## Create

```cypher
CREATE (t:Tracer:Therapeutic {name: '[177Lu]Lu-DOTATATE', nuclide: 'Lu-177'})   // node, 2 labels
CREATE (:Tracer {name: 'X'})-[:BINDS {affinity_nM: 1.0}]->(:Protein {symbol: 'Y'})  // 2 nodes + edge
```

Edge between nodes that already exist:

```cypher
MATCH (t:Tracer {name: '[177Lu]Lu-DOTATATE'}), (p:Protein {symbol: 'SSTR2'})
CREATE (t)-[:BINDS {affinity_nM: 1.5}]->(p)
```

## Create Only If Missing (MERGE)

```cypher
MERGE (p:Protein {symbol: 'SSTR2'})
ON CREATE SET p.created = datetime()
ON MATCH  SET p.lastSeen = datetime()
```

Import pattern: merge each node on its ID, **then** the edge:

```cypher
MERGE (t:Tracer  {name: $tracer})
MERGE (p:Protein {symbol: $target})
MERGE (t)-[b:BINDS]->(p)
SET b.affinity_nM = $affinity
```

Symmetric edge, stored once:

```cypher
MATCH (a:Protein {symbol: 'SLC2A1'}), (b:Protein {symbol: 'HK2'})
MERGE (a)-[:INTERACTS_WITH]-(b)       // MERGE may be undirected, CREATE may not
```

## Update

```cypher
MATCH (p:Protein {symbol: 'SSTR2'})
SET p.uniprot = 'P30874'                  // set/overwrite one property
SET p.a = 1, p.b = 2                      // several
SET p:Target                              // add label
SET p += {name: 'Somatostatin receptor 2'}  // merge in a map
SET p = {symbol: 'SSTR2'}                 // replace ALL properties
SET p.count = coalesce(p.count, 0) + 1    // counter
REMOVE p.old_prop, p:Target               // remove property + label
```

## Delete

```cypher
MATCH (t:Tracer {name: '[11C]PiB'}) DETACH DELETE t        // node + all its edges
MATCH (:Tracer {name: 'X'})-[r:BINDS]->() DELETE r         // edge only
MATCH (n) DETACH DELETE n                                  // EVERYTHING
```

## Useful Functions

| Function                          | Result                                  |
|-----------------------------------|-----------------------------------------|
| `labels(n)`                       | list of labels                          |
| `type(r)`                         | relationship type                       |
| `keys(n)` / `properties(n)`       | property names / all properties as map  |
| `elementId(n)`                    | internal ID (string, don't store it)    |
| `coalesce(a, b)`                  | first non-null value                    |
| `toLower(s)` `toUpper(s)` `trim(s)` | string cleanup                        |
| `split(s, ';')`                   | string → list                           |
| `toIntegerOrNull(s)` `toFloatOrNull(s)` | convert, `null` instead of error  |
| `size(list)` / `list[0..2]`       | list length / slice (end exclusive)     |
| `round(x, 3)`                     | round                                   |
| `datetime()` `date()`             | now                                     |

Conditional value:

```cypher
MATCH (t:Tracer)
RETURN t.name,
  CASE
    WHEN t.half_life_min < 30  THEN 'short'
    WHEN t.half_life_min < 120 THEN 'medium'
    ELSE 'long'
  END AS half_life_class
```

List → rows:

```cypher
UNWIND ['SSTR2', 'FOLH1', 'ESR1'] AS sym
MATCH (p:Protein {symbol: sym})
RETURN p.name
```

## Inspect the Database

```cypher
CALL db.schema.visualization()               // meta graph: labels + relationship types
MATCH (n) RETURN labels(n), count(*)         // nodes per label
MATCH ()-[r]->() RETURN type(r), count(*)    // edges per type
CALL db.labels()
CALL db.relationshipTypes()
CALL db.propertyKeys()
CALL dbms.components()                       // Neo4j version + edition
```

## Debug a Query

```cypher
EXPLAIN MATCH ...      // query plan without running it
PROFILE MATCH ...      // query plan + actual DB hits
```

| In the plan                        | Means                                     |
|------------------------------------|-------------------------------------------|
| `AllNodesScan` / `NodeByLabelScan` | missing label or index                    |
| `CartesianProduct`                 | two unconnected patterns                  |
| `Eager`                            | large write, split into batches           |

## Neo4j Browser

| Command / shortcut      | Effect                                  |
|-------------------------|-----------------------------------------|
| `Ctrl + Enter`          | run                                     |
| `Shift + Enter`         | new line                                |
| `Ctrl + ↑`              | previous query                          |
| `:param nuc => 'F-18'`  | set a parameter, use it as `$nuc`       |
| `:params`               | show parameters                         |
| `:schema`               | constraints and indexes                 |
| `:sysinfo`              | DB size, counts                         |
| `:use <db>`             | switch database                         |
| `:clear`                | clear result frames                     |

# Notes
- Keywords are case-insensitive — labels, relationship types, property names and string values are **case-sensitive**
- Typo, wrong case or wrong arrow direction → empty result, **no error** (only a warning for unknown labels/properties)
- `DELETE` on a node that still has edges fails → use `DETACH DELETE`
- `CREATE` always creates new → duplicates; use `MERGE` + a unique constraint for anything with an ID
- `MERGE` on a whole pattern creates the **entire** pattern if any part is missing → merge nodes first, then the edge
- `WHERE x = null` never matches → `IS NULL`
- Two unconnected patterns separated by a comma → Cartesian product, result explodes
- Pagination without `ORDER BY` → unstable order
- Always give a label: `MATCH (n {name: ...})` scans **all** nodes
- Naming convention: labels `PascalCase` singular, relationship types `UPPER_SNAKE_CASE`, properties `camelCase` or `snake_case` (be consistent)
