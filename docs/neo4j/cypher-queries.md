# Cypher Queries Documentation

This document contains the complete Cypher query collection used in the research:

> “A Graph-Based Big Data Pipeline Architecture for Social Network Topology Mapping: A Case Study of the 2025 Indonesian Parliament Protests”

The queries are organized sequentially to support:
- graph construction
- graph validation
- graph enrichment
- graph analytics
- Graph Data Science (GDS)

---

# Prerequisites

Before executing the queries:

1. Ensure Neo4j Desktop is installed
2. Ensure the database instance is running
3. Ensure the CSV dataset has been placed inside the Neo4j import directory
4. Ensure the Graph Data Science (GDS) plugin is installed

Detailed Neo4j setup instructions are available in:

```text
docs/neo4j/neo4j-installation-guide.md
```

---

# Dataset Preparation

The following dataset is required:

```text
data/preprocessed_data.csv
```

Dataset location inside Neo4j:

```text
<Neo4j Installation Folder>/import/
```

Required separator:

```text
;
```

---

# Recommended Execution Order

Execute the queries in the following order:

| Step | Section | Purpose |
|---|---|---|
| 1 | Section A | Graph Import |
| 2 | Section B | Data Validation |
| 3 | Section C | User Enrichment |
| 4 | Section D | Initial Exploration |
| 5 | Section E | Graph Data Science |
| 6 | Section F | Result Analysis |

---

# SECTION A — GRAPH IMPORT

This section constructs the graph database structure.

---

# A0 — Constraints and Indexes

Create constraints and indexes before importing the dataset.

```cypher
CREATE CONSTRAINT user_id IF NOT EXISTS
FOR (u:User)
REQUIRE u.id IS UNIQUE;

CREATE CONSTRAINT post_id IF NOT EXISTS
FOR (p:Post)
REQUIRE p.id IS UNIQUE;

CREATE CONSTRAINT hashtag_text IF NOT EXISTS
FOR (h:Hashtag)
REQUIRE h.text IS UNIQUE;

CREATE INDEX post_created IF NOT EXISTS
FOR (p:Post)
ON (p.created_at);

CREATE INDEX post_engagement IF NOT EXISTS
FOR (p:Post)
ON (p.engagement_score);

CREATE INDEX post_type IF NOT EXISTS
FOR (p:Post)
ON (p.tweet_type);

CREATE INDEX user_username IF NOT EXISTS
FOR (u:User)
ON (u.username);
```

---

# A1 — Create User Nodes

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.user_id_str IS NOT NULL

MERGE (u:User {
    id: row.user_id_str
});
```

---

# A2 — Create Post Nodes

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.id_str IS NOT NULL

CREATE (p:Post {
    id: row.id_str,
    text: row.full_text,
    created_at: datetime(row.created_at),
    favorite_count: toInteger(row.favorite_count),
    retweet_count: toInteger(row.retweet_count),
    reply_count: toInteger(row.reply_count),
    quote_count: toInteger(row.quote_count),
    lang: row.lang,
    url: row.tweet_url,
    has_image: toBoolean(row.has_image),
    engagement_score: toFloat(row.engagement_score),
    tweet_type: row.tweet_type,
    is_root_tweet: toBoolean(row.is_root_tweet)
});
```

---

# A3 — Create Hashtag Nodes

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.hashtags IS NOT NULL
AND row.hashtags <> ''

UNWIND split(row.hashtags, ',') AS tag

WITH trim(tag) AS clean_tag

WHERE clean_tag <> ''

MERGE (h:Hashtag {
    text: clean_tag
});
```

---

# A4 — Create POSTS Relationships

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

MATCH (u:User {
    id: row.user_id_str
})

MATCH (p:Post {
    id: row.id_str
})

CREATE (u)-[:POSTS]->(p);
```

---

# A5 — Create USES_HASHTAG Relationships

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.hashtags IS NOT NULL
AND row.hashtags <> ''

MATCH (p:Post {
    id: row.id_str
})

UNWIND split(row.hashtags, ',') AS tag

WITH p, trim(tag) AS clean_tag

WHERE clean_tag <> ''

MATCH (h:Hashtag {
    text: clean_tag
})

