# Connecting and Querying Knowledge Graph using Neo4j Cloud and Python Driver

This guide outlines the steps to connect to and query a knowledge graph using Neo4j Cloud and the Python Neo4j driver. The example demonstrates the use of Pandas for data manipulation and the official Neo4j driver for Python.

## Dependencies

### Pandas (pd)

Pandas is a powerful data manipulation library in Python. It is used here to read and manipulate CSV data.

### Neo4j Driver

The neo4j module provides the official Neo4j driver for Python. It facilitates communication with a Neo4j database.

``` python
import pandas as pd
from neo4j import GraphDatabase
```

## Neo4j Connection Details

``` python
uri = "neo4j+s://68a3ea64.databases.neo4j.io"
username = "neo4j"
password = "tgRxNc1xksKPESo2Nci8e_MKOw1jEKUFPMneUOWxbnw"
```

**Neo4j Connection Details:**
- `uri`: Uniform Resource Identifier specifying the Neo4j database's address (in this case, a Neo4j Cloud instance).
- `username` and `password`: Credentials for authentication.

## Reading CSV Data

``` python
jobs_df = pd.read_csv('/content/new_synthetic_job_posting_dataset.csv')
candidates_df = pd.read_csv('/content/new_synthetic_resume_dataset.csv')
```

**Reading CSV Data:**
- `pd.read_csv`: Reads CSV files into Pandas DataFrames.

## Connecting to Neo4j

``` python
with GraphDatabase.driver(uri, auth=(username, password)) as driver:
    with driver.session() as session:
```

**Connecting to Neo4j:**
- `GraphDatabase.driver`: Creates a Neo4j driver instance.
- `driver.session()`: Establishes a session for database interactions.

## Cypher Query for Creating Nodes (Jobs)

``` python
for index, job in jobs_df.iterrows():
    create_job_query = """
    MERGE (j:Job {job_id: $job_id, title: $title, skills: $skills, experience: $experience, major: $major, location: $location})
    """
    session.run(create_job_query, job_id=job['Job ID'], title=job['Job Title'], skills=job['Required Skills'], experience=job['Experience Needed'], major=job['Major'], location=job['Location'])
```

**Cypher Query for Creating Nodes (Jobs):**
- `MERGE`: Ensures a node with the specified properties doesn't already exist.
- Parameters (`$job_id`, `$title`, `$skills`, etc.) are filled using the Pandas DataFrame.

## Cypher Query for Creating Nodes (Candidates)

``` python
for index, candidate in candidates_df.iterrows():
    create_candidate_query = """
    MERGE (c:Candidate {candidate_id: $candidate_id, name: $name, skills: $skills, experience: $experience, major: $major, location: $location})
    """
    session.run(create_candidate_query, candidate_id=candidate['Candidate ID'], name=candidate['Name'], skills=candidate['Skills'], experience=candidate['Experience'], major=candidate['Major'], location=candidate['Location'])
```

**Cypher Query for Creating Nodes (Candidates):**
- Similar to the job node creation, creates nodes for candidates.

## Cypher Query for Creating Relationships

``` python
create_relationship_query = """
MATCH (j:Job), (c:Candidate)
WHERE j.skills CONTAINS c.skills AND j.experience >= c.experience AND j.location = c.location
MERGE (j)-[:REQUIRES_COMMON_ATTRIBUTES]->(c)
"""
session.run(create_relationship_query)
```

**Cypher Query for Creating Relationships:**
- `MATCH`: Retrieves job and candidate nodes based on specified conditions.
- `WHERE`: Filters nodes based on common attributes.
- `MERGE`: Creates a relationship between the matched job and candidate nodes labeled "REQUIRES_COMMON_ATTRIBUTES."

These Python modules (Pandas and Neo4j) are used to read data, connect to a Neo4j Cloud database, and execute Cypher queries to create nodes and relationships in the graph. The Cypher queries define the graph structure and relationships based on common attributes between jobs and candidates.
