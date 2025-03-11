### **Vector Search in AI Search**
*Vector search enables indexing, storing, and retrieving vector embeddings for AI-powered search applications.*

---

### **When to Use Vector Search**
- **Text Encoding & Retrieval**: Use OpenAI or open-source models to encode text and retrieve documents via query vectors.
- **Similarity Search**: Compare encoded images, text, video, and audio, or a mix of these (multi-modal search).
- **Multi-Lingual Retrieval**: Find documents across different languages using a multilingual embedded model.
- **Hybrid Search**: Combine vector search with traditional text search at the field level for comprehensive results.
- **Filtered Search**: Apply filters on text and numeric fields to optimize query performance.
- **Vector Database**: Store long-term knowledge or external knowledge bases using a vector database.

---

### **Limitations**
- **Embedding Generation**: Azure AI Search does not generate embeddings; you must provide them using Azure OpenAI or an open-source solution.
- **Security Constraint**: Customer Managed Keys (CMK) are not supported.

---

### **Checking for Vector Fields in Your Search Index**
- Run an **empty search** to check for a vector field with a **number array**.
- Look for a field named **vectorSearch** with type **Collection(Edm.single)**.
- Ensure it has an **algorithm configuration** and a **dimension attribute**.
- Note: If you use ada-002, the **dimension value will be 1536** as the vector space has that many dimensions.

---

### **Converting Query Input into a Vector**
- Queries to a vector field must use a **query vector**.
- Convert user text queries into vectors using the **same embedding model** used for indexing (e.g., `text-embedding-ada-002`).
- Results return documents with **matching vector embeddings** from the search index.

---

### **Embedding Models**
- **Similarity Search Embeddings**: Capture semantic similarity between texts.
- **Text Search Embeddings**: Match a short query to a long document for relevance.
- **Code Search Embeddings**: Enable natural language searches for code snippets.
- **Consistent Model Use**: Ensure the same model is used for both **indexing** and **querying** to get accurate results.

---

### **Embedding Space**
- **Definition**: The space consisting of vector fields from the same embedding model.
- **Proximity Matters**: Similar concepts are **closer** in the embedding space, while dissimilar concepts are **farther** apart.
  - Example: **Hotels with water parks** are close together; **hotels without water parks** are farther but still within the "hotel" space.
  - **Restaurants** would be much farther away in the embedding space.