MERGE (p)-[:USES_HASHTAG]->(h);
```

---

# A6 — Create MENTIONS Relationships

## A6a — Mentions from Tweet Text

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.mentions IS NOT NULL
AND row.mentions <> ''

MATCH (p:Post {
    id: row.id_str
})

UNWIND split(row.mentions, ',') AS mention_raw

WITH p,
replace(trim(mention_raw), '@', '') AS username_clean

WHERE username_clean <> ''

MERGE (u:User {
    id: username_clean
})

MERGE (p)-[:MENTIONS]->(u);
```

---

## A6b — Reply Mention Relationships

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.replied_to_username IS NOT NULL
AND row.replied_to_username <> ''

MATCH (p:Post {
    id: row.id_str
})

WITH p, trim(row.replied_to_username) AS ruser

WHERE ruser <> ''

MERGE (u:User {
    id: ruser
})

MERGE (p)-[:MENTIONS]->(u);
```

---

# A7 — Create REPLY_TO Relationships

```cypher
LOAD CSV WITH HEADERS
FROM 'file:///preprocessed_data.csv' AS row
FIELDTERMINATOR ';'

WITH row
WHERE row.parent_id_str IS NOT NULL
AND row.parent_id_str <> ''

MATCH (child:Post {
    id: row.id_str
})

MATCH (parent:Post {
    id: row.parent_id_str
})

MERGE (child)-[:REPLY_TO]->(parent);
```

---

# SECTION B — DATA VALIDATION

This section validates graph integrity and imported properties.

---

# B1 — Graph Schema Visualization

```cypher
CALL db.schema.visualization();
```

---

# B2 — Node and Relationship Counts

```cypher
MATCH (u:User)
RETURN count(u) AS Total_User;

MATCH (p:Post)
RETURN count(p) AS Total_Post;

MATCH (h:Hashtag)
RETURN count(h) AS Total_Hashtag;

MATCH ()-[r]->()
RETURN type(r) AS Relationship_Type,
count(r) AS Total_Count;
```

---

# B3 — Verify Imported Properties

```cypher
MATCH (p:Post)

RETURN
    p.id,
    p.created_at,
    p.engagement_score,
    p.tweet_type,
    p.is_root_tweet

LIMIT 5;
```

---

# SECTION C — USER ENRICHMENT

This section calculates additional user statistics.

```cypher
MATCH (u:User)-[:POSTS]->(p:Post)

WITH
    u,
    count(p) AS total_posts,
    sum(p.favorite_count) AS total_favorites

SET
    u.total_posts = total_posts,
    u.total_favorites = total_favorites;
```

---

# SECTION D — INITIAL GRAPH EXPLORATION

This section provides quick exploratory analysis.

---

# D1 — Most Mentioned Users

```cypher
MATCH (u:User)<-[:MENTIONS]-(p:Post)

RETURN
    u.id AS Account,
    count(p) AS Total_Mentions

ORDER BY Total_Mentions DESC
LIMIT 10;
```

---

# D2 — Most Frequent Hashtags

```cypher
MATCH (h:Hashtag)<-[:USES_HASHTAG]-(p:Post)

RETURN
    h.text AS Hashtag,
    count(p) AS Frequency

ORDER BY Frequency DESC
LIMIT 10;
```

---

# SECTION E — GRAPH DATA SCIENCE (GDS)

This section performs graph projection and centrality analysis.

---

# E1 — Graph Projection

```cypher
CALL gds.graph.project(
    'sna',
    ['User', 'Post'],
    {
        POSTS: {orientation: 'NATURAL'},
        MENTIONS: {orientation: 'NATURAL'},
        REPLY_TO: {orientation: 'NATURAL'}
    }
);
```

---

# E2 — PageRank Centrality

```cypher
CALL gds.pageRank.write('sna', {
    writeProperty: 'pagerankScore',
    maxIterations: 50,
    dampingFactor: 0.85
})
YIELD nodePropertiesWritten, ranIterations;
```

---

# E3 — Louvain Community Detection

```cypher
CALL gds.louvain.write('sna', {
    writeProperty: 'communityId'
})
YIELD communityCount, modularity, modularities;
```

---

# E4 — Degree Centrality

```cypher
CALL gds.degree.write('sna', {
    writeProperty: 'degreeScore',
    orientation: 'UNDIRECTED'
})
YIELD nodePropertiesWritten;
```

---

# E5 — Betweenness Centrality

```cypher
CALL gds.graph.project(
    'sna_undirected',
    ['User', 'Post'],
    {
        POSTS: {orientation: 'UNDIRECTED'},
        MENTIONS: {orientation: 'UNDIRECTED'},
        REPLY_TO: {orientation: 'UNDIRECTED'}
    }
);

CALL gds.betweenness.write('sna_undirected', {
    writeProperty: 'betweennessScore'
})
YIELD nodePropertiesWritten, centralityDistribution;

CALL gds.graph.drop('sna_undirected', false);
```
---

# Notebook-Based Rank Aggregation

After completing the Graph Data Science (GDS) centrality analysis, the WASPAS and Borda rank aggregation processes are performed using the following notebook:

```text
code/RankAggregation.ipynb
```

The notebook performs:
- centrality score normalization
- WASPAS score calculation
- Borda Count aggregation
- final actor ranking
- Neo4j node enrichment

The resulting scores are then written back into the Neo4j database and used for graph analysis and visualization.

---

# SECTION F — RESULT ANALYSIS

This section provides graph analysis and interpretation queries after the Graph Data Science (GDS) and notebook-based rank aggregation processes have been completed.

---

# F1 — Total Nodes and Relationships

```cypher
MATCH (u:User)
RETURN count(u) AS Total_User;

MATCH (p:Post)
RETURN count(p) AS Total_Post;

MATCH (h:Hashtag)
RETURN count(h) AS Total_Hashtag;

MATCH ()-[r]->()
RETURN type(r) AS Relationship_Type,
count(r) AS Total_Count;
```

---

# F2 — Community Profiling

```cypher
MATCH (u:User)

WHERE u.communityId IS NOT NULL

WITH u.communityId AS Community, u

ORDER BY u.degreeScore DESC

WITH
    Community,
    collect(u.id)[0..3] AS Top_Members,
    count(u) AS Community_Size

RETURN
    Community,
    Community_Size,
    Top_Members

ORDER BY Community_Size DESC
LIMIT 10;
```

---

# F3 — Top Degree Centrality

```cypher
MATCH (u:User)

WHERE u.degreeScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.degreeScore AS Degree_Score

ORDER BY u.degreeScore DESC
LIMIT 10;
```

---

# F4 — Top PageRank Centrality

```cypher
MATCH (u:User)

WHERE u.pagerankScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.pagerankScore AS PageRank_Score

ORDER BY u.pagerankScore DESC
LIMIT 10;
```

---

# F5 — Top Betweenness Centrality

```cypher
MATCH (u:User)

WHERE u.betweennessScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.betweennessScore AS Betweenness_Score

ORDER BY u.betweennessScore DESC
LIMIT 10;
```

---

# F6 — Top WASPAS Ranking

```cypher
MATCH (u:User)

WHERE u.waspasScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.waspasScore AS WASPAS_Score

ORDER BY u.waspasScore DESC
LIMIT 10;
```

---

# F7 — Top Borda Ranking

```cypher
MATCH (u:User)

WHERE u.bordaScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.bordaScore AS Borda_Score

ORDER BY u.bordaScore DESC
LIMIT 10;
```

---

# F8 — Top Key Actors

```cypher
MATCH (u:User)

WHERE u.waspasScore IS NOT NULL

RETURN
    u.id AS Actor,
    u.communityId AS Community,
    u.waspasScore AS WASPAS_Score,
    u.bordaScore AS Borda_Score,
    u.degreeScore AS Degree,
    u.pagerankScore AS PageRank,
    u.betweennessScore AS Betweenness

ORDER BY u.waspasScore DESC
LIMIT 10;
```

---

# F9 — Community Distribution

```cypher
MATCH (u:User)

WHERE u.communityId IS NOT NULL

RETURN
    u.communityId AS Community,
    count(u) AS Total_Actors

ORDER BY Total_Actors DESC
LIMIT 10;
```

---

# Notes

- The graph structure focuses on:
  - mentions
  - replies
  - hashtag interactions
  - user-post relationships

---

# References

- Neo4j Documentation:
  https://neo4j.com/docs/

- Neo4j Graph Data Science:
  https://neo4j.com/docs/graph-data-science/