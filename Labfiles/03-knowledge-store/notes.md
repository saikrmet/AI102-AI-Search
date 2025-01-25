# Knowledge Stores in Azure AI Search


## Use Cases for Knowledge Stores
- **Exporting JSON**:
  - The index is essentially a collection of JSON objects. These can be exported as JSON files for integration with tools like **Azure Data Factory**.
- **Relational Schema**:
  - Normalizing index records into relational tables allows analysis and reporting using tools such as **Microsoft Power BI**.
- **Image Extraction**:
  - Extracted images during indexing can be saved as files for further use.


## Data Projections in Knowledge Stores
- **Projections**: Persist some or all fields of the enriched documents into a knowledge store.  
  - Projections are based on the document structures generated during the enrichment process.
  - Projections can include objects, tables, and files.


## Using the Shaper Skill

### Purpose
- Simplifies complex document schemas created during enrichment.
- Maps fields into a simpler, well-structured JSON object for easier projection.

### Shaper Skill Definition
Example:
```json
{
  "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
  "name": "define-projection",
  "description": "Prepare projection fields",
  "context": "/document",
  "inputs": [
    {
      "name": "file_name",
      "source": "/document/metadata_content_name"
    },
    {
      "name": "url",
      "source": "/document/url"
    },
    {
      "name": "sentiment",
      "source": "/document/sentimentScore"
    },
    {
      "name": "key_phrases",
      "source": null,
      "sourceContext": "/document/merged_content/keyphrases/*",
      "inputs": [
        {
          "name": "phrase",
          "source": "/document/merged_content/keyphrases/*"
        }
      ]
    }
  ],
  "outputs": [
    {
      "name": "output",
      "targetName": "projection"
    }
  ]
}
```

### Resulting JSON
The Shaper skill generates a structured JSON field `projection`:
```json
{
  "file_name": "file_name.pdf",
  "url": "https://<storage_path>/file_name.pdf",
  "sentiment": 1.0,
  "key_phrases": [
    { "phrase": "first key phrase" },
    { "phrase": "second key phrase" },
    { "phrase": "third key phrase" }
  ]
}
```


## Defining a Knowledge Store

### Knowledge Store Object
- Defined in skillset definition at the name/skills level.
- Requires:
  - **Storage Connection String**: Links to an Azure Storage account.
  - **Projections**: Specifies objects, tables, and files to be stored.

### Projection Types
1. **Object Projections**:
   - Exports structured JSON data into a storage container.
2. **Table Projections**:
   - Converts data into tabular format for relational joins and reporting.
   - **Key Fields**: Generated unique keys can link relational tables.
3. **File Projections**:
   - Stores files (e.g., extracted images) in storage containers.

### Example Knowledge Store Definition
```json
"knowledgeStore": {
  "storageConnectionString": "<storage_connection_string>",
  "projections": [
    {
      "objects": [
        {
          "storageContainer": "<container>",
          "source": "/projection"
        }
      ],
      "tables": [],
      "files": []
    },
    {
      "objects": [],
      "tables": [
        {
          "tableName": "KeyPhrases",
          "generatedKeyName": "keyphrase_id",
          "source": "projection/key_phrases/*"
        },
        {
          "tableName": "docs",
          "generatedKeyName": "document_id",
          "source": "/projection"
        }
      ],
      "files": []
    },
    {
      "objects": [],
      "tables": [],
      "files": [
        {
          "storageContainer": "<container>",
          "source": "/document/normalized_images/*"
        }
      ]
    }
  ]
}
```

### Key Notes on Projections
- **Mutually Exclusive**: Projections must specify either objects, tables, or files within each definition. If you want projections for all three types, you will need three projection objects in the projections array to hold each type. 
- **Automatic Creation**:
  - Containers for objects/files and tables are created automatically if they do not exist.
- **Relational Joins**:
  - Keys are generated (e.g., `keyphrase_id`, `document_id`) to enable relational joins in tables for analysis. 
- **No changes to index/indexer**: When creating a knowledge store, the index and indexer definitions remain the same. The shaper skill and the knowledgeStore definition are stored in the skillset definition.

