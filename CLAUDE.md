# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Retrieval-Augmented Generation (RAG) system for querying course materials using semantic search and AI-powered responses. The system uses ChromaDB for vector storage, Anthropic's Claude API with tool calling, and FastAPI for the backend API.

## Key Commands

### Running the Application
```bash
# Quick start (from root directory)
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# Run commands with uv
uv run <command>
```

### Access Points
- Web Interface: http://localhost:8000
- API Documentation: http://localhost:8000/docs

## Architecture

### Core Components

The application follows a modular RAG architecture with these key components:

**RAGSystem** (`backend/rag_system.py`) - Main orchestrator that coordinates all components:
- Initializes and connects document processor, vector store, AI generator, session manager, and tool manager
- Handles document ingestion from files or folders
- Processes queries through tool-enabled AI generation
- Manages conversation sessions

**VectorStore** (`backend/vector_store.py`) - ChromaDB wrapper with dual collection strategy:
- `course_catalog` collection: Stores course metadata (titles, instructors, lesson lists) for semantic course name matching
- `course_content` collection: Stores chunked course content with course/lesson metadata for content search
- Unified search interface that resolves course names semantically and filters by course/lesson

**AIGenerator** (`backend/ai_generator.py`) - Claude API integration with tool calling:
- Uses Anthropic's tool calling API to enable search during response generation
- Handles conversation history context
- Implements two-phase response: tool execution followed by final answer generation

**ToolManager & SearchTool** (`backend/search_tools.py`) - Tool abstraction layer:
- `CourseSearchTool`: Implements search as a Claude tool with course name and lesson filtering
- `ToolManager`: Registers tools and routes execution, tracks sources from searches

**DocumentProcessor** (`backend/document_processor.py`) - Parses course documents:
- Expected format: Course metadata (title, link, instructor) followed by lesson markers
- Chunks text using sentence-based splitting with overlap
- Creates `Course`, `Lesson`, and `CourseChunk` objects

**SessionManager** (`backend/session_manager.py`) - Conversation state:
- Maintains per-session message history with configurable max history
- Formats history for context in AI prompts

### Data Flow

1. **Document Ingestion**: Documents in `docs/` are loaded on startup
   - `DocumentProcessor` parses course structure
   - Course metadata added to `course_catalog` collection
   - Content chunks added to `course_content` collection

2. **Query Processing**:
   - User query → FastAPI endpoint → `RAGSystem.query()`
   - `AIGenerator` receives query + tool definitions + conversation history
   - Claude decides whether to call `search_course_content` tool
   - If tool called: `VectorStore.search()` resolves course name and searches content
   - `AIGenerator` receives search results and generates final response
   - Session history updated

### Configuration

All configuration is centralized in `backend/config.py`:
- `ANTHROPIC_API_KEY`: Required environment variable
- `ANTHROPIC_MODEL`: Claude model (default: claude-sonnet-4-20250514)
- `EMBEDDING_MODEL`: Sentence transformer model for embeddings
- `CHUNK_SIZE`: Text chunk size (default: 800)
- `CHUNK_OVERLAP`: Overlap between chunks (default: 100)
- `MAX_RESULTS`: Search results limit (default: 5)
- `MAX_HISTORY`: Conversation turns to remember (default: 2)
- `CHROMA_PATH`: Vector DB location (default: ./chroma_db)

### Document Format

Course documents in `docs/` should follow this format:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: [lesson title]
Lesson Link: [url]
[lesson content]

Lesson 1: [lesson title]
Lesson Link: [url]
[lesson content]
```

## Environment Setup

Required environment variable in `.env` file:
```
ANTHROPIC_API_KEY=your_key_here
```

Python version: 3.13 or higher (specified in `.python-version`)
