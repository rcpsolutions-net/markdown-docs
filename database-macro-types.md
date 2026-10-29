# From SQL to AI: A Quick Glance At Different Database Architectures

*Prompted and edited by Lawrence Ham*

> **Original Question:** "Explain the differences between a relational database, nosql database and a vector database. Also, include any similarities and closest matching database compared to way human memory works for reference"

## 1. Relational Database (MS-SQL, PostgreSQL, MySQL)

- **Core idea:** predefined tables of rows/columns; strict data integrity via relationships.
- **Structure:** schema-on-write — define the schema before adding data; primary/foreign keys.
- **Query:** SQL for precise, exact-criteria matches.
- **Best for:** financial systems, e-commerce, inventory — anywhere ACID consistency and a stable model matter.
- **Analogy:** a library's **card catalog** — cross-referenced, find by exact title/author/number.

## 2. NoSQL Database (MongoDB, Redis)

- **Core idea:** flexible storage beyond rigid tables; prioritizes scale and speed.
- **Structure:** schema-on-read. Most common model is the **document store** (JSON-like); others include key-value, column-family, graph.
- **Query:** per-DB APIs/languages; whole documents or key-value lookups.
- **Best for:** big data, real-time feeds, CMS, rapid development + horizontal scaling.
- **Analogy:** **digital file folders** — each folder can hold a different collection of items.

## 3. Vector Database (Pinecone, Milvus, Chroma)

- **Core idea:** organizes data by *semantic meaning*, not literal attributes.
- **Structure:** unstructured data (text/images/audio) → **vector embeddings** via an AI model; similar meanings are stored "close" together in a high-dimensional space.
- **Query:** **similarity search** — the query is also vectorized; returns nearest neighbors.
- **Best for:** the engine behind modern AI — semantic search, image-to-image search, recommendations, RAG for LLMs.
- **Analogy:** a **galaxy of stars** — navigate to a region of space (a concept) and find the closest constellation.

## Comparison & Similarities

| Feature | Relational | NoSQL | Vector |
| :--- | :--- | :--- | :--- |
| **Data Model** | Tables (rows & columns) | Documents, key-value, etc. | Vector embeddings |
| **Schema** | Strict, defined upfront | Flexible, dynamic | Flexible, vector dimensions |
| **Primary Use** | Transactions, structured data | Web-scale apps, big data | AI, semantic search, similarity |
| **Query Type** | Exact matches (SQL) | Key lookups, document scans | Similarity search (ANN) |
| **Key Strength** | Consistency & integrity | Scalability & flexibility | Context & meaning |

Despite differences, all three provide **data persistence**, support **CRUD**, and use **indexing** to accelerate queries.

## Closest Match to Human Memory

Human memory works by **associative recall** — a network of connections based on context, feeling, and conceptual closeness.

- **Relational** ≈ memory for hard facts ("What is your address?") — precise but no associative power.
- **NoSQL Graph** is a step closer — explicit relationships ("Bob is friends with Alice").
- **Vector** is the **closest match to the *retrieval mechanism* of human associative memory** — finding information by "vibe" or a cluster of related concepts rather than a direct query.

## Understanding Vectors: The Language of Modern AI

- **A vector is a list of numbers representing a point in a space.** In 2D that's `[x, y]`; AI vectors have hundreds/thousands of dimensions.
- **"Converting to a vector" plots data in a *meaning space*.** The numbers aren't physical distances — they're conceptual characteristics. (Example: a 3-dim "fruit space" of sweetness/crunchiness/sourness puts apple `[0.8, 0.9, 0.3]` and lemon `[0.1, 0.2, 0.9]` far apart.)
- **The converter is an *embedding model*.** Pre-trained on massive text/image corpora, it outputs a vector whose dimensions are abstract learned concepts (formality, sentiment, animal-relatedness…). Words/sentences with similar meanings get similar values across those dimensions.

### The Final "Meaning"

> When a vector database runs a query: it uses a pre-trained AI model to transform a piece of data (text or image) into a list of numbers — a **vector embedding** — a mathematical representation of the data's underlying meaning and context. Once the query is encoded, a **similarity search** retrieves the nearest neighboring vectors in the database.

***This is revolutionary because it allows us to perform mathematical operations on meaning itself. By calculating the "distance" between two vectors, a database can find similar concepts without ever matching keywords — finding a match based on the essence of the data.***
