# Gemini Fullstack LangGraph Quickstart

This project is a full-stack application featuring a React-based frontend and a Python backend that uses LangGraph to create a powerful research agent. This document provides a detailed breakdown of the project's architecture, setup, and workflow to help you understand how it all works together.

## Project Overview

The application allows users to interact with a research agent through a chat interface. When a user asks a question, the backend agent performs web research to find the most relevant information, and then synthesizes a comprehensive answer. The frontend provides a real-time view of the agent's progress, from generating search queries to analyzing sources and finalizing the answer.

## How to Run the Application

The project is designed to be easy to run with a single command, which starts both the frontend and backend development servers.

### Prerequisites

- [Node.js and npm](https://nodejs.org/en/download/)
- [Python 3.11+](https://www.python.org/downloads/)
- [Make](https://www.gnu.org/software/make/) (or you can run the commands from the `Makefile` manually)

### Setup

1.  **Clone the repository**
2.  **Install frontend dependencies:**
    ```bash
    cd frontend
    npm install
    ```
3.  **Install backend dependencies:**
    ```bash
    cd backend
    pip install -e .
    ```
4.  **Set up environment variables:**

    Create a `.env` file in the `backend` directory and add your Gemini API key:

    ```
    GEMINI_API_KEY="your_api_key_here"
    ```

### Running the Servers

From the root directory, run the following command to start both the frontend and backend servers:

```bash
make dev
```

This command executes two separate processes:

- **Frontend (`make dev-frontend`)**: Starts the Vite development server for the React application. By default, it runs on `http://localhost:5173`.
- **Backend (`make dev-backend`)**: Starts the LangGraph development server using `langgraph dev`. This server exposes the research agent's API and a WebSocket endpoint for real-time communication. It typically runs on `http://localhost:8000`.

## How It Works

The frontend and backend are tightly integrated to provide a seamless, real-time user experience. Here’s a step-by-step look at how they interact.

### Frontend (React + Vite)

The frontend is built with React and Vite, and it uses the `@langchain/langgraph-sdk` to communicate with the backend.

- **`App.tsx`**: This is the main component of the frontend. It manages the chat state and handles communication with the backend using the `useStream` hook.
- **`useStream` Hook**: This hook is the bridge between the frontend and the backend. It sends the user's messages to the agent and listens for a stream of events in return. The `apiUrl` is configured to point to the backend server.
- **Real-Time Updates**: As the backend agent works, it sends back events that are captured by the `onUpdateEvent` callback in `App.tsx`. These events are then used to update the "Activity Timeline," giving you a live look into the agent's process.
- **Vite Proxy**: The `vite.config.ts` file is configured to proxy requests from `/api` to the backend server at `http://127.0.0.1:8000`. This simplifies API calls from the frontend.

### Backend (Python + FastAPI + LangGraph)

The backend is powered by a LangGraph agent, which is a state machine that defines the research process.

- **`graph.py`**: This file defines the structure of the research agent. It consists of several "nodes," each responsible for a specific task:
  - `generate_query`: Generates search queries based on the user's question.
  - `web_research`: Executes the search queries and gathers information from the web.
  - `reflection`: Analyzes the gathered information and decides if more research is needed.
  - `finalize_answer`: Synthesizes the final answer from the research.
- **`state.py`**: This file defines the `OverallState` of the agent, which is a dictionary that holds all the data for a given research task, including the conversation history, search queries, and research results. This state is passed from node to node as the agent works.

### Frontend-Backend Communication Flow

1.  **User Submits a Message**: You type a message and hit send. The `handleSubmit` function in `App.tsx` is triggered.
2.  **Request Sent to Backend**: The `useStream` hook sends the message and any other configuration (like the desired "effort" level) to the backend.
3.  **Agent Starts Work**: The LangGraph agent on the backend receives the request and begins executing its graph, starting with the `generate_query` node.
4.  **Events Streamed to Frontend**: As the agent moves through each node, it streams back status updates. The frontend listens for these events and updates the UI accordingly.
5.  **Final Answer Delivered**: When the agent reaches the `finalize_answer` node, it sends the complete response back to the frontend, which is then displayed in the chat window.

## Understanding LangGraph Concepts

- **State**: The `OverallState` object is the "memory" of the agent for a single task. It's how information is passed between the different steps (nodes) in the research process.
- **Threads**: A thread represents a single conversation. Every time you start a new chat, you're creating a new thread. This allows the agent to handle multiple conversations at once without getting them mixed up.
- **Messages**: The `messages` list inside the `OverallState` keeps track of the conversation history for the current thread. This is crucial for allowing the agent to understand the context of the conversation.

## Integrating this Project into "Strides"

This section provides a guide for integrating a similar LangGraph-powered agent into the "Strides" project.

### The Prompt to Use in Your "Strides" Project

To begin the integration, navigate to the root directory of your "Strides" project in your terminal and execute the following prompt. This prompt is designed to be a comprehensive, multi-step instruction that will guide the agent (me) to create a detailed integration plan, generate the necessary boilerplate code, and create the initial files for the agent.

```
I want to integrate a LangGraph-powered agent into this "Strides" project, using the "gemini-fullstack-langgraph-quickstart" application as a reference. My goal is to create a conversational interface for adding expenses.

Please perform the following steps:

1.  **Create a New README:** Create a new file named `INTEGRATION_README.md` in the root of this project. This file will serve as a comprehensive guide and contain all the planning and code for this integration.

2.  **Document the Integration Plan:** Inside the new `INTEGRATION_README.md`, add a section titled "Integration Plan". This section should detail the step-by-step plan for integrating the agent. It must cover:
    *   How the backend will be modified to include a LangGraph agent.
    *   How the agent will use existing backend logic from `routes/transactions.py` and `routes/categories.py` as tools.
    *   How the frontend `AIAgent.tsx` component will be connected to the new backend endpoint.
    *   The overall data flow from the user's natural language input to a created transaction in the database.

3.  **Add Boilerplate Code to the README:** Following the plan, add a section titled "Boilerplate Code" to the `INTE_README.md`. For each new file that needs to be created, provide the full file path and a complete boilerplate code block. This should include:
    *   The new tool definitions in `backend/src/tools/transaction_tools.py`.
    *   The agent and graph definition in `backend/src/agent/graph.py`.
    *   The new API endpoint in `backend/src/routes/agent.py`.

4.  **Implement the Changes:** After documenting the plan and boilerplate in the README, proceed to actually write the new files (`backend/src/tools/transaction_tools.py`, `backend/src/agent/graph.py`, `backend/src/routes/agent.py`) to the filesystem with the specified boilerplate code.
```
