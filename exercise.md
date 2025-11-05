# 🧩 Agentic-AI Exercises

## Exercise 1: Add Brave Search MCP Server to VS Code Copilot

### 🛠️ Configuration Steps

1. [Enable VS Code MCP](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)  
2. Create a [Brave Search API account](https://github.com/brave/brave-search-mcp-server) — use the free plan.  
3. Generate an API key from the [Brave Developer Dashboard](https://api-dashboard.search.brave.com/app/keys).  
4. (Optional) Install **npx** or **Docker** if not already available.  
5. Configure the [Brave Search MCP Server](https://github.com/brave/brave-search-mcp-server#usage-with-vs-code).  
6. (Optional) Create a `.env` file and add your API key:  

    ```env
    BRAVE_API_KEY=your_api_key_here
    ```
    Update your mcp.json configuration to reference it:

    ```json
    "env": {
        "BRAVE_API_KEY": "${env:BRAVE_API_KEY}"
    }
    ```

### 💬 Exercise Tasks

1. Read about the different **chat modes** in VS Code Copilot:  
   [Copilot Ask, Edit, and Agent modes](https://github.blog/ai-and-ml/github-copilot/copilot-ask-edit-and-agent-modes-what-they-do-and-when-to-use-them/)

2. Using **Agent Mode**, try triggering a **Brave Search** through MCP.

3. Using **#brave##** to enforce tool use

4. Answer:  
   🔍 What keyboard shortcut or trigger activates the Brave Search integration?


## 🧩 Exercise 2: Add Brave Search MCP Server to Jan

### 🛠️ Configuration Steps

1. Download and install [Jan.ai](https://www.jan.ai/docs/desktop/mcp).  
2. Configure **Gemini API Key** and add the model `models/gemini-2.5-flash`  
   *(Settings → Model Providers → Gemini → under Models press **+** → ...)*  
3. Download **Model Jan-V1-4B-GGUF**  
   *(Hub → search “Model” → Download)*  
4. Enable **Tools** for models *Gemini* and *Jan-V1-4B-GGUF*  
   *(Settings → Model Providers → Gemini | Llama.cpp → under Models press edit pen → enable Tools)*  
5. Add **Brave Search MCP Server**  
   *(Settings → MCP Servers → Press **+** → Add Server by JSON by pressing `{...}` → copy npx or docker config from [Brave Search MCP Server](https://github.com/brave/brave-search-mcp-server?tab=readme-ov-file#usage-with-claude-desktop) → remove `"mcpServers": {` section → save)*  

### 💬 Exercise Tasks

Using **Jan**, try triggering a **Brave Search** through MCP.


## 🧩 Exercise 3: Update LangChain RAG Agent to Include the MCP PubMed Server to Ask Questions

### 🔧 Add MCP PubMed Server

1. Install **mcp_simple_pubmed** manually via pip:  
   [mcp_simple_pubmed](https://github.com/andybrandt/mcp-simple-pubmed?tab=readme-ov-file#manual-installation)

2. Create an account for [PubMed](https://pubmed.ncbi.nlm.nih.gov/).

3. Add **MCP Simple PubMed MCP Server** to **Jan**  
   *(Settings → MCP Servers → Press **+** → Add Server by JSON by pressing `{...}` → copy npx or docker config from [MCP Simple PubMed](https://github.com/andybrandt/mcp-simple-pubmed?tab=readme-ov-file#usage-with-claude-desktop) → remove `"mcpServers": {` section → save → add registered email)*  

4. Add **MCP Simple PubMed MCP Server** to **VS Code Copilot**  
   *In `.vscode/mcp.json` add npx or docker config from [MCP Simple PubMed](https://github.com/andybrandt/mcp-simple-pubmed?tab=readme-ov-file#usage-with-claude-desktop) → remove `"mcpServers": {` section → save → add registered email*  

5. Test by calling the server using **Jan** and **VS Code Copilot**.

---

### 🧠 Update LangChain RAG Agent to Include the MCP PubMed Server

**Starting project:** [RAG-Agent-exercise](https://github.com/zhaw-iwi/RAG-Agent-exercise)

#### 1. Add Email to `.env`

```env
PUB_MED_REGISTRED_MAIL=your_registered_email
```

2. Load the Email in rag_agent_annotated.ipynb

```python
pub_med_mail = os.getenv("PUB_MED_REGISTRED_MAIL")
```

3. Configure MCP Client with MCP Simple PubMed and Create Tools

(Note: You may need to change ".venv/bin/python" to the environment where mcp-simple-pubmed is installed.)

```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from langchain.chat_models import init_chat_model
from langgraph.graph import MessagesState

client = MultiServerMCPClient(
    {
        "simple-pubmed": {
            "transport": "stdio",
            "command": ".venv/bin/python",
            "args": ["-m", "mcp_simple_pubmed"],
            "env": {"PUBMED_EMAIL": pub_med_mail},
        }
    }
)

tools = await client.get_tools()

```

4. Modify generate_query_or_respond to Include Your Tools.
```python
def generate_query_or_respond(state: MessagesState):
    """Call the model to generate a response based on the current state. Given
    the question, it will decide to retrieve using the retriever tool, or simply respond to the user.
    """
    response = (
        response_model
        .bind_tools(tools+[retriever_tool]).invoke(state["messages"])  
    )
    return {"messages": [response]}
```

5. Experiment:

Change your query to observe how different keywords affect which tool (retriever_tool or PubMed tool) is used by generate_query_or_respond.


## 🧩 Exercise 4: Design a SQLite-MCP-Server to Handle Requests for a SQLite Database

### 📘 Project Context

Base your work on the project:  
[Table-Augmented-Generation-Exercise](https://github.com/zhaw-iwi/Table-Augmented-Generation-Exercise/tree/main)

As a group, **design an MCP Server** that can handle requests for a **SQLite database**.  
Your MCP Server should replicate and extend the functionality demonstrated in the [Table-Augmented-Generation-Exercise](https://github.com/zhaw-iwi/Table-Augmented-Generation-Exercise/tree/main).

---

### 🧩 Objective

Design a **SQLite MCP Server** that enables:
- Connecting to a local SQLite database  
- Querying tables and retrieving structured data  
- Returning responses through MCP-compatible tools and resources  

---

### 🧠 Key Concepts to Apply

#### **For the MCP Server:**
From the [Model Context Protocol Server Concepts](https://modelcontextprotocol.io/docs/learn/server-concepts#core-server-features)

| Concept | Description |
|----------|--------------|
| **Resources** | Define which data sources (like SQLite databases or tables) the server can access. |
| **Prompts** | Specify predefined interactions or questions that guide how data should be retrieved or processed. |
| **Tools** | Define operations (e.g., execute query, list tables) that the client can invoke. |

#### **For the MCP Client:**
From the [Model Context Protocol Client Concepts](https://modelcontextprotocol.io/docs/learn/client-concepts#core-client-features)

| Concept | Description |
|----------|--------------|
| **Sampling** | Lets the client ask the LLM to generate or refine structured requests, such as building SQL queries. |
| **Elicitation** | Allows the server to request additional details or parameters from the user when needed. |

---

### 📊 Optional: Sequence Diagram

Create a **sequence diagram** to visualize your system flow using  
[draw.io](https://drawio.com).

**Example flow:**
1. User asks: *“Show me all customers from Switzerland.”*  
2. Client (via Sampling) builds a query using LLM.  
3. MCP Server executes SQL via `run_query`.  
4. Server returns structured results.  
5. Client (via Elicitation) may ask for more details if the query is incomplete.

---

### 🎤 Presentation Task

Prepare a **5-minute group presentation** explaining:
- Your MCP Server architecture  
- How it handles SQLite queries  
- Key design decisions and trade-offs  
- The interaction between Server (Resources, Prompts, Tools) and Client (Sampling, Elicitation)  
- (Optional) Show your sequence diagram or demo

Focus on **clarity, scope, and reasoning** behind your design choices.


## 🧩 Exercise 5: Implement the Designed SQLite-MCP-Server

Based on your design from Exercise 4, now implement the **SQLite MCP Server**.

---

### 🧑‍💻 Development Setup

1. **Install gofastmcp**  
   Follow the installation guide:  
   [gofastmcp Installation](https://gofastmcp.com/getting-started/installation)

2. **Complete the Quickstart**  
   Go through the Quickstart tutorial to ensure your environment works correctly.

3. **Start the MCP Inspector**  
   Run the following command:
   ```bash
   npx @modelcontextprotocol/inspector
   ```
   Configure the connection:
      Transport Type: STDIO
      Command: fastmcp
      Arguments: run my_server.py
      (Adjust the filename if you used a different name for your server.)

4. Add the Configuration to `.vscode/mcp.json`

Example configuration:
```json
        "My MCP Server": {
            "command": "uv",
            "args": [
                "run",
                "--python",
                "/Users/kuhs/Documents/GitHub/mcp-sqlite-server/.venv/bin/python",
                "--with",
                "fastmcp",
                "fastmcp",
                "run",
                "/Users/kuhs/Documents/GitHub/mcp-sqlite-server/my_server.py"
            ]
        }
```

### 🧠 Task

Implement the **MCP-SQLite-Server** using your design from Exercise 4.  

Test your implementation with the **MCP Inspector** and/or **VS Code Copilot** to verify connectivity and functionality.


## 🧩 (Optional) Exercise 6: Customize Instructions for Copilot

Follow GitHub’s guide to add **repository-specific Copilot instructions**:  
[How to Add Repository Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions?tool=visualstudio)

Use these instructions to help Copilot understand:

- Your MCP-SQLite-Server structure  
- How to generate SQL queries  
- How to extend or debug your MCP integration  

