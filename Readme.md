# Advance RAG 
Use Langgraph framework, we can implements advance RAG easier. There are some advance RAG: Agentic RAG, SelfRAG, CRAG and SQL Agent



## Agentic RAG
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AgenticRAG_workflow.png)
Use an agent to consider use RAG or not and retrieve the most relevant information before using the retrieved information to answer the user's question (Check first).

## Adaptive RAG
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AdaptvieRAG_architecture.png)
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/AdaptiveRAG_workflow.png)
Adaptive RAG use a strategy that whether use self-corrective RAG to retrieve document or use web search tool (Route data source first). [paper](https://arxiv.org/abs/2403.14403)

## CRAG (Corrective RAG)
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/CorrectiveRAG_architecture.png)
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/CorrectiveRAG_workflow.png)
Uses an LLM to grade the quality of the retrieved information from the given source. If the quality is low, it will try to retrieve the information from another source. [paper](https://arxiv.org/pdf/2401.15884.pdf)
## Self RAG
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SelfRAG_architecture.png)
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SelfRAG_workflow.png)
Self RAG use a stategy of self-reflection / self-grading. [paper](https://arxiv.org/abs/2310.11511)

## SQL Agent
[!Image](https://github.com/ganoliz/AdvanceRAG/blob/main/images/SQLAgent_workflow.png)
Build a agent that can do some search and can answer questions about a SQL database.
