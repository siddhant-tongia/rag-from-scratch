# 1. Indexing 

Before you can search for information, you have to store it. Indexing is the ETL (Extract, Transform, Load) pipeline of the AI world.

### Chunk Optimization (Semantic Splitter)
* **What it is:** You can't dump a 500-page PDF into a vector store all at once. You have to break it into "chunks." Instead of just cutting text every 500 characters (which might cut a sentence in half), a Semantic Splitter looks at the meaning of the sentences and splits them where the topic actually changes.
* **Why it's important:** Clean, coherent chunks mean the LLM gets precise context without useless filler.

### Multi-Representation Indexing (Parent Document Retriever)
* **What it is:** You summarize a large chunk of text into a short, dense summary. You embed and search the summary, but when a match is found, you feed the entire parent document to the LLM.
* **Why it's important:** Small texts are easier to search accurately, but large texts give the LLM better context to write a rich answer.

### Specialized Embeddings (ColBERT)
* **What it is:** Standard embeddings convert an entire sentence into a single list of numbers (a vector). Models like ColBERT embed every single word individually.
* **Why it's important:** It allows for highly detailed, token-level matching, making search incredibly precise for specialized industries (like legal or medical tech).

### Hierarchical Indexing (RAPTOR)
* **What it is:** This technique clusters related chunks together, summarizes them, clusters those summaries, and summarizes them, building a tree-like structure of your data.
* **Why it's important:** If a user asks a high-level question like "What were the company's financial trends over the last 5 years?", standard RAG fails because the answer is spread across 50 pages. RAPTOR grabs the high-level summaries from the top of the tree to answer global questions perfectly.

---

# 2. Routing 

When a user types a question into your FastAPI endpoint, where should it go? You don't always want to hit your expensive vector database if the user just said "Hello!".

### Logical Routing
* **What it is:** An LLM reads the user’s question and decides which database to query. For example, if the question is about an order status, it routes to a SQL database. If it's about a company policy, it routes to a Vector Store.

### Semantic Routing
* **What it is:** You map the user's input against predefined prompt templates using embedding similarity.
* **Why it's important:** It saves a massive amount of money and speed (latency). You bypass complex LLM reasoning by using quick mathematical vector comparisons to guide the traffic.

---

# 3. Query Translation 

Users are notoriously bad at asking questions. If they type a poorly phrased query, standard keyword or vector searches will return terrible results. Query Translation rewrites the question before searching.

### Multi-Query / RAG-Fusion
The LLM takes one user question and rewrites it into 3 to 5 different variations. You run all of them through the database and combine the results.

### Decomposition
If a user asks a complex, multi-part question, the LLM breaks it down into sub-questions, searches for each individually, and aggregates them.

### Step-Back
The LLM generates a broader, more abstract question first (e.g., instead of asking about a specific error code, it asks about the underlying architecture principle) to gather foundational context.

### HyDE (Hypothetical Document Embeddings)
The LLM writes a fake, hypothetical answer to the user's question first. Then, you use that fake answer to search the database. Why? Because an answer looks more like the target document than a question does!

---

# 4. Query Construction 

Not all data is unstructured text. Sometimes your data lives in specific database types, and you need to convert natural language into database queries.

### Text-to-SQL (Relational DBs)
Translates "Who are our top 5 clients in New York?" into `SELECT client_name FROM clients WHERE city='NY' LIMIT 5;.`

### Text-to-Cypher (GraphDBs)
Converts questions into graph queries to find complex relationships between data points (e.g., fraud rings or social networks).

### Self-Query Retriever (VectorDBs)
Extracts metadata filters. If a user asks, "Show me articles about AI written in 2023", the LLM separates the semantic search term (AI) from the metadata filter (year == 2023) so your Vector DB searches efficiently.

---

# 5. Retrieval & Ranking

Once you query your databases, they might spit back 50 relevant-looking documents. You can't feed all 50 to the LLM (it's too expensive, slow, and causes "lost in the middle" confusion).

### Re-Ranking & RankGPT
Vector databases are fast but sometimes lack nuance. You grab the top 25 results quickly using a vector search, then use a slower, smarter Re-Ranker model (like Cohere Rerank or RankGPT) to pick the absolute top 5 best results.

### CRAG (Corrective RAG)
Evaluates the retrieved documents. If the system realizes the retrieved documents are completely irrelevant to the question, it triggers an alternate path, like spinning up a web search tool to find the answer on Google.

---

# 6. Generation 

This is the final mile: passing the perfect context alongside the user's question to the LLM to generate the final response.

### Active Retrieval / Self-RAG
The LLM doesn't just passively read the text you gave it. It grades its own output. It checks: Did I actually answer the user's question? Is my answer fully backed by the source documents, or did I hallucinate? If it fails its self-check, it loops back, rewrites the query, and tries again.

---

# Diagram