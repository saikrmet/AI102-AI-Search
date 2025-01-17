# Azure AI Search


## Key Components of Azure AI Search

### **1. Replicas and Partitions**
- **Replicas**: Instances of the search service.
  - Ensure high capacity and availability.
  - Useful for concurrent query processing and indexing operations.
- **Partitions**: Divide index storage for better input/output (I/O) performance.
  - Helps with querying or rebuilding an index efficiently.
- **Search Units (SU)**: Measure the resources allocated for the search service.
  - Formula: `SU = Replicas × Partitions`.
  - Example: 4 replicas and 3 partitions = 12 SU.

---

### **2. Data Source**
- **Starting point for data to be indexed.**
- Supported data sources:
  - **Azure Blob Storage**: Unstructured files.
  - **Azure SQL Database**: Tables.
  - **Cosmos DB**: Documents.
- Data can also be pushed directly into an index using JSON without relying on an existing data store.

---

### **3. Skillset**
- Defines a pipeline for enriching data during indexing using AI capabilities.
- AI-driven insights include:
  - **Language Detection**: Determines the document's language.
  - **Key Phrases**: Extracts main themes or topics.
  - **Sentiment Analysis**: Assigns a positivity/negativity score.
  - **Named Entities**: Identifies locations, people, organizations, etc.
  - **OCR**: Extracts text from images (optical character recognition).
  - **Image Descriptions**: AI-generated captions for images.
  - **Custom Skills**: User-defined skills for specific requirements.

---

### **4. Indexer**
- **Engine that drives the indexing process.**
- Functions:
  - Maps enriched data and metadata to index fields.
  - Automatically runs when created and can be scheduled or run on demand.
  - Requires index reset if new fields or skills are added.
- Works with data and skillset outputs to create a searchable index.

---

### **5. Index**
- **Searchable result of indexing.**
- JSON document structure includes fields such as:
  - **`metadata_storage_name`**: Name of the stored document.
  - **`metadata_author`**: Document’s author.
  - **`content`**: Text content of the document.
- Field attributes:
  - **Key**: Unique record identifier.
  - **Searchable**: Usable in full-text search.
  - **Filterable**: Filters results based on constraints.
  - **Sortable**: Allows results to be ordered by values.
  - **Facetable**: Displays discrete values for filtering.
  - **Retrievable**: Included in search results (default for all fields unless excluded).

---

## Indexing Process

### **Document Structure During Indexing**
1. **Base Document**:
   - Initial fields from the source data:
     ```json
     {
       "metadata_storage_name": "file1.pdf",
       "metadata_author": "Author Name",
       "content": "This is the document text."
     }
     ```
2. **Enrichment**:
   - AI-driven skill outputs added to the document:
     ```json
     {
       "metadata_storage_name": "file1.pdf",
       "metadata_author": "Author Name",
       "content": "This is the document text.",
       "language": "en",
       "key_phrases": ["document text", "example"],
       "sentiment_score": 0.85
     }
     ```
3. **Image Normalization**:
   - For documents with images, normalized images are included:
     ```json
     {
       "metadata_storage_name": "file1.pdf",
       "metadata_author": "Author Name",
       "content": "This is the document text.",
       "normalized_images": {
         "image0": { "Text": "Image 1 extracted text" },
         "image1": { "Text": "Image 2 extracted text" }
       }
     }
     ```
4. **Hierarchical Merging**:
   - Skills such as OCR or key phrase extraction are applied to combine original text and enriched fields:
     ```json
     {
       "metadata_storage_name": "file1.pdf",
       "metadata_author": "Author Name",
       "content": "This is the document text.",
       "normalized_images": {
         "image0": { "Text": "Image 1 extracted text" },
         "image1": { "Text": "Image 2 extracted text" }
       },
       "merged_content": "This is the document text. Image 1 extracted text. Image 2 extracted text."
     }
     ```

---

### **Field Mapping**
- **Implicit Mapping**: Automatically maps source fields to index fields of the same name.
- **Explicit Mapping**: Custom mapping to rename fields or apply transformations during mapping.


---

## Searching the Index


### **1. Full Text Search**
- Uses **Lucene Query Syntax**:
  - **Simple**: For basic queries (e.g., match literal terms).
  - **Full**: For advanced filtering and regular expressions.
- **Query Parameters**:
  - `search`: Query terms.
  - `queryType`: Type of query syntax (simple/full).
  - `searchFields`: Index fields to search.
  - `select`: Fields to include in results.
  - `searchMode`: Criteria for term matching (`Any` or `All`).

---

### **2. Query Process**
1. **Parsing**: Breaks down query into subqueries (e.g., terms, phrases).
2. **Lexical Analysis**: Refines terms (e.g., lowercasing, stemming).
3. **Document Retrieval**: Matches terms with indexed data.
4. **Scoring**: Assigns relevance scores using TF/IDF calculations.

---

### **3. Filtering**
- Methods:
  - **Simple Filtering**:
    ```plaintext
    search=London+author='Reviewer'
    ```
  - **Full Filtering**:
    ```plaintext
    search=London
    $filter=author eq 'Reviewer'
    ```
- **Facets**:
  - Retrieve discrete values for filtering:
    ```plaintext
    search=*
    facet=author
    ```

---

### **4. Sorting**
- Default: Results sorted by relevance.
- Custom: Use `$orderby` with fields:
  ```plaintext
  search=*
  $orderby=last_modified desc
  ```



## Enhancements and Customization

### **1. Search-as-you-Type**
- **Suggestions**: Displays results while typing.
- **Autocomplete**: Completes partial terms.
- Requires a **suggester** defined for index fields.

---

### **2. Custom Scoring**
- Modify relevancy with scoring profiles:
  - Boost recent documents or specific field values.
- Example:
  - Recent documents have higher scores.

---

### **3. Synonym Maps**
- Links related terms (e.g., "UK", "United Kingdom", "Great Britain").
- Applied to fields to broaden search inclusivity.

### Field Mappings vs Output Field Mappings
- In the indexer JSON definition, field mappings correlate to document content and metadata, while output field mappings 
correlate to values extracted by skills in the skillset

## Coding Notes

- If you choose to create your schemas via REST, you will indexer, skillset, and index JSON definitions and PUT requests.

- If you want to add fields to index, follow these steps:
    1. Update index fields with field name, type, and attributes
    2. If fields require AI services (skillset), update the skillset definition to include the context hierarchy, the inputs necessary for the skillset and their sources (context), and the outputs. NOTE: the 'targetName' value in the skillset definition will be used in the path of 'sourceFieldName' for the indexer definition
    3. Update the indexer with the 'sourceFieldName' and 'targetFieldName' and rerun the indexer  


- The skillset file will need the key of your Azure AI Service resource. Instead of pasting the key, 
which can be unsecure and deprecated if the key changes, follow these steps to authenticate via system assigned managed identity:
    1. In your AI Search resource, turn on system assigned managed identity.
    2. In your Key Vault resource, add "Get Secrets" access policy for the AI Search resource.
    3. In your AI Services resource, add the Cognitive Services User as a role assignment in Access Control.
    4. In the skillsets.json script, @odata.type is always #Microsoft.Azure.Search.AIServicesByIdentity.
    5. In the skillsets.json script, subdomainUrl is the endpoint of your Azure AI multi-service resource. 
    - Use this link for more inforation: https://learn.microsoft.com/en-us/azure/search/cognitive-search-attach-cognitive-services?tabs=cogkey-rest%2Ccogkey-rest-remove