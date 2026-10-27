# Cypher: Variable-Length Paths and Shortest Path

## Variable-length patterns
| Pattern                              | Meaning                          |
|--------------------------------------|----------------------------------|
| `(a)-[*2]-(b)`                       | exactly 2 hops                   |
| `(a)-[*1..3]-(b)`                    | 1 to 3 hops                      |
| `(a)-[:INTERACTS_WITH*..4]-(b)`      | up to 4 hops of one type         |
| `(a)-[*]-(b)`                        | any length — avoid, can hang     |
| `p = (a)-[*1..3]-(b)`                | bind the path to a variable      |

```cypher
MATCH p = (:Tracer {name: '[18F]FDG'})-[*1..3]-(d:Disease)
RETURN p
```

## Shortest path
```cypher
MATCH p = shortestPath(
  (a:Protein {symbol: 'SLC2A1'})-[:INTERACTS_WITH*..6]-(b:Protein {symbol: 'HK2'})
)
RETURN length(p), [n IN nodes(p) | n.symbol] AS path
```

### All shortest paths (ties included)
```cypher
MATCH p = allShortestPaths(
  (a:Protein {symbol: 'SSTR2'})-[:INTERACTS_WITH*..6]-(b:Protein {symbol: 'APP'})
)
RETURN p
```

## New syntax: quantified path patterns (5.21+)
```cypher
// 1–3 hops via INTERACTS_WITH
MATCH (a:Protein {symbol: 'SLC2A1'}) (()-[:INTERACTS_WITH]-()){1,3} (b:Protein)
RETURN DISTINCT b.symbol

// shortest path, -+ = one or more hops
MATCH p = SHORTEST 1 (a:Protein {symbol: 'SLC2A1'})-[:INTERACTS_WITH]-+(b:Protein {symbol: 'HK2'})
RETURN length(p)
```
- `{1,3}` = 1 to 3 repetitions, `+` = 1 or more, `*` = 0 or more
- `SHORTEST k` / `ALL SHORTEST` replace `shortestPath()` / `allShortestPaths()` (old ones still work)

## Work with a path
| Function               | Result               |
|------------------------|----------------------|
| `length(p)`            | number of edges      |
| `nodes(p)`             | list of nodes        |
| `relationships(p)`     | list of edges        |

Write to every node on a path:
```cypher
MATCH p = shortestPath((a:Protein {symbol: 'SLC2A1'})-[:INTERACTS_WITH*..6]-(b:Protein {symbol: 'HK2'}))
FOREACH (n IN nodes(p) | SET n.onPath = true)
```

Filter on nodes along a path:
```cypher
WHERE any(n IN nodes(p) WHERE n:Disease)
WHERE none(n IN nodes(p) WHERE n:Modality)
WHERE none(n IN nodes(p) WHERE coalesce(n.symbol, '') = 'TP53')
```
- Property checks need `coalesce()` on mixed paths: a node without `symbol` makes `n.symbol = 'TP53'` → `null` → `none()` returns `null` → the row is dropped silently

# Notes
- Always set an upper bound (`*..4`, `{1,4}`) — unbounded paths explode on dense graphs
- Supernodes (nodes with thousands of edges, e.g. a `:Modality {name: 'PET'}` node) make traversals slow
- For network statistics over many pairs (distances, permutation tests) → export to NetworkX/igraph, see [python_driver.md](python_driver.md)
