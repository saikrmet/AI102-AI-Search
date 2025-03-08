# **Semantic Ranking in Azure AI Search**  

Semantic ranking improves search results by using **language understanding**, enhancing relevance beyond keyword matching.  

## **BM25 vs. Semantic Ranking**  
- **BM25 (default):** Ranks based on keyword frequency but lacks semantic understanding.  
- **Semantic Ranking:** Re-ranks BM25 results using AI models to **extract meaning** and improve relevance.  

## **How Semantic Ranking Works**  
1. **BM25 selects the top 50 results.**  
2. **Text fields are processed** (trimmed to 256 tokens).  
3. **AI models analyze context** to determine relevance.  
4. **Results are re-ranked** based on semantic meaning.  
5. **Captions & answers (optional)** are extracted from documents.  

## **Semantic Captions & Answers**  
- **Captions:** Highlight relevant text from documents.  
- **Answers:** Provide direct responses if the query is a question.  

## **Limitations**  
- **Only refines BM25’s top 50 results.**  
- **Doesn’t retrieve additional documents.**  

## **Pricing**  
- **Free for up to 1,000 queries/month.**  
- **Standard pricing applies beyond this, based on usage and region.**  

## **How to Configure in Azure**  
1. Open **Azure Portal** → **Search Service** → **Indexes**.  
2. Go to **Semantic Configurations** → **Add Configuration**.  
3. Set **Title, Content, and Keyword fields** → **Save**.  
