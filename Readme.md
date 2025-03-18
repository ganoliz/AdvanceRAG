# Advance RAG 

![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/RAG_mechanism.png)

Use Langgraph framework, we can implements advance RAG easier. There are some advance RAG implements: Agentic RAG, SelfRAG, CRAG and SQL Agent in this repository.



## Agentic RAG
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AgenticRAG_workflow.png)

Use an agent to consider use RAG or not and retrieve the most relevant information before using the retrieved information to answer the user's question (Check first). After we retrieve documents, we use LLM to grade those documents that are relate to user query or not. If not, we re-write query to form an improved question.

Re-write question prompt example:
```python
prompt = """ \n 
    Look at the input and try to reason about the underlying semantic intent / meaning. \n 
    Here is the initial question:
    \n ------- \n
    {question} 
    \n ------- \n
    Formulate an improved question: """
```

## Adaptive RAG
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AdaptvieRAG_architecture.png)
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AdaptiveRAG_workflow.png)

Adaptive RAG use a strategy that whether use self-corrective RAG to retrieve document or use web search tool (Route data source first). You can also have more data source. Then we do same things like Self-RAG: Retrieve and grade documents are relevant or not. When generate, we grade generated answer is hallucination or not by another llm. If content is hallucination, we need to re-generate again. At final, we need to check the answer is actually answer user questions or not, if not we need re-write question and retrieve again. [paper](https://arxiv.org/abs/2403.14403)


Route prompt example:
```python
system = """You are an expert at routing a user question to a vectorstore or web search.
The vectorstore contains documents related to agents, prompt engineering, and adversarial attacks.
Use the vectorstore for questions on these topics. Otherwise, use web-search."""
```

Grade relevant prompt example:
```python
system = """You are a grader assessing relevance of a retrieved document to a user question. \n 
    If the document contains keyword(s) or semantic meaning related to the user question, grade it as relevant. \n
    It does not need to be a stringent test. The goal is to filter out erroneous retrievals. \n
    Give a binary score 'yes' or 'no' score to indicate whether the document is relevant to the question."""
```

Grade Hallucination prompt example:
```python
system = """You are a grader assessing whether an LLM generation is grounded in / supported by a set of retrieved facts. \n Give a binary score 'yes' or 'no'. 'Yes' means that the answer is grounded in / supported by the set of facts."""
```

Grade answer prompt example:
```python
system = """You are a grader assessing whether an answer addresses / resolves a question \n 
     Give a binary score 'yes' or 'no'. Yes' means that the answer resolves the question."""

```
Re-write question prompt example:

```python
system = """You a question re-writer that converts an input question to a better version that is optimized \n 
     for vectorstore retrieval. Look at the input and try to reason about the underlying semantic intent / meaning."""
```

 

## CRAG (Corrective RAG)
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/CorrectiveRAG_architecture.png)
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/CorrectiveRAG_workflow.png)

Uses an LLM to grade the quality of the retrieved information from the given source. If the quality is low, it will try to retrieve the information from another source. In this example if retrieved contents is not relevant (we skip knowledge refinement step in this example), we rewrite user query and then do web search and generate answer from sources. [paper](https://arxiv.org/pdf/2401.15884.pdf)

Re-write question to do web search prompt example:
```python
system = """You a question re-writer that converts an input question to a better version 
            that is optimized \n for web search. Look at the input and try to reason about the underlying semantic intent / meaning."""
```

## Self RAG
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SelfRAG_architecture.png)
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SelfRAG_workflow.png)

Self RAG use a stategy of self-reflection / self-grading. Check documents relevant, hallucinations, actually answer question is similiar to AdaptiveRAG we write above. [paper](https://arxiv.org/abs/2310.11511)

## SQL Agent
![image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SQLAgent_workflow.png)

Build a agent that can do some search and can answer questions about a SQL database.

We need two useful built-in tools
1. list_tables_tool: Fetch the available tables from the database
2. get_schema_tool: Fetch the DDL for a table

And then we can do SQL query
db_query_tool: Execute the query and fetch the results OR return an error message if the query fails

We need to check common SQL mistakes and correct it, here is a prompt example:
```python
query_check_system = """You are a SQL expert with a strong attention to detail.
Double check the SQLite query for common mistakes, including:
- Using NOT IN with NULL values
- Using UNION when UNION ALL should have been used
- Using BETWEEN for exclusive ranges
- Data type mismatch in predicates
- Properly quoting identifiers
- Using the correct number of arguments for functions
- Casting to the correct data type
- Using the proper columns for joins

If there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.

You will call the appropriate tool to execute the query after running this check."""
```

After we retrieve and analysis, we have the answer or some error when SQL query. Here is a prompt example for generate query and re-write SQL query to try if return error:
```python
query_gen_system = """You are a SQL expert with a strong attention to detail.

Given an input question, output a syntactically correct SQLite query to run, then look at the results of the query and return the answer.

DO NOT call any tool besides SubmitFinalAnswer to submit the final answer.

When generating the query:

Output the SQL query that answers the input question without a tool call.

Unless the user specifies a specific number of examples they wish to obtain, always limit your query to at most 5 results.
You can order the results by a relevant column to return the most interesting examples in the database.
Never query for all the columns from a specific table, only ask for the relevant columns given the question.

If you get an error while executing a query, rewrite the query and try again.

If you get an empty result set, you should try to rewrite the query to get a non-empty result set. 
NEVER make stuff up if you don't have enough information to answer the query... just say you don't have enough information.

If you have enough information to answer the input question, simply invoke the appropriate tool to submit the final answer to the user.

DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the database."""
```

Note that we create a tool that can execute query:
```python
@tool
def db_query_tool(query: str) -> str:
    """
    Execute a SQL query against the database and get back the result.
    If the query is not correct, an error message will be returned.
    If an error is returned, rewrite the query, check the query, and try again.
    """
    result = db.run_no_throw(query)
    if not result:
        return "Error: Query failed. Please rewrite your query and try again."
    return result
```