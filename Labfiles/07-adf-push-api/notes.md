
### **Push Data Using Azure Data Factory (ADF)**
- **Zero-code solution** for integrating external data.
- **ADF supports 100+ data stores** via built-in connectors (HTTP, REST, etc.).
- **Azure AI Search index connector** acts as a sink in the copy activity.

#### **Steps to Create an ADF Pipeline**
1. **Create an Azure AI Search index** with required fields.
2. **Set up a pipeline** with a copy data activity.
3. **Create a data source connection** for the original data.
4. **Configure the sink** to connect to the search index.
5. **Map fields** from the source data to the search index.
6. **Run the pipeline** to push data into the index.

#### Creating pipeline in portal: https://learn.microsoft.com/en-us/training/modules/search-data-outside-azure-platform-cognitive-search/02-index-data-from-external-data-sources-factory/?ns-enrollment-type=learningpath&ns-enrollment-id=learn.wwl.implement-knowledge-mining-azure-cognitive-search

#### **Limitations of Azure AI Search as a Linked Service**
- **No support for ComplexTypes or arrays.**  
  - Example: Only the first phone number of a customer can be mapped.

---

### **Push Data Using the REST API**
- **Most flexible approach** for indexing data.
- Works with **any programming language** or tools that send JSON POST requests.

#### **Supported REST API Operations**
| **Feature** | **Operations** |
|------------|--------------|
| **Index** | Create, delete, update, configure |
| **Document** | Get, add, update, delete |
| **Indexer** | Configure data sources, schedule indexing |
| **Skillset** | Get, create, delete, update, list |
| **Synonym Map** | Get, create, delete, update, list |

---

### **Add Data to an Index Using REST API**
- Use **HTTP POST request** to the endpoint:  
  ```
  POST https://[service-name].search.windows.net/indexes/[index-name]/docs/index?api-version=[api-version]
  ```
- **JSON request format:**
  ```json
  {
    "value": [
      {
        "@search.action": "upload | merge | mergeOrUpload | delete",
        "key_field_name": "unique_key_of_document",
        "field_name": "field_value"
      }
    ]
  }
  ```

#### **Actions**
| **Action** | **Description** |
|------------|--------------|
| **upload** | Upsert (insert or replace) a document |
| **merge** | Update specific fields of an existing document (fails if not found) |
| **mergeOrUpload** | Update or insert if the document doesn’t exist |
| **delete** | Removes a document using the key field |

- **Success Response:** HTTP **200 OK**

---

### **Factors Affecting Index Performance**
1. **Service Tier & Resources**  
   - More **replicas and partitions** improve performance.
2. **Index Schema Complexity**  
   - Reduce the number of **searchable, facetable, and sortable properties**.
3. **Batch Size**  
   - Optimal size depends on schema and document size.
4. **Multithreading**  
   - Improve efficiency by running indexing in parallel.
5. **Error & Throttling Handling**  
   - Use an **exponential backoff retry strategy**.
6. **Data Location**  
   - Index data **close to Azure Search** for better performance.

---

### **Exponential Backoff Retry Strategy**
- **Handles request throttling** (503 or 207 errors).
- **Pauses before retrying**, increasing wait time on consecutive failures.
- Helps **reduce overload** and improves success rates.

