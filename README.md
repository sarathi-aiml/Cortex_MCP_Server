
# MCP Server for Cortex Agents Setup

Run a local MCP server that exposes Snowflake Cortex Agents tools, then wire it into Claude Desktop.

## What is MCP?

MCP (Model Context Protocol) is a lightweight protocol that lets external tools and services connect to AI assistants like Claude.  
It works as a bridge between the AI and your own systems.


## MCP vs Agents

- **MCP**: Exposes tools. It does not decide how to use them.  
- **Agents**: Run reasoning loops, choose tools, and handle workflows. MCP gives agents the tools they need.  

---

## 1) Prerequisites

- macOS with Terminal and zsh.
- Snowflake account with access to Cortex Search and Cortex Analyst.
- Programmatic Access Token (PAT) in Snowflake.
- Claude Desktop installed.

> Follow the Quickstart for Cortex Agents and create a Cortex Search service and a Cortex Analyst in your Snowflake account:  
> https://quickstarts.snowflake.com/guide/getting_started_with_cortex_agents/

---

## 2) Install `uv`

`uv` runs Python projects fast without manual virtualenv setup.

Pick one method.

**Option A, Homebrew**

```bash
brew install uv
```

**Option B, official script**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Add `uv` to PATH

If `uv --version` fails, add the install path.

- For Homebrew on Apple Silicon:

```bash
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

- For installer script (default path):

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Verify

```bash
uv --version
```

You should see a version number.

---

## 3) Prepare the project folder

- Download the code from the link below and place it in the project folder:  
  https://github.com/sarathi-aiml/Cortex_MCP_Server

- Place your MCP server code in a folder. Example:  
  `/Users/<you>/Downloads/MCP/mcp-server-cortex`

- Create PAT in Snowflake and add it to the `.env.dev` file.

- Update `.env.dev` in your local machine with the following values:

```dotenv
SNOWFLAKE_ACCOUNT_URL=https://<account>.snowflakecomputing.com
SNOWFLAKE_PAT=<your_pat_here>
CORTEX_SEARCH_SERVICE=<your_search_service_name>
SEMANTIC_MODEL_FILE=<path_or_identifier>
```

---

## 4) Install Python requirements

Create a file named `requirements.txt` in your project folder with:

```text
httpx
python-dotenv
mcp
```

Install them with:

```bash
uv pip install -r requirements.txt
```

---

## 5) Run the MCP server

Run the server using:

```bash
uv run mcp_cortex_server.py
```

or

```bash
python mcp_cortex_server.py
```

You will not see output in the terminal, but the server is running.

---

## 6) Wire it into Claude Desktop

1. Open Claude Desktop.  
2. Go to **Settings** → **Developer**.  
3. Click **Edit config** or **Add config**. Finder opens `claude_desktop_config.json`. Open this file in any code editor.  
4. Insert a block like this. Adjust paths to your user folder.

```json
{
  "mcpServers": {
    "cortex-agent": {
      "command": "/Users/<you>/.local/bin/uv",
      "args": [
        "--directory",
        "/Users/<you>/Downloads/MCP/mcp-server-cortex",
        "run",
        "mcp_cortex_server.py"
      ]
    }
  }
}
```

Make sure `"command"` points to the correct `uv` path and `"args"` has the right path to `mcp_cortex_server.py`.

5. Save the file.  
6. Quit Claude Desktop completely (not just close).  
7. Reopen Claude Desktop.  
8. In a chat, click the **Tools** icon next to the **+** button. You should see **cortex-agent**.

---

## 7) Test the server from Claude Desktop

Sample prompts:

- "What is the total deal value for each product line? Also find top performing product and list the reason. Use Cortex agents."  
- "What is the total deal value for each product line? Use Cortex agents."  

---

## 8) MCP Server Flow

```text
Claude Desktop ---> MCP Config ---> Cortex MCP Server ---> Snowflake API
                                           |  
                                           +--> Cortex Analyst (SQL generation)
                                           +--> Cortex Search (semantic search)
                                           +--> SQL Exec (runs in Snowflake)
```

---

## 9) Troubleshooting

- **`uv` not found** → Fix PATH as in section 2, restart terminal, run `uv --version`.  
- **No `cortex-agent` in Tools** → Recheck `claude_desktop_config.json` syntax, restart Claude Desktop.  
- **401/403 from Snowflake** → Verify PAT is valid, check role/warehouse/network policy.  
- **Cortex Search errors** → Confirm the service name matches `.env.dev`, check index status.  
- **Cortex Analyst errors** → Verify semantic model file path or identifier.  
- **SSE stalls** → Try different network if proxies block event streams.  

---

## 10) Security tips

- Store secrets in `.env.dev` with restricted permissions.  
- Rotate PAT regularly.  
- Do not print secrets in logs.  

---

## 11) Summary checklist

- [ ] `uv` installed and in PATH.  
- [ ] `.env.dev` contains `SNOWFLAKE_ACCOUNT_URL`, `SNOWFLAKE_PAT`, `CORTEX_SEARCH_SERVICE`, `SEMANTIC_MODEL_FILE`.  
- [ ] `requirements.txt` installed.  
- [ ] Snowflake Cortex Search and Analyst are available.  
- [ ] Claude Desktop config points to `uv` and the project folder.  
- [ ] Restarted Claude. **cortex-agent** visible and working.  


### Best Practices for MCP Server Setup

- Use **TLS and authentication** to secure communication.  
- Keep services **modular, stateless, and independently deployable**.  
- Implement **monitoring, structured logging, and trace IDs** for visibility.  
- Design for **scalability and graceful degradation** under load.  
- Add **health checks and auto-restart policies** for resilience.  
- Regularly **rotate PAT tokens** and review Snowflake roles.  
- Test with **mock queries and sandbox data** before production.  
- Apply **resource limits** (CPU, memory) to prevent overload.  
- Document **tool contracts** (inputs, outputs, failure modes) for easier extension.  
- Run MCP servers in **isolated environments** (containers, VMs, or sandboxes). 
