# Multi-Agent Customer Service Platform

A robust multi-agent customer service platform leveraging Google's Agent Development Kit (ADK) and the Model Context Protocol (MCP). The system showcases sophisticated agent-to-agent (A2A) communication, tool integration capabilities, and smart request routing for customer support operations.

## ✨ Key Features

- **Multi-Agent Design**: Coordinated operation between Router, Customer Data Agent, and Support Agent
- **MCP Protocol**: Standardized access to customer database tools
- **Inter-Agent Communication**: Fluid coordination among specialized agents
- **Live Tool Execution**: Direct database interactions via MCP server
- **Full Customer Management**: Complete CRUD functionality, ticket generation, and activity tracking
- **Production-Grade**: Includes error handling, logging, and monitoring capabilities

## 🏗️ System Architecture

```
┌─────────────────┐
│  Router Agent   │  ← Manages all incoming requests
└────────┬────────┘
         │
    ┌────┴────┐
    ↓         ↓
┌─────────┐ ┌─────────┐
│Customer │ │Support  │  ← Domain specialists
│Data     │ │Agent    │
│Agent    │ │         │
└────┬────┘ └─────────┘
     │
     ↓
┌─────────────────┐
│  MCP Server     │  ← Tool interface
│  (Flask + SSE)  │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  SQLite DB      │  ← Persistent storage
│  (Customers +   │
│   Tickets)      │
└─────────────────┘
```

## 🚀 Getting Started

### Requirements

- Python 3.10 or higher
- Google API Key (Gemini models)
- ngrok authtoken (optional, for remote access)

### Setup Instructions

**1. Clone the repository**

```bash
git clone <repository-url>
cd customer-service-mcp
```

**2. Install required packages**

```bash
pip install google-adk google-generativeai flask flask-cors pyngrok termcolor
```

**3. Configure environment**

```python
# Add your API credentials
os.environ['GOOGLE_API_KEY'] = 'your-google-api-key'
os.environ['NGROK_AUTHTOKEN'] = 'your-ngrok-token'  # Optional
```

### Launch Instructions

**Step 1: Initialize the MCP Server**

Execute the `mcp_server.ipynb` notebook:

```python
# This process will:
# 1. Set up the SQLite database
# 2. Launch the Flask MCP server
# 3. Create ngrok tunnel (optional)
# 4. Show the server URL
```

Expected output:

```
✅ MCP Server is running!
🔗 Local URL: http://127.0.0.1:5000
🌍 Public URL: https://your-app.ngrok-free.dev
🔍 MCP Endpoint: https://your-app.ngrok-free.dev/mcp
```

**Step 2: Launch the Agent System**

Execute the `a2a_team.ipynb` notebook:

```python
# Set the MCP server URL
MCP_SERVER_URL = "http://127.0.0.1:5000/mcp"

# Agents initialize automatically
# Ready to process queries!
await ask_agent_team("Get customer information for ID 5")
```

## 🤖 Agent Functions

### Router Agent (Coordinator)

The Router Agent serves as the orchestration layer, managing request flow and response synthesis.

**Primary Responsibilities:**
- Understanding user intent
- Directing requests to specialist agents
- Consolidating final responses
- Managing clarification requests

**Core Skills:**
- Customer query orchestration
- Multi-phase workflow coordination
- Intent classification

### Customer Data Agent (Data Specialist)

Dedicated agent with direct MCP tool access for all database operations.

**Primary Responsibilities:**
- Managing customer data operations
- Handling data modifications with validation

**Core Skills:**
- `retrieve_customer`: Query customer records using ID or filters
- `update_customer`: Modify customer data with validation checks
- `validate_customer_input`: Pre-update field validation

**Available MCP Tools:**
- `get_customer(customer_id)`: Fetch individual customer records
- `list_customers(status?)`: Retrieve all or filtered customer lists
- `update_customer(customer_id, name?, email?, phone?)`: Modify customer data
- `create_ticket(customer_id, issue, priority)`: Generate support tickets
- `get_customer_history(customer_id)`: Access ticket history

### Support Agent (Service Specialist)

Handles general support inquiries and provides customer assistance.

**Primary Responsibilities:**
- Processing customer support requests
- Offering troubleshooting assistance
- Escalating complex cases

**Core Skills:**
- `handle_support_query`: Process support inquiries
- `request_customer_context`: Retrieve data from Customer Data Agent
- `escalate_issue`: Forward to specialized teams
- `provide_recommendation`: Offer solution suggestions

## 📝 Usage Examples

### Example 1: Fetching Data

```python
await ask_agent_team("Get customer information for ID 5")
```

**Output:**

```
Here is the information for customer ID 5:

**Name:** Charlie Brown
**Email:** charlie.brown@email.com
**Phone:** +1-555-0105
**Status:** active
**Created At:** 2025-11-27 23:12:44
**Updated At:** 2025-11-27 23:12:44
```

### Example 2: Support Request

```python
await ask_agent_team("I'm customer 12345 and need help upgrading my account")
```

