# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Verba is a full-stack Retrieval-Augmented Generation (RAG) application with:
- **Backend**: Python FastAPI server with modular RAG components
- **Frontend**: Next.js 14 with React, TypeScript, TailwindCSS, and DaisyUI
- **Vector Database**: Weaviate (embedded, Docker, or cloud deployment)

## Development Commands

### Backend Development

```bash
# Install development dependencies
pip install -e .

# Run Verba server (default: http://localhost:8000)
verba start
verba start --port 8000 --host 0.0.0.0

# Run tests (minimal coverage currently)
pytest goldenverba/tests

# Format Python code (required before PR)
black goldenverba/
```

### Frontend Development

```bash
cd frontend

# Install dependencies (requires Node.js >=21.3.0)
npm install

# Development server (http://localhost:3000)
npm run dev

# Build static files for FastAPI serving
npm run build

# Production server (http://0.0.0.0:8080)
npm start

# Lint frontend code
npm run lint
```

### Docker Development

```bash
# Build and run with docker-compose
docker-compose up -d

# Run standalone
docker build -t verba .
docker run -p 8000:8000 verba
```

## Architecture Overview

### Backend Structure (`goldenverba/`)

**Core Components:**
- `VerbaManager`: Central orchestrator managing all RAG components
- Component Managers: Reader, Chunker, Embedder, Retriever, Generator, Weaviate
- FastAPI server with WebSocket support for real-time communication
- Batch processing for large file uploads

**Component Interfaces:**
- **Readers**: Import documents from various sources (PDF, GitHub, URLs, etc.)
- **Chunkers**: Split text using different strategies (Token, Sentence, Semantic, Code, etc.)
- **Embedders**: Generate vectors (OpenAI, Ollama, Cohere, HuggingFace, etc.)
- **Generators**: LLM integrations (OpenAI, Anthropic, Ollama, Groq, etc.)
- **Retrievers**: Search strategies including hybrid search

### Frontend Structure (`frontend/`)

- Next.js App Router architecture
- Main views: Chat Interface, Document Explorer, Ingestion, Settings
- 3D vector visualizations using Three.js
- WebSocket connection for real-time updates
- Static build outputs to `goldenverba/server/frontend/out/`

## Key Development Patterns

### Adding New Components

1. Create component class inheriting from appropriate interface
2. Register in corresponding manager class
3. Add UI configuration in frontend settings

### API Endpoints

- REST API: `/api/*` routes in `goldenverba/server/api.py`
- WebSocket: `/ws` for real-time chat and status updates
- Static files served from `/` route

### Environment Variables

Key variables for development:
- `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc. for LLM providers
- `WEAVIATE_URL`, `WEAVIATE_API_KEY` for vector database
- `UNSTRUCTURED_API_KEY`, `ASSEMBLYAI_API_KEY` for data processing
- See README.md for complete list

## Testing Strategy

- Backend: pytest with tests in `goldenverba/tests/`
- Frontend: No formal test suite currently
- Manual testing recommended for:
  - File ingestion workflows
  - RAG pipeline configurations
  - Multi-client WebSocket connections

## Common Development Tasks

### Running a Single Test
```bash
pytest goldenverba/tests/document/test_document.py::test_specific_function -v
```

### Debugging WebSocket Issues
- Check client connection in `ClientManager`
- Monitor WebSocket logs in browser console
- Verify FastAPI WebSocket endpoint configuration

### Adding a New LLM Provider
1. Create generator class in `goldenverba/components/generation/`
2. Implement `generate()` and `prepare_messages()` methods
3. Register in `GenerationManager`
4. Add frontend configuration in Settings component

### Modifying Chunking Strategy
- Context position controlled by `prepare_messages()` in generator classes
- Chunk metadata preserved through ingestion pipeline
- Window size configurable per retriever

## Important Considerations

- Frontend changes require rebuild: `npm run build` in frontend directory
- All Python code must be formatted with Black before PR
- WebSocket connections handle multiple clients simultaneously
- Batch processing for files >5MB split into chunks
- Embedding dimension must match across components and Weaviate schema

## PR Workflow

1. Fork and create branch from `main`
2. Format Python code with Black
3. Run existing tests: `pytest goldenverba/tests`
4. Build frontend if modified: `cd frontend && npm run build`
5. Submit PR with clear description and linked issue