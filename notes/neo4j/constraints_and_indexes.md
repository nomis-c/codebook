# Neo4j: Constraints and Indexes

## Unique constraint
```cypher
CREATE CONSTRAINT protein_symbol IF NOT EXISTS
FOR (p:Protein) REQUIRE p.symbol IS UNIQUE
```
- Prevents duplicate nodes — a second `CREATE` with the same symbol fails
- Automatically creates an index on `p.symbol` → makes `MERGE` / `MATCH` on it fast

### Composite (identity made of several properties)
```cypher
CREATE CONSTRAINT tracer_id IF NOT EXISTS
FOR (t:Tracer) REQUIRE (t.ligand, t.nuclide) IS UNIQUE
```

### On relationships (5.7+)
```cypher
CREATE CONSTRAINT binds_id IF NOT EXISTS
FOR ()-[b:BINDS]-() REQUIRE b.id IS UNIQUE
```

## Indexes
```cypher
CREATE INDEX tracer_nuclide IF NOT EXISTS FOR (t:Tracer) ON (t.nuclide)                    // range (default)
CREATE INDEX tracer_nuc_appr IF NOT EXISTS FOR (t:Tracer) ON (t.nuclide, t.approved)       // composite
CREATE INDEX binds_tier IF NOT EXISTS FOR ()-[b:BINDS]-() ON (b.evidence_tier)             // relationship
CREATE TEXT INDEX disease_name_txt IF NOT EXISTS FOR (d:Disease) ON (d.name)               // CONTAINS / ENDS WITH
CREATE FULLTEXT INDEX disease_ft IF NOT EXISTS FOR (d:Disease) ON EACH [d.name]            // fuzzy search
```

## Full-text search
```cypher
CALL db.index.fulltext.queryNodes('disease_ft', 'alzh*') YIELD node, score
RETURN node.name, score
```
- Lucene syntax: `*` wildcard, `~` fuzzy (`alzheimer~`)

## List and drop
```cypher
SHOW CONSTRAINTS
SHOW INDEXES
DROP CONSTRAINT protein_symbol IF EXISTS
DROP INDEX tracer_nuclide IF EXISTS
```

## Which constraints exist in Community?
| Constraint      | Syntax                                 | Edition         |
|-----------------|----------------------------------------|-----------------|
| Unique          | `REQUIRE n.x IS UNIQUE`                | Community       |
| Existence       | `REQUIRE n.x IS NOT NULL`              | Enterprise      |
| Property type   | `REQUIRE n.x IS :: FLOAT`              | Enterprise 5.9+ |
| Node key        | `REQUIRE (n.a, n.b) IS NODE KEY`       | Enterprise      |

# Notes
- Create constraints **before** the first import — `MERGE` without an index on the merge key gets extremely slow with many nodes
- Keep them in a `schema.cypher` file in the repo and run it during setup (see [run_neo4j_in_docker.md](run_neo4j_in_docker.md))
- In Community, existence/type checks have to happen in the pipeline (e.g. pydantic/pandera in Python)
- Use your own stable IDs (HGNC, UniProt, MONDO) as unique keys — never `elementId()`, it changes after reimport and gets reused after deletion
