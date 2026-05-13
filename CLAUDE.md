# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TradingAgents-CN is a Chinese-enhanced multi-agent LLM financial trading analysis platform. Based on the upstream [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents) project, it adds Chinese localization, A-share market support, domestic LLM integration, and a modern web interface.

**Architecture**: FastAPI backend + Vue 3 frontend + MongoDB + Redis + Docker deployment

**License**: Hybrid - Apache 2.0 for most code, proprietary for `app/` and `frontend/` directories requiring commercial licensing.

## Development Commands

### Backend Development
```bash
# Start FastAPI backend server
python -m app

# Start with specific host/port
uvicorn app.main:create_app --host 0.0.0.0 --port 8000 --reload

# Run backend worker for background tasks
python -m app.worker
```

### Frontend Development
```bash
cd frontend
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build
npm run type-check   # TypeScript type checking
```

### Core Trading Analysis
```bash
# Run trading analysis directly (bypasses web interface)
python main.py                    # Run example in root main.py
python -m tradingagents           # Use the core trading system

# CLI interface
python -m cli                     # Interactive CLI for stock analysis
python -m cli analyze 000001      # Analyze specific stock
python -m cli sync-stock 000001   # Sync stock data
```

### Testing
```bash
# Run all tests
python -m pytest tests/

# Run specific test file
python tests/test_akshare_api.py

# Run with markers
python -m pytest tests/ -m integration
python -m pytest tests/ -k "test_akshare"

# Quick test (skips integration/slow tests)
python -m pytest tests/ -m "not integration"
```

### Docker Deployment
```bash
# Full stack deployment
docker-compose up -d

# Build and start specific services
docker-compose up backend frontend

# Include management interfaces (Mongo Express, Redis Commander)
docker-compose --profile management up -d

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend
```

## Architecture Overview

### Multi-Agent System (`tradingagents/`)

The core trading analysis system uses a multi-agent architecture:

- **Agents** (`tradingagents/agents/`):
  - `analysts/`: fundamentals_analyst, market_analyst, news_analyst, social_media_analyst, china_market_analyst
  - `researchers/`: Deep research and shallow research agents
  - `managers/`: Risk and portfolio managers
  - `trader/`: Final trading decision maker

- **Graph System** (`tradingagents/graph/`):
  - `trading_graph.py`: Main orchestration graph that coordinates all agents
  - `setup.py`: Graph initialization and agent wiring
  - `propagation.py`: Forward propagation of analysis
  - `reflection.py`: Learning from past decisions

- **LLM Integration** (`tradingagents/llm_clients/`):
  - Abstracted LLM client interface supporting multiple providers
  - `factory.py`: Provider factory pattern
  - `model_catalog.py`: Shared model catalog across the system
  - `provider_keys.py`: Canonical provider key normalization

- **Data Sources** (`tradingagents/dataflows/`):
  - `data_source_manager.py`: Unified data source manager with fallback chains
  - `interface.py`: Standardized data interface
  - `optimized_china_data.py`: Optimized Chinese market data access
  - Providers: `providers/akshare.py`, `providers/tushare.py`, `providers/baostock.py`

### Backend API (`app/`)

FastAPI-based REST API with real-time notifications:

- **Routers** (`app/routers/`): Analysis, config, stocks, screening, sync, reports, etc.
- **Services** (`app/services/`): Business logic layer
  - `analysis_service.py`: Core analysis orchestration
  - `simple_analysis_service.py`: Simplified analysis flow
  - `config_service.py`: LLM and system configuration management
  - `stock_data_service.py`: Stock data operations
  - `database_service.py`: MongoDB operations
- **Models** (`app/models/`): Pydantic data models
- **Schemas** (`app/schemas/`): API request/response schemas

### Frontend (`frontend/`)

Vue 3 + TypeScript + Element Plus SPA:
- `src/`: Main source code
- `src/api/`: API client modules
- `src/components/`: Vue components
- `src/stores/`: Pinia state management
- `src/router/`: Vue Router configuration