**Processing Flow:**
1. Router → Customer Data Agent (customer verification)
2. Customer Data Agent → Support Agent (process upgrade)
3. Support Agent → Router → User (deliver guidance)

### Example 3: Multi-Step Workflow

```python
await ask_agent_team("I've been charged twice, please refund immediately!")
```

**Response:**

```json
{
  "response": "I understand your concern about being double-charged and the urgency of a refund. 
              I'll need to access your account details to investigate this issue thoroughly 
              and process your request. Please bear with me while I look into this for you."
}
```

## 🔍 MCP Protocol Specifications

### Protocol Information

- **MCP Version:** 2024-11-05
- **Transport Method:** Server-Sent Events (SSE) over HTTP
- **Message Format:** JSON-RPC 2.0

### MCP Tool Catalog

| Tool | Purpose | Parameters |
|------|---------|------------|
| `get_customer` | Fetch customer by ID | `customer_id: int` |
| `list_customers` | Retrieve all/filtered customers | `status?: "active" \| "disabled"` |
| `update_customer` | Modify customer information | `customer_id: int, name?: str, email?: str, phone?: str` |
| `create_ticket` | Generate support ticket | `customer_id: int, issue: str, priority: str` |
| `get_customer_history` | Retrieve ticket history | `customer_id: int` |

### MCP Server API

- **MCP Endpoint:** `POST /mcp` - Core protocol endpoint
- **Health Check:** `GET /health` - Server status verification
- **Protocol:** JSON-RPC 2.0 over SSE

## 📊 Event Trace Sample

```
📡 FULL AGENT EVENT TRACE
──────────────────────────────────────────────────────────

[Event 1]
  author       : router_agent
  model_version: gemini-2.5-flash
  🛠 Function Call (part 1)
     name : transfer_to_agent
     args : {"agent_name": "customer_data_agent"}

[Event 2]
  author       : router_agent
  🔀 transfer_to_agent
     router_agent → customer_data_agent

[Event 3]
  author       : customer_data_agent
  🛠 Function Call (part 1)
     name : get_customer
     args : {"customer_id": 5}

[Event 4]
  📦 Function Response (part 1)
     from : get_customer
     response : {"success": true, "customer": {...}}
```

## 🛠️ Technical Stack

### Agent Cards (A2A Protocol)

Each agent exposes an Agent Card containing:
- **Capabilities:** Streaming support, input/output configurations
- **Skills:** Comprehensive skill descriptions with usage examples
- **Transport:** JSON-RPC as preferred protocol
- **Version:** Semantic versioning

### Technology Stack

- **Google ADK:** Agent coordination and management
- **Gemini 2.5 Flash:** LLM for agent intelligence
- **Flask:** MCP server framework
- **SQLite:** Customer data persistence
- **ngrok:** Public tunneling (optional)
- **SSE:** Real-time event streaming

## 📈 Project Insights & Obstacles

### What I Learned

This project gave me practical experience with how Agent-to-Agent communication enables scalable coordination in multi-agent customer service architectures. By implementing distinct roles for routing, data retrieval, and support handling, I gained firsthand insight into how structured message passing, tool-grounded actions, and effective state management work together to create dependable automated workflows. Working with A2A message architecture deepened my appreciation for how distributed agents can collaborate, negotiate, and share context to tackle multi-step tasks in a controlled and transparent way. I also learned how the Model Context Protocol provides a standardized approach to tool integration, allowing agents to coordinate effectively through shared data sources and enabling seamless interaction between agents and external systems while maintaining consistency and reliability. Understanding the importance of clear role separation became evident throughout development, as specialized agents with well-defined responsibilities significantly improve system reliability and make the architecture more maintainable and scalable.

### Challenges Resolved

One of the primary challenges was debugging errors that occurred across multiple system layers. When issues arose, pinpointing whether they originated from the router's intent logic, an MCP tool malfunction, an agent-to-agent transfer problem, or a state transition within the graph proved difficult. This complexity pushed me to develop more comprehensive event logging, implement detailed message tracing, and create modular testing approaches, which ultimately strengthened the overall system stability and made troubleshooting far more manageable. Ensuring consistent state across agent transfers required careful design of how context gets passed between agents, involving clear protocols for state handoffs and maintaining data integrity throughout the workflow. Mapping responses from MCP tools to appropriate agent actions demanded robust error handling and validation logic, as building reliable bridges between tool outputs and agent behaviors was essential for smooth operation. The experience highlighted the critical importance of observability, well-defined interfaces, and thoughtful task decomposition when designing production AI systems, particularly when integrating external data sources and coordinating multiple autonomous agents in real-world applications.

## 🔐 Implementation Guidelines

- **Error Handling:** All MCP operations include try-catch protection
- **Validation:** Input validation performed before database operations
- **Logging:** Detailed event logging for troubleshooting
- **State Management:** Explicit context passing between agents
- **Tool Design:** Clear tool schemas with comprehensive descriptions
