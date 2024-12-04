## Design and Implementation of a Multidocument Retrieval Agent Using LlamaIndex

## AIM:
To design and implement a multidocument retrieval agent using LlamaIndex to extract and synthesize information from multiple research articles, and to evaluate its performance by testing it with diverse queries, analyzing its ability to deliver concise, relevant, and accurate responses.

## PROBLEM STATEMENT:
As the volume of research papers increases, efficiently retrieving and synthesizing information from multiple documents remains a challenge. This project aims to address the problem by creating a system that uses the LlamaIndex framework to retrieve and synthesize data from various academic papers. The system is designed to respond accurately to complex queries and provide insightful comparisons of the extracted information.

## DESIGN STEPS:

#### STEP 1: Dataset Preparation
- Downloaded research papers in PDF format, such as MetaGPT, SWE-Bench, LongLoRA, and LoftQ.
- Mapped each paper to its respective retrieval and summarization tools using a utility function (get_doc_tools).
#### STEP 2: Tool Generation
- Created vector tools for document similarity analysis and summary tools for abstracting key content.
- Stored these tools for each document in a dictionary (paper_to_tools_dict).

#### STEP 3: Index Construction
- Used VectorStoreIndex and ObjectIndex from LlamaIndex to index the document tools.
- Configured a retriever to fetch the most relevant tools based on similarity to the query.

#### STEP 4: Query Handling
- Developed an AgentWorker to process queries and retrieve necessary tools using the configured retriever.
- Implemented a FunctionCallingAgentWorker to ensure all responses are tool-based without reliance on prior knowledge.
  
#### STEP 5: Evaluation
- Designed diverse queries for testing, such as comparing evaluation datasets used in MetaGPT and SWE-Bench, and analyzing approaches in LongLoRA and LoftQ.
- Synthesized concise and comparative responses using the agent system.

## PROGRAM:
```py
from helper import get_openai_api_key
OPENAI_API_KEY = get_openai_api_key()

import nest_asyncio
nest_asyncio.apply()

papers = [
    "docs/metagpt.pdf",
    "docs/longlora.pdf",
    "docs/selfrag.pdf",
]

from utils import get_doc_tools
from pathlib import Path

paper_to_tools_dict = {}
for paper in papers:
    print(f"Getting tools for paper: {paper}")
    vector_tool, summary_tool = get_doc_tools(paper, Path(paper).stem)
    paper_to_tools_dict[paper] = [vector_tool, summary_tool]

initial_tools = [t for paper in papers for t in paper_to_tools_dict[paper]]

from llama_index.llms.openai import OpenAI

llm = OpenAI(model="gpt-3.5-turbo")
len(initial_tools)

from llama_index.core.agent import FunctionCallingAgentWorker
from llama_index.core.agent import AgentRunner

agent_worker = FunctionCallingAgentWorker.from_tools(
    initial_tools,
    llm=llm,
    verbose=True
)
agent = AgentRunner(agent_worker)

# The agent can then choose the appropriate tools for each steps
response = agent.query(
    "Tell me about the evaluation dataset used in LongLoRA, "
    "and then tell me about the evaluation results"
)
# we can chat/query about the LLM's reasoning process
response = agent.chat(
    "Tell me about the tools you leverage to answer the above question."
)

from utils import get_doc_tools
from pathlib import Path

paper_to_tools_dict = {}
for paper in papers:
    print(f"Getting tools for paper: {paper}")
    vector_tool, summary_tool = get_doc_tools(paper, Path(paper).stem)
    paper_to_tools_dict[paper] = [vector_tool, summary_tool]

all_tools = [t for paper in papers for t in paper_to_tools_dict[paper]]

# define an "object" index over these tools
from llama_index.core import VectorStoreIndex
from llama_index.core.objects import ObjectIndex

obj_index = ObjectIndex.from_objects(
    all_tools,
    index_cls=VectorStoreIndex,
)

# define the "retriever" over the index with specified retrieval method
obj_retriever = obj_index.as_retriever(similarity_top_k=3)

tools = obj_retriever.retrieve(
    "Tell me about the eval dataset used in MetaGPT and longLora"
)

# check for the top 3 tools selected
tools[0].metadata

# Define the agentWorker and agentRunner
from llama_index.core.agent import FunctionCallingAgentWorker
from llama_index.core.agent import AgentRunner

agent_worker = FunctionCallingAgentWorker.from_tools(
    tool_retriever=obj_retriever,
    llm=llm,
    system_prompt=""" \
You are an agent designed to answer queries over a set of given papers.
Please always use the tools provided to answer a question. Do not rely on prior knowledge.\

""",
    verbose=True
)
agent = AgentRunner(agent_worker)

response = agent.query(
    "Tell me about the evaluation dataset used "
    "in MetaGPT and compare it against longLora"
)
print(str(response))

```
## OUTPUT:

![image](https://github.com/user-attachments/assets/7f539132-54bf-41f2-a705-8b510c4beee3)


## RESULT:
The multidocument retrieval agent was successfully designed and implemented using LlamaIndex. The agent demonstrated its capability to retrieve and synthesize information from multiple academic papers, answering complex queries with concise, relevant, and accurate responses.