### Configuration System

Configuration is centralized in `tradingagents/config/`:
- `config_manager.py`: Main configuration management
- `database_manager.py`: Database configuration and connection
- `providers_config.py`: LLM provider configuration
- `runtime_settings.py`: Runtime settings management

Environment variables are loaded from `.env` file (see `.env.example`).

## Key Integration Points

### Adding a New LLM Provider

1. Create adapter in `tradingagents/llm_adapters/` or `tradingagents/llm_clients/`
2. Add provider to `tradingagents/llm_clients/factory.py`
3. Update `tradingagents/llm_clients/model_catalog.py` with available models
4. Add environment variables to `.env.example`
5. Update `tradingagents/config/providers_config.py` for configuration management

### Adding a New Data Source

1. Create provider in `tradingagents/dataflows/providers/`
2. Register in `tradingagents/dataflows/data_source_manager.py`
3. Add to fallback chains in `optimized_china_data.py` if for Chinese markets
4. Update `app/services/stock_data_service.py` for API exposure

### Adding a New Analyst Agent

1. Create agent in `tradingagents/agents/analysts/`
2. Follow existing analyst pattern (state management, tools, prompts)
3. Register in `tradingagents/graph/setup.py`
4. Add to trading graph in `tradingagents/graph/trading_graph.py`

## Data Source Priority Chains

**Chinese Markets (A-shares)**:
1. AkShare (free, no API key needed)
2. Tushare (requires token, more reliable)
3. BaoStock (free, limited)

**US Markets**:
1. FinnHub (primary)
2. yfinance (fallback)

**Hong Kong Markets**:
1. AkShare (eastmoney data)
2. FinnHub
3. yfinance

## Important Patterns

### Provider Key Normalization
Always use canonical provider keys from `tradingagents/llm_clients/provider_keys.py`:
- `openai`, `anthropic`, `google`, `deepseek`, `dashscope`, etc.
- This enables consistent configuration across the system

### Database Version Isolation
The system uses version-isolated database naming:
- Format: `{database_name}_v{version}`
- Example: `tradingagentscn_v101` for v1.0.1
- Migration scripts in `scripts/mongo/` handle version upgrades

### Analysis State Management
Analysis jobs track progress through Redis:
- Progress updates via SSE (`app/routers/sse.py`)
- WebSocket fallback for real-time updates
- State persisted in MongoDB for resume capability

### Configuration Persistence
User configurations stored in MongoDB:
- LLM provider settings
- Model selection and parameters
- Data source preferences
- Retrieved via `app/services/config_service.py`

## Testing Strategy

- **Unit tests**: Test individual components in isolation
- **Integration tests**: Test API endpoints and service interactions (marked with `@pytest.mark.integration`)
- **Data source tests**: Verify data provider connectivity
- **Model tests**: Test LLM provider integrations

## Common Issues

### Import Errors
The project uses both `app/` and `tradingagents/` as top-level packages. Ensure:
- Project root is in `PYTHONPATH`
- Use `python -m package` syntax for running modules
- Docker builds properly set `PYTHONPATH`

### Database Connection Issues
- Verify MongoDB is running: `docker-compose logs -f mongodb`
- Check connection string in `.env`
- Ensure proper authentication credentials

### Frontend Build Issues
- Delete `node_modules/` and `package-lock.json` then reinstall
- Ensure Node.js >= 18.0.0
- Check Vite proxy configuration in `frontend/vite.config.ts`

## Upstream Synchronization

The project selectively integrates changes from the upstream TauricResearch/TradingAgents:
- See `docs/maintenance/upstream-sync.md` for synchronization strategy
- `docs/maintenance/manual-upstream-absorption-checklist.md` tracks absorbed features
- Key absorbed features include: `llm_clients` abstraction, shared model catalog, provider canonical keys, graph initialization paths
