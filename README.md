#### psurneo4j
#####3 Understanding
```
MATCH (a)-(r)->() return a.name, type(r);
```
#####4 Getting prop from paths
get full path
```
MATCH p=(a)-[:ACTED_IN]->(m) RETURN p;
MATCH p=(a)-[:ACTED_IN]->(m) q=(d)-[:DIRECTED]->(m) RETURN p q;
```

only show relationship
```
MATCH p=(a)-[:ACTED_IN]->(m) RETURN rels(p);
```
######Modify aggregate
```
MATCH(a)-[:ACTED_IN]->(m)<-[:DIRECTED]-(d) RETURN a.name,d.name,count(*);   //return number of m fulfill both a and b
```

list function.return all 
```
MATCH(a)-[:ACTED_IN]->(m)<-[:DIRECTED]-(d) RETURN a.name,d.name,list(m);
```
#####5 Using specific nodes for a cypher
basic: return all.
```
MATCH(n) RETURN n;
```
director work with K:
```
MATCH (k:Person{name:"K"})-[r:ACTED_IN]->(m),(director)-[:DIRECTED]->(movie) RETURN director.name  //or
MATCH (k:Person)-[r:ACTED_IN]->()<-[:DIRECTED]->(director) WHERE k.name="K" RETURN director.name
```
create index:
```
CREATE INDEX ON :Person(name)   //like :table(attr)
```
######Handling conditions
```
MATCH(t:Person{name:"K"})-[:ACTED_IN]->(m:Moive) WHERE m.release=1111 RETURN DISTINCT m.title
```
in
```
MATCH(t:Person{name:"K"})-[r:ACTED_IN]->(m:Moive) WHERE "N" IN (r.roles) RETURN DISTINCT m.title
```
another syntax
```
MATCH(t:Person{name:"K"})-[r:ACTED_IN]->(m:Moive) WHERE ANY (x IN r.roles WHERE x="N") RETURN DISTINCT m.title
```


######Handling on pattern
```
MATCH (k:Person{name:"A"})-[:ACTED_IN]->(m)<-[:ACTED_IN]-(actor), (j:Person{name:"B"}) WHERE NOT(j)-[:ACTED_IN]->(m) RETURN DISTINCT actor.name;
```

######find 5 busiest actor
```
MATCH (a:Person)-[:ACTED_IN]->() RETURN a.name,count(*) AS count ORDER BY count DESC LIMIT 5;
```

#####find 3 actors should work with a. (look for people work with a, then find other people woek with these people)
```
MATCH (a:Person)-[:ACTED_IN]->()<-[:ACTED_IN]-(c), (c)-[:ACTED_IN]->()<-[:ACTED_IN]-(coc) WHERE a.name="A"
AND NOT((a)-[:ACTED_IN]->()<-[:ACTED_IN]-(coc)) AND coc<>a RETURN coc.name,count(coc) ORDER BY count(coc) DESC LIMIT 3;
```

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
