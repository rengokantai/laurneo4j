#### psurneo4j
#####6 Creating Entities with cypher
match a movie
```
MATCH (k:Person {name:"Kevin"})-[:ACTED_IN]->(movie) RETURN DISTINCT movie.title;
```
create movie
```
CREATE (m:Movie {title:"T:,released:1999});
```
match a person
```
MATCH (m:Movie {title:"T"})<-[:ACTED_IN]-(a) RETURN a.name;
```
return this movie
```
MATCH(m:Movie {title:"T"}) RETURN m;
```
add properties to existing node. /update a existing value
```
MATCH(m:Movie {title:"T"}) SET m.newprop="test" RETURN m;
```

creating relationships
```
MATCH (m:Movie{title:"T"}),(k:Person{name:"K"}) MERGE (k)-[:ACTED_IN]->(m);
```

creating relationships and set action prop
```
MATCH (m:Movie{title:"T"}),(k:Person{name:"K"}) MERGE (k)-[r:ACTED_IN]->(m) ON CREATE SET r.roles=["n'];
```
update relationship
```
MATCH (k:Person{name:"K"})-[r:ACTED_IN]->(m:Movie{title:"T"}) SET r.roles=["n'] return r.roles;
```
add role
```
MATCH (k:Person{name:"K"})-[r:ACTED_IN]->(m:Movie{title:"T"}) SET r.roles=[n in r.roles WHERE n <> "n"]="m" return r.roles;
```
#####7 Deleting Entities and Using Adv
######Delete
warmup: all roles in movie T
```
MATCH (m:Movie{tille:"T"})<-[r:ACTED_IN]-() return r.roles;
```
simple regex
```
MATCH (m:Movie{tille:"T"})<-[r:ACTED_IN]-(k) WHERE k.name ~= ".*k.*" RETURN k;
```
delete
```
MATCH {k:Person {name:"K"}) DELETE k; //error.must delete relationship first
```
delete rela
```
MATCH {k:Person {name:"K"})-[r]-() DELETE r; //delete relationship,no arrow
MATCH {k:Person {name:"K"}) DELETE k; //success
```
challenge: all actors in same movie
```
MATCH (a:Person)-[:ACTED_IN]->()<-[:ACTED_IN]-(b:Person) CREATE UNIQUE (a)-[:KNOWS]-(b);
```
######Using the merge
```
MERGE (k:Person {name:"m"}) RETURN k;
```
create or match if does not exist
```
MERGE (k:Person {name:"k"}) ON CREATE SET k.created = timestamp() ON MATCH SET k.acc = coalesce(k.a,0)+1 RETURN k;
```

challenge: all actors in same movie
```
MATCH (a:Person)-[:ACTED_IN|:DIRECTED]->()<-[:ACTED_IN|:DIRECTED]-(b:Person) CREATE UNIQUE (a)-[:KNOWS]-(b);
```
######Exploring var len path
```
(a)-[*1..3]->(b)
```
find length=2
```
MATCH (k:Person {name:"T"})-[:KNOWS*2]-(fof) RETURN DISTINCT fof.name;
```
find friend of friend and not immediate friend
```
MATCH (k:Person {name:"T"})-[:KNOWS*2]-(fof) WHERE k <> fof AND NOT (k)-[:KNOWS]-(fof) RETURN DISTINCT fof.name;
```
######finding shortest path
```
MATCH p =shortestPath((a:Person)-[:KNOWS*]-(b:Person)) WHERE a.name="a" AND b.name="b" RETURN length(p);
```
######cahllenge, get all names
```
MATCH p =shortestPath((a:Person)-[:KNOWS*]-(b:Person)) WHERE a.name="a" AND b.name="b" RETURN [ n in nodes(p) | n.name] AS names;
```
