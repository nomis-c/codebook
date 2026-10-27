# Neo4j in Docker: Run, Connect, Back Up

## Start a container
```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/password123 \
  -v "$PWD/neo4j-data:/data" \
  -v "$PWD/import:/import" \
  neo4j:5
```
- `7474` = Neo4j Browser → `http://localhost:7474`
- `7687` = Bolt, for code and cypher-shell → `bolt://localhost:7687`
- `/data` volume keeps the database across container restarts
- `/import` volume is where `LOAD CSV` reads from
- Password must be at least 8 characters

### With plugins (APOC, Graph Data Science)
```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/password123 \
  -e NEO4J_PLUGINS='["apoc","graph-data-science"]' \
  -v "$PWD/neo4j-data:/data" \
  -v "$PWD/import:/import" \
  neo4j:5
```

## Start / stop / logs
```bash
docker stop neo4j
docker start neo4j
docker logs -f neo4j
```

## Run Cypher from the terminal
```bash
docker exec -it neo4j cypher-shell -u neo4j -p password123          # interactive
docker exec -i  neo4j cypher-shell -u neo4j -p password123 < schema.cypher   # run a file
```
- Separate statements in the file with `;` (the last one may omit it)
- Leave cypher-shell with `:exit`

## Back up and restore (dump / load)
```bash
mkdir -p backups && chown 7474:7474 backups     # neo4j-admin runs as uid 7474 in the container
docker stop neo4j

# dump
docker run --rm \
  -v "$PWD/neo4j-data:/data" -v "$PWD/backups:/backups" \
  neo4j:5 neo4j-admin database dump neo4j --to-path=/backups

# load (overwrites the current database)
docker run --rm \
  -v "$PWD/neo4j-data:/data" -v "$PWD/backups:/backups" \
  neo4j:5 neo4j-admin database load neo4j --from-path=/backups --overwrite-destination=true

docker start neo4j
```
- Community Edition can only dump a **stopped** database → stop the container, run `neo4j-admin` in a throwaway container on the same data volume
- Creates `backups/neo4j.dump`
- Without the `chown` the dump fails with `AccessDeniedException: /backups` (prefix with `sudo` if you're not root)

## Reset everything
```bash
docker rm -f neo4j
rm -rf neo4j-data
```
- `neo4j-data` belongs to uid 7474 → `sudo rm -rf` if you're not root
- Small DB: `MATCH (n) DETACH DELETE n` is enough, but constraints and indexes survive that

## Notes
- Community has only one user database (`neo4j`) — `CREATE DATABASE` is Enterprise only
- Pin a version (`neo4j:5.26`) for reproducible projects instead of the moving `neo4j:5` tag
- Treat the DB as a build artifact: schema in `schema.cypher`, data loaded by a pipeline script → can be regenerated any time
