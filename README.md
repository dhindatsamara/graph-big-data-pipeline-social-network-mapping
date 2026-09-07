# Graph-Based Big Data Pipeline Architecture for Social Network Topology Mapping
## A Case Study of the 2025 Indonesian Parliament Protests
### Undergraduate Thesis Research Artifact

---

## 1. Executive Summary

This repository hosts the source code, data assets, and analytical workflows developed as part of an undergraduate thesis research project. It presents an end-to-end graph analytics pipeline designed to map and quantify social network topology. Using public discourse surrounding the 2025 Indonesian Parliament protests on the Twitter/X platform as an empirical case study, this project models unstructured social interactions into a property graph schema.

The analytical architecture covers raw data acquisition, tabular data cleaning, graph database construction via Neo4j, network centrality and community detection using the Neo4j Graph Data Science (GDS) library, multi-criteria rank aggregation via WASPAS and Borda Count, and interactive analytical reporting using NeoDash.

This repository serves as a verifiable and reproducible research artifact accompanying the corresponding undergraduate thesis and scientific publication.

---

## 2. Research and Analytical Objectives

The primary objectives of this analytical system are:

* Design a reproducible pipeline to ingest, clean, and convert unstructured microblog interaction data into an enterprise property graph model.
* Construct an attributed directed network capturing relational dynamics between users, posts, and topical hashtags.
* Quantify structural influence and connectivity patterns using foundational Graph Data Science metrics, including Degree Centrality, PageRank, Betweenness Centrality, and Louvain Community Detection.
* Address multi-metric rank discrepancy by synthesizing individual centrality metrics into unified actor importance rankings via WASPAS (Weighted Aggregated Sum Product Assessment) and Borda Count algorithms.
* Provide an interactive visualization layer to profile community clusters, top-tier actors, and network dispersion metrics.

---

## 3. End-to-End Pipeline Architecture

The pipeline processes data across five sequential tiers:

1. Data Ingestion: Scrapes interaction records, user identifiers, and textual metadata via targeted query terms.
2. ETL and Transformation: Normalizes schemas, strips redundant noise, parses hashtag vectors, and exports graph-aligned CSV files.
3. Graph Persistence: Imports relational data into Neo4j with unique node constraints and indexes for query optimization.
4. Graph Algorithmic Computation: Computes global and local topology indicators within the Neo4j GDS engine and writes scores to node properties.
5. Analytical Synthesis: Retrieves computed scores into Python for WASPAS and Borda rank aggregation, updates the database, and exposes visualizations via NeoDash.

```text
+-------------------+      +----------------------+      +----------------------+
|  Data Ingestion   | ---> | ETL & Preprocessing  | ---> |   Neo4j Ingestion    |
| (Twitter/X Crawl) |      | (Pandas / Cleaning)  |      | (Constraints/Import) |
+-------------------+      +----------------------+      +----------------------+
                                                                    |
                                                                    v
+-------------------+      +----------------------+      +----------------------+
| NeoDash Reporting | <--- |   Rank Aggregation   | <--- |  Graph Data Science  |
| (Visual Insights) |      |   (WASPAS / Borda)   |      |  (Centrality / GDS)  |
+-------------------+      +----------------------+      +----------------------+
```

### 3.1 Architectural Diagram

![Pipeline Architecture](docs/figures/pipeline-architecture.png)

---

## 4. Repository Structure

```text
graph-big-data-pipeline-social-network-mapping/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── code/
│   ├── CrawlingData.ipynb          # Raw data acquisition workflow
│   ├── PreprocessingData.ipynb     # Data sanitation, normalization, and CSV preparation
│   └── RankAggregation.ipynb      # Multi-criteria decision analysis (WASPAS and Borda)
│
├── data/
│   ├── raw_data.csv                # Unprocessed crawling output
│   └── preprocessed_data.csv       # Cleaned, semicolon-delimited dataset for Neo4j import
│
└── docs/
    ├── figures/
    │   ├── pipeline-architecture.png
    │   ├── graph-schema.png
    │   ├── neodash-page1.png
    │   ├── neodash-page2.png
    │   └── neodash-page3.png
    │
    └── neo4j/
        ├── neo4j-installation-guide.md
        └── cypher-queries.md
```

---

## 5. Data Specifications and Graph Schema

### 5.1 Dataset Metadata

* Raw Dataset (`data/raw_data.csv`): Contains raw tweet extractions, timestamps, raw interaction indicators, and account references.
* Processed Dataset (`data/preprocessed_data.csv`): Fully cleaned, semicolon-delimited (`opt: ;`) dataset optimized for Neo4j `LOAD CSV` operations. All entity references have been verified, duplicate statuses removed, and text fields sanitized.

### 5.2 Property Graph Schema

![Graph Schema](docs/figures/graph-schema.png)

The structural configuration uses labeled property nodes connected by directed relationships:

#### Node Labels
* `User`: Identified uniquely by `id`. Represents social accounts generating or receiving interaction.
* `Post`: Identified uniquely by `id`. Represents individual microblog entries.
* `Hashtag`: Identified uniquely by normalized text value (`text`).

