# Neo4j Setup and Installation Guide

This document explains how to install, configure, and prepare Neo4j Desktop for the graph-based social network topology analysis used in this research.

---

# Neo4j Version

This research uses:

- Neo4j Desktop 2
- Bolt Protocol Connection

---

# 1. Install Neo4j Desktop

## Download Neo4j Desktop

Download Neo4j Desktop from the official website:

https://neo4j.com/download/

Run the installer and complete the installation process according to your operating system.

Supported platforms:
- Windows
- macOS
- Linux

---

# 2. Create a New Local Database

After opening Neo4j Desktop:

1. Click `New`
2. Select `Create a Local DBMS`
3. Configure:
   - Database Name
   - Username
   - Password

---

# 3. Start the Database

Click the `Start` button to run the database instance.

When successfully started:
- Neo4j Browser becomes available
- Bolt connection becomes active

Default Bolt URI:

```text
bolt://localhost:7687
```

---

# 4. Configure Python Connection

The following configuration is used inside the notebook:

```python
URI = "bolt://localhost:7687"
AUTH = ("neo4j", "your_password")
DATABASE_NAME = "your_db_name"
```
---

# 5. Import CSV Dataset into Neo4j

Place the processed CSV dataset inside the Neo4j import directory.

Example:

```text
processed_dataset.csv
```

Default Neo4j import location:

```text
<Neo4j Installation Folder>/import/
```

---

# 6. Verify Database Connection

Run the following Python code to verify the Neo4j connection:

```python
from neo4j import GraphDatabase

URI = "bolt://localhost:7687"
AUTH = ("neo4j", "your_password")

driver = GraphDatabase.driver(URI, auth=AUTH)

with driver.session() as session:
    result = session.run("RETURN 'Neo4j Connected Successfully' AS message")
    
    for record in result:
        print(record["message"])

driver.close()
```
---

# 7. Execute Cypher Queries

Cypher queries used in this research are available in:

```text
docs/neo4j/cypher-queries.md
```
---

# 8. Additional Notes

- Ensure Neo4j Desktop is running before executing notebooks
- Ensure the Bolt URI matches the notebook configuration
- Ensure CSV files are placed inside the Neo4j import directory
- Large graph datasets may require additional memory allocation in Neo4j settings

---

# 9. References

- Neo4j Official Documentation:
  https://neo4j.com/docs/