# API Guidelines for Custom Skillset in AI Search

#### Regardless of how your custom skillset function works or where it exists, in order to integrate it into the skillset definition, it must follow these guidelines:

## Input Schema for a Custom Skill

- **Structure**: A JSON object containing an array of documents to be processed.
- **Fields**:
  - **`values`**: An array where each element corresponds to one document.
  - **`recordId`**: A unique identifier for each document in the array.
  - **`data`**: A key-value pair object containing the inputs required for processing.

### Example Input Schema
```json
{
  "values": [
    {
      "recordId": "doc1",
      "data": {
        "text": "This is the first document.",
        "language": "en"
      }
    },
    {
      "recordId": "doc2",
      "data": {
        "text": "Este es el segundo documento.",
        "language": "es"
      }
    }
  ]
}
```

### Key Points:
- The `recordId` must be unique for each document in the array to track individual processing results.
- The `data` object can include multiple fields (`<input1_name>`, `<input2_name>`, etc.), which serve as inputs for the skill.

---

## Output Schema for a Custom Skill

- **Structure**: Similar to the input schema but with additional elements for errors and warnings.

### Example Output Schema
```json
{
  "values": [
    {
      "recordId": "doc1",
      "data": {
        "translatedText": "This is the first document translated.",
        "sentimentScore": 0.95
      },
      "errors": [],
      "warnings": []
    },
    {
      "recordId": "doc2",
      "data": {
        "translatedText": "Este es el segundo documento traducido.",
        "sentimentScore": 0.85
      },
      "errors": [],
      "warnings": ["Language detection confidence low."]
    }
  ]
}
```
---

# Steps for creating a Custom Text Classification Skillset

## Requirements
- **Blob Storage**: to store training data 
- **AI Language**: to build and train the model
    - **NOTE**: Make sure to select custom features when provisioning resource
- **Function App**: to pass in data for AI Language, run the model, and format output for AI Search
- **AI Search**: to create skillset, indexer, and index 

## A. Training the model
1. In the Blob Storage, add training data to container
2. In AI Language, connect the blob storage
    - A. In AI Language, turn on SA managed identity
    - B. In Blob storage under Access control, create Storage Blob Data Contributor role
    and assign it to the SA managed identity. 
    - NOTE: If you have a virtual network or private endpoint, be sure to select Allow Azure services on the trusted services list to access this storage account in the Azure portal.
3. In Language Studio, create a project and connect the container
4. In Language Studio, split/auto-split the data and classify training data in the portal or JSON file

## B. Skillset Definition
Along with the other skillsets like LanguageDetection, the skillset file will contain a Custom.WebApiSkill which has an input of data from the front-end
and outputs the formatted response from the function app. Here is what would go in the skillset file:
``` 
{
  "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
  "name": "Genre Classification",
  "description": "Identify the genre of your movie from its summary",
  "context": "/document",
  "uri": "https://learn-acs-lang-serives.cognitiveservices.azure.com/language/analyze-text/jobs?api-version=2022-05-01",
  "httpMethod": "POST",
  "timeout": "PT30S",
  "batchSize": 1,
  "degreeOfParallelism": 1,
  "inputs": [
    {
      "name": "lang",
      "source": "/document/language"
    },
    {
      "name": "text",
      "source": "/document/content"
    }
  ],
  "outputs": [
    {
      "name": "text",
      "targetName": "class"
    }
  ],
  "httpHeaders": {}
}
```
**When the indexer runs this, it sends a POST request with the inputs to the URI of the Function App.**


## C. Function App
**The function app sends another POST request to the AI Language resource prediction URL with the following:**
1. The text to be classified: Received from WebApiSkill POST request and sent in body of prediction POST request
2. The endpoint for your trained custom text classification deployed model: Found in AI Language and sent in header of prediction POST request
3. The primary key for the custom text classification project: Found in AI Language and sent in header of prediction POST request
4. The project name: Found in AI Language and sent in body of prediction POST request
5. The deployment name: Found in AI Language and sent in body of prediction POST request

Here is an example of the prediction POST request body:
```
{
    "displayName": "Extracting custom text classification", 
    "analysisInput": {
        "documents": [
            {
                "id": "1", 
                "language": "en-us", 
                "text": "This film takes place during the events of Get Smart. Bruce and Lloyd have been testing out an invisibility cloak, but during a party, Maraguayan agent Isabelle steals it for El Presidente. Now, Bruce and Lloyd must find the cloak on their own because the only non-compromised agents, Agent 99 and Agent 86  are in Russia"
            }
        ]
      }, 
    "tasks": [
        {
        "kind": "CustomMultiLabelClassification", 
        "taskName": "Multi Label Classification", 
        "parameters": {
            "project-name": "movie-classifier", 
            "deployment-name": "test-release"}
        }
    ]
}
```
Here is an example of the prediction POST response body:
```
{
  "jobId": "be1419f3-61f8-481d-8235-36b7a9335bb7",
  "lastUpdatedDateTime": "2022-06-13T16:24:27Z",
  "createdDateTime": "2022-06-13T16:24:26Z",
  "expirationDateTime": "2022-06-14T16:24:26Z",
  "status": "succeeded",
  "errors": [],
  "displayName": "Extracting custom text classification",
  "tasks": {
    "completed": 1,
    "failed": 0,
    "inProgress": 0,
    "total": 1,
    "items": [
      {
        "kind": "CustomMultiLabelClassificationLROResults",
        "taskName": "Multi Label Classification",
        "lastUpdateDateTime": "2022-06-13T16:24:27.7912131Z",
        "status": "succeeded",
        "results": {
          "documents": [
            {
              "id": "1",
              "class": [
                {
                  "category": "Action",
                  "confidenceScore": 0.99
                },
                {
                  "category": "Comedy",
                  "confidenceScore": 0.96
                }
              ],
              "warnings": []
            }
          ],
          "errors": [],
          "projectName": "movie-classifier",
          "deploymentName": "test-release"
        }
      }
    ]
  }
}
```
**The function app must then process the response and ultimately return a JSON message like the below. This is critical for the skillset response and index definition.**
```
[{"category": "Action", "confidenceScore": 0.99}, {"category": "Comedy", "confidenceScore": 0.96}]
```

## D. Index and Indexer
In order for the skillset response to be stored in the index, the index definition must contain fields for the data. Here is an example
where the category and confidence score fields are stored together as a compound field. This processes the data together and should be used
for data that is logically grouped together for easier processing.
```
{
  "name": "classifiedtext",
  "type": "Collection(Edm.ComplexType)",
  "analyzer": null,
  "synonymMaps": [],
  "fields": [
    {
      "name": "category",
      "type": "Edm.String",
      "facetable": true,
      "filterable": true,
      "key": false,
      "retrievable": true,
      "searchable": true,
      "sortable": false,
      "analyzer": "standard.lucene",
      "indexAnalyzer": null,
      "searchAnalyzer": null,
      "synonymMaps": [],
      "fields": []
    },
    {
      "name": "confidenceScore",
      "type": "Edm.Double",
      "facetable": true,
      "filterable": true,
      "retrievable": true,
      "sortable": false,
      "analyzer": null,
      "indexAnalyzer": null,
      "searchAnalyzer": null,
      "synonymMaps": [],
      "fields": []
    }
  ]
}
```
Finally, the indexer "outputFieldMappings" definition needs to be edited so it knows how to map the "class" output from the skillset to the index:
```
{
  "sourceFieldName": "/document/class",
  "targetFieldName": "classifiedtext"
}
```

**Now, we can run the indexer manually or through a script, which will extract the metadata and map it to the index via fieldMappings as well as running the skillsets (including the custom text classification model) and map it to the index via outputFieldMappings.**