#### Relationship Types
* `(:User)-[:POSTS]->(:Post)`: Identifies post authorship.
* `(:Post)-[:REPLY_TO]->(:Post)`: Identifies conversational response threads.
* `(:Post)-[:MENTIONS]->(:User)`: Captures direct user mentions within content.
* `(:Post)-[:USES_HASHTAG]->(:Hashtag)`: Links thematic categories to posts.

---

## 6. Analytical and Algorithmic Methodology

### 6.1 Graph Centrality Algorithms

Graph analytics are computed via Neo4j Graph Data Science (GDS) procedures using projection graphs:

1. Degree Centrality: Measures local interaction volume by tallying incoming and outgoing adjacent edges.
2. PageRank: Measures recursive network authority, accounting for both the quantity and relative significance of incoming connections.
3. Betweenness Centrality: Identifies bridge nodes controlling shortest information paths across separate network clusters.
4. Louvain Modularity: Uncovers natural community partitioning by maximizing modularity scores iteratively.

### 6.2 Multi-Criteria Rank Aggregation

Because individual centrality metrics often yield conflicting rankings for key actors, multi-criteria rank aggregation is employed:

#### WASPAS (Weighted Aggregated Sum Product Assessment)
Combines the Weighted Sum Model (WSM) and Weighted Product Model (WPM) using a joint utility coefficient:
* Matrix normalization is applied across Degree, PageRank, and Betweenness metrics.
* Relative attribute weights are assigned based on analytical priority.
* The composite score determines final influence rankings, mitigating extreme metric bias.

#### Borda Count
An ordinal positional voting mechanism where actors receive points based on their rank position across each individual centrality metric. The sum of Borda scores provides an alternative, non-parametric consensus ranking.

Both calculated scores are written back into the Neo4j instance to enrich `User` nodes.

---

## 7. Environment Requirements and Dependencies

### 7.1 Runtime Environments
* Python 3.10 or higher
* Neo4j Desktop 2.x (Neo4j Enterprise or Community Server 5.x)
* Neo4j Graph Data Science (GDS) Plugin 2.x
* NeoDash 2.4+ (Neo4j Desktop Graph App or Web deployment)

### 7.2 Core Python Libraries
* `neo4j` (Official Bolt Driver)
* `pandas`
* `numpy`
* `jupyter` / `ipykernel`

---

## 8. Replication Protocol

To replicate the analytical pipeline from source, execute the workflow in the following sequence:

1. Environment Configuration:
   * Set up a local DBMS in Neo4j Desktop following instructions in `docs/neo4j/neo4j-installation-guide.md`.
   * Install the Graph Data Science (GDS) plugin from the DBMS Plugins menu.
   * Place `data/preprocessed_data.csv` into the DBMS `import` directory.

2. Database Initialization:
   * Open Neo4j Browser or Cypher Shell.
   * Execute schema constraints and indexes from `docs/neo4j/cypher-queries.md` (Section A0).
   * Execute batch `LOAD CSV` commands from Section A1 to populate nodes and relationships.
   * Run validation and enrichment queries from Sections B and C.

3. Graph Analytics Execution:
   * Execute GDS projection and algorithm queries from Section E of `docs/neo4j/cypher-queries.md`.
   * Ensure `degreeScore`, `pagerankScore`, and `betweennessScore` are written to `User` nodes.

4. Multi-Criteria Synthesis:
   * Open `code/RankAggregation.ipynb`.
   * Configure Bolt connection credentials (`URI`, `AUTH`, `DATABASE_NAME`).
   * Run all cells to compute WASPAS and Borda aggregations, then persist updated scores back to Neo4j.

5. Visual Analytics and Reporting:
   * Launch NeoDash and connect to the active DBMS instance.
   * Load dashboard configurations to observe global topology summaries, multi-criteria rankings, and community structures.

---

## 9. Visual Analytics Dashboard (NeoDash)

The following dashboard views illustrate the visual analytics layer built to synthesize graph metrics and community patterns:

### 9.1 Network Topology Overview
![Network Topology Overview](docs/figures/neodash-page1.png)

### 9.2 Multi-Criteria Actor Ranking
![Multi-Criteria Actor Ranking](docs/figures/neodash-page2.png)

### 9.3 Distribution and Community Profiling
![Distribution and Community Profiling](docs/figures/neodash-page3.png)

---

## 10. Analytical Constraints and Data Notes

* API Retweet Restrictions: Retweet interactions were excluded from relational graph generation because contemporary platform endpoints omit uniform retweet propagation metadata without enterprise-tier access.
* Bipartite Projections: Analysis between actors reflects explicit interaction events (mentions, replies, and shared hashtag topics) rather than passive follower-followee topologies.
* Dashboard Portability: NeoDash visualizations represent dynamic query visualizers and can be restructured based on user-defined Cypher aggregations.

---

## 11. Academic Context

This project represents the primary research artifact and computational engineering implementation for an Undergraduate Thesis. It is provided to ensure full academic transparency, replicability, and algorithmic auditability of the data pipeline, Graph Data Science procedures, and multi-criteria ranking models.

---

## 12. License

This research codebase is distributed under the MIT License. Refer to `LICENSE` for complete terms.