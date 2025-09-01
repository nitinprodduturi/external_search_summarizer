# GenAI Search Component - Developer Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Folder Structure](#folder-structure)
4. [Core Components](#core-components)
5. [Data Models](#data-models)
6. [Configuration](#configuration)
7. [API Reference](#api-reference)
8. [Running the Code](#running-the-code)
9. [Development Workflow](#development-workflow)
10. [Testing](#testing)
11. [Troubleshooting](#troubleshooting)
12. [Contributing](#contributing)

## Project Overview

The GenAI Search Component is a modular, AI-powered search system that combines Large Language Models (LLMs) with web search capabilities to provide intelligent content discovery, extraction, and summarization. It's designed with a clean architecture that separates concerns and makes it easy to extend and maintain.

### Key Features
- **Modular Architecture**: Clean separation of concerns with pluggable components
- **Multi-Provider Support**: Google Generative AI and Azure OpenAI
- **Intelligent Search**: LLM-generated search terms for better results
- **Content Extraction**: Robust web scraping with Selenium and BeautifulSoup
- **Relevance Scoring**: AI-powered content relevance assessment
- **Content Summarization**: Creates comprehensive summaries with **source citations**
- **Enhanced Excel Export**: Professional styling with source citations and reference tables
- **Comprehensive Logging**: Structured logging with performance metrics

## Architecture Overview

The system follows a **Component-Based Architecture** with the following layers:

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                     │
│                 (example_modular.py)                       │
│                 (test_enhanced_export.py)                  │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Orchestrator Layer                       │
│              (GenAISearchOrchestrator)                     │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Component Layer                          │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐ │
│  │Search Terms │Web Search   │Content      │Relevance   │ │
│  │Generator    │Engine       │Extractor    │Scorer      │ │
│  └─────────────┴─────────────┴─────────────┴─────────────┘ │
└─────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────┐
│                   Infrastructure Layer                      │
│  ┌─────────────┬─────────────┬─────────────┬─────────────┐ │
│  │LLM Manager  │Data Models  │Logging      │Exporters   │ │
│  │             │             │Config       │            │ │
│  └─────────────┴─────────────┴─────────────┴─────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Design Principles
- **Single Responsibility**: Each component has one clear purpose
- **Dependency Injection**: Components receive dependencies through constructor
- **Interface Segregation**: Clean interfaces between components
- **Open/Closed**: Easy to extend without modifying existing code
- **Error Handling**: Comprehensive error handling at each layer

## Folder Structure

```
external_search/
├── README.md                    # User-facing documentation
├── DEVELOPER_GUIDE.md          # This developer guide
├── requirements.txt            # Python dependencies
├── example_modular.py          # Basic usage example
├── test_enhanced_export.py     # Enhanced features test script
├── .env                        # Environment variables (create this)
├── runs/                       # Export output directory
│   └── *.xlsx                 # Excel export files
└── genai_search/               # Main package directory
    ├── __init__.py            # Package initialization
    ├── core/                  # Core functionality
    │   ├── __init__.py        # Core module exports
    │   ├── orchestrator.py    # Main orchestrator class
    │   ├── llm_manager.py     # LLM provider management
    │   ├── components/        # Functional components
    │   │   ├── __init__.py    # Component exports
    │   │   ├── search_term_generator.py
    │   │   ├── web_search_engine.py
    │   │   ├── content_extractor.py
    │   │   ├── relevance_scorer.py
    │   │   └── content_summarizer.py  # Enhanced with citations
    │   └── utils/             # Shared utilities
    │       ├── __init__.py    # Utility exports
    │       ├── models.py      # Data models and schemas
    │       ├── prompts.py     # LLM prompt templates (enhanced)
    │       └── logging_config.py # Logging configuration
    └── exporters/             # Export functionality
        ├── __init__.py        # Exporter exports
        └── excel_exporter.py  # Enhanced Excel export with styling
```

## Core Components

### 1. GenAISearchOrchestrator (`core/orchestrator.py`)

The main orchestrator that coordinates all components and manages the search workflow.

**Key Responsibilities:**
- Initialize and manage all components
- Coordinate the search workflow
- Handle error handling and logging
- Manage search sessions
- Provide the main API interface

**Main Methods:**
- `process_query()`: Main entry point for search requests
- `close()`: Cleanup resources (closes Selenium WebDriver)

**Usage:**
```python
from genai_search import create_genai_component

component = create_genai_component(
    llm_provider="google",
    search_engine="google",
    max_search_results=5
)

results = component.process_query("Your search query")
component.close()  # Important: Always close to cleanup resources
```

### 2. LLMManager (`core/llm_manager.py`)

Manages different LLM providers and provides a unified interface for LLM operations.

**Supported Providers:**
- Google Generative AI (Gemini)
- Azure OpenAI

**Key Features:**
- Automatic provider detection
- Environment variable configuration
- Retry logic with exponential backoff
- Chain creation and management

**Configuration:**
```bash
# For Google AI
export GOOGLE_API_KEY="your_key_here"

# For Azure OpenAI
export AZURE_OPENAI_API_KEY="your_key_here"
export AZURE_OPENAI_ENDPOINT="your_endpoint_here"
export AZURE_OPENAI_DEPLOYMENT_NAME="your_deployment_here"
```

### 3. Search Components

#### SearchTermGenerator (`components/search_term_generator.py`)
- Generates optimized search terms from user queries
- Uses LLM to understand query intent
- Supports different query types (technical, general, etc.)

#### WebSearchEngine (`components/web_search_engine.py`)
- Performs web searches using SerpAPI
- Supports multiple search engines (Google, Bing, DuckDuckGo)
- Handles rate limiting and pagination

#### ContentExtractor (`components/content_extractor.py`)
- Extracts content from web pages
- Uses Selenium for JavaScript-heavy pages
- Falls back to BeautifulSoup for simple pages
- **Important**: Manages Selenium WebDriver lifecycle

#### RelevanceScorer (`components/relevance_scorer.py`)
- Evaluates content relevance using LLM
- Provides confidence scores and reasoning
- Supports different scoring strategies

#### ContentSummarizer (`components/content_summarizer.py`) - **ENHANCED**
- Creates comprehensive summaries with **source citations**
- Uses LLM for intelligent summarization with citation prompts
- Supports both cited and non-cited summarization modes
- Automatically adds source reference tables
- **New**: Inline citations using [Source X] format

### 4. Data Models (`core/utils/models.py`)

Comprehensive data structures for type safety and consistency.

**Key Models:**
- `SearchResult`: Web search results
- `ExtractedContent`: Extracted web content
- `SearchTerms`: Generated search terms
- `ProcessingStats`: Performance metrics
- `ComponentConfig`: Component configuration
- `SearchResponse`: Complete search response

**Example:**
```python
from genai_search.core.utils.models import SearchResult, ExtractedContent

# Create a search result
result = SearchResult(
    title="Example Article",
    url="https://example.com",
    snippet="This is a sample snippet...",
    relevance_score=0.85
)

# Create extracted content
content = ExtractedContent(
    url="https://example.com",
    title="Example Article",
    content="Full article content...",
    relevance_score=0.85,
    word_count=500
)
```

### 5. Utilities

#### Logging (`core/utils/logging_config.py`)
- Structured logging with performance metrics
- Execution time tracking
- Component-specific logging
- Configurable log levels

#### Prompts (`core/utils/prompts.py`) - **ENHANCED**
- LLM prompt templates
- **New**: Citation-specific prompts for source attribution
- Optimized for different tasks
- Consistent prompt structure

### 6. Exporters - **ENHANCED**

#### ExcelExporter (`exporters/excel_exporter.py`)
- **Professional styling** with color-coded headers and borders
- **Source citations** in summaries with [Source X] format
- **Reference tables** with complete source information
- **Enhanced formatting** for better readability
- **Citation highlighting** and visual indicators
- **Auto-formatting** with optimal column widths

## Configuration

### Environment Variables

Create a `.env` file in your project root:

```env
# LLM Provider Configuration
GOOGLE_API_KEY=your_google_api_key_here
AZURE_OPENAI_API_KEY=your_azure_openai_api_key_here
AZURE_OPENAI_ENDPOINT=your_azure_openai_endpoint_here
AZURE_OPENAI_DEPLOYMENT_NAME=your_deployment_name_here

# Web Search Configuration
SERPAPI_API_KEY=your_serpapi_key_here

# Optional: Custom Configuration
MAX_SEARCH_RESULTS=10
RELEVANCE_THRESHOLD=0.7
HEADLESS_BROWSER=true
```

### Component Configuration

```python
from genai_search.core.utils.models import ComponentConfig

config = ComponentConfig(
    max_search_results=15,
    max_content_length=8000,
    relevance_threshold=0.6,
    headless_browser=True,
    search_delay=1.5,
    page_load_timeout=15,
    llm_temperature=0.2,
    llm_max_tokens=4096
)

component = create_genai_component(
    llm_provider="google",
    config=config
)
```

## API Reference

### Main Entry Point

```python
from genai_search import create_genai_component

# Create component
component = create_genai_component(
    llm_provider="google",      # "google" or "azure"
    search_engine="google",     # "google", "bing", "duckduckgo"
    **config_kwargs
)

# Process query (now includes citations by default)
results = component.process_query(
    user_query="Your search query",
    query_type="technical",     # "general", "technical", "news", "academic"
    max_results=10,
    relevance_threshold=0.7,
    include_news=False,
    include_academic=False
)

# Cleanup
component.close()
```

### Response Structure

```python
# results is a SearchResponse object
print(results.user_query)           # Original query
print(results.search_terms)         # Generated search terms
print(results.search_results)       # Web search results
print(results.extracted_contents)   # Extracted content
print(results.summary)              # Final summary with citations
print(results.statistics)           # Processing statistics
print(results.processing_time)      # Total processing time
```

### Export Results with Enhanced Styling

```python
from genai_search.exporters import ExcelExporter

exporter = ExcelExporter(output_dir="runs")
xlsx_path = exporter.export(
    user_query=query,
    config=config_dict,
    search_terms=search_terms_dict,
    search_results=results.search_results,
    extracted_contents=results.extracted_contents,
    summary=results.summary,  # Now includes source citations
    statistics=results.statistics.__dict__
)
```

## Running the Code

### 1. **Quick Start (Recommended)**

```bash
# Clone or navigate to the project directory
cd external_search

# Create virtual environment
python -m venv .venv

# Activate virtual environment
# On macOS/Linux:
source .venv/bin/activate
# On Windows:
# .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env  # If available, or create manually
# Edit .env with your API keys

# Run the enhanced test script
python test_enhanced_export.py
```

### 2. **Basic Example**

```bash
# Run the basic example
python example_modular.py
```

### 3. **Interactive Python Session**

```bash
# Start Python interpreter
python

# Import and use
>>> from genai_search import create_genai_component
>>> component = create_genai_component()
>>> results = component.process_query("test query")
>>> print(results.summary)
>>> component.close()
```

### 4. **Required API Keys**

You need the following API keys to run the system:

#### Google Generative AI
1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Create a new API key
3. Add to `.env`: `GOOGLE_API_KEY=your_key_here`

#### SerpAPI (for web search)
1. Go to [SerpAPI](https://serpapi.com/)
2. Sign up and get an API key
3. Add to `.env`: `SERPAPI_API_KEY=your_key_here`

#### Azure OpenAI (optional alternative)
1. Go to [Azure OpenAI](https://azure.microsoft.com/en-us/products/cognitive-services/openai-service)
2. Create a resource and get credentials
3. Add to `.env`:
   ```env
   AZURE_OPENAI_API_KEY=your_key_here
   AZURE_OPENAI_ENDPOINT=your_endpoint_here
   AZURE_OPENAI_DEPLOYMENT_NAME=your_deployment_here
   ```

### 5. **File Structure After Running**

After running the code, you'll see:
```
external_search/
├── runs/                       # Excel export files
│   ├── run_20241201_143022.xlsx
│   └── run_20241201_143045.xlsx
└── .venv/                     # Virtual environment
```

### 6. **Troubleshooting Common Issues**

#### Chrome/Selenium Issues
```bash
# Install Chrome browser if not present
# On macOS:
brew install --cask google-chrome

# On Ubuntu:
sudo apt-get install google-chrome-stable

# On Windows: Download from https://www.google.com/chrome/
```

#### API Key Issues
```bash
# Check environment variables
echo $GOOGLE_API_KEY
echo $SERPAPI_API_KEY

# Or check .env file
cat .env
```

#### Import Issues
```bash
# Ensure virtual environment is activated
source .venv/bin/activate

# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

## Development Workflow

### 1. Setup Development Environment

```bash
# Clone repository
git clone <repository-url>
cd external_search

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install development dependencies
pip install -r requirements-dev.txt  # If available
```

### 2. Code Structure Guidelines

- **Component Development**: Each component should inherit from `LoggedClass`
- **Error Handling**: Use try-catch blocks with proper logging
- **Type Hints**: Use type hints for all function parameters and return values
- **Documentation**: Follow docstring conventions
- **Testing**: Write tests for new components

### 3. Adding New Components

1. Create component file in `genai_search/core/components/`
2. Inherit from `LoggedClass` for logging capabilities
3. Implement required interface methods
4. Add to `genai_search/core/components/__init__.py`
5. Update orchestrator if needed
6. Write tests
7. Update documentation

### 4. Adding New LLM Providers

1. Update `LLMProvider` enum in `models.py`
2. Add provider detection in `LLMManager`
3. Implement provider-specific initialization
4. Update requirements.txt
5. Add tests and documentation

## Testing

### Running Tests

```bash
# Install pytest
pip install pytest

# Run all tests
pytest

# Run specific test file
pytest tests/test_orchestrator.py

# Run with coverage
pytest --cov=genai_search

# Run with verbose output
pytest -v
```

### Test Structure

```
tests/
├── test_orchestrator.py
├── test_llm_manager.py
├── test_components/
│   ├── test_search_term_generator.py
│   ├── test_web_search_engine.py
│   ├── test_content_extractor.py
│   ├── test_relevance_scorer.py
│   └── test_content_summarizer.py
├── test_utils/
│   ├── test_models.py
│   ├── test_prompts.py
│   └── test_logging_config.py
└── test_exporters/
    └── test_excel_exporter.py
```

### Writing Tests

```python
import pytest
from genai_search.core.orchestrator import GenAISearchOrchestrator

class TestGenAISearchOrchestrator:
    def test_initialization(self):
        """Test orchestrator initialization"""
        orchestrator = GenAISearchOrchestrator()
        assert orchestrator is not None
        assert orchestrator.config is not None
    
    def test_component_initialization(self):
        """Test all components are initialized"""
        orchestrator = GenAISearchOrchestrator()
        assert hasattr(orchestrator, 'search_term_generator')
        assert hasattr(orchestrator, 'web_search_engine')
        assert hasattr(orchestrator, 'content_extractor')
        assert hasattr(orchestrator, 'relevance_scorer')
        assert hasattr(orchestrator, 'content_summarizer')
```

## Troubleshooting

### Common Issues

#### 1. Selenium WebDriver Issues
```bash
# Error: ChromeDriver not found
# Solution: Install Chrome browser and ensure it's in PATH
# Or use headless mode:
config = ComponentConfig(headless_browser=True)
```

#### 2. API Key Errors
```bash
# Error: Missing API key
# Solution: Check .env file and environment variables
echo $GOOGLE_API_KEY
echo $SERPAPI_API_KEY
```

#### 3. Import Errors
```bash
# Error: Module not found
# Solution: Install missing dependencies
pip install -r requirements.txt
```

#### 4. Memory Issues
```bash
# Error: Out of memory during content extraction
# Solution: Reduce max_content_length and max_search_results
config = ComponentConfig(
    max_content_length=3000,
    max_search_results=5
)
```

### Debug Mode

```python
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)

# Or set specific component logging
from genai_search.core.utils.logging_config import setup_logging
setup_logging(log_level="DEBUG")
```

### Performance Monitoring

```python
# Check processing statistics
results = component.process_query("Your query")
stats = results.statistics

print(f"Total processing time: {stats.processing_time:.2f}s")
print(f"Search time: {stats.search_time:.2f}s")
print(f"Extraction time: {stats.extraction_time:.2f}s")
print(f"Scoring time: {stats.scoring_time:.2f}s")
print(f"Summarization time: {stats.summarization_time:.2f}s")
```

## Contributing

### Development Guidelines

1. **Code Style**: Follow PEP 8 guidelines
2. **Type Hints**: Use type hints for all functions
3. **Documentation**: Update docstrings and this guide
4. **Testing**: Write tests for new functionality
5. **Error Handling**: Implement proper error handling
6. **Logging**: Use structured logging with appropriate levels

### Pull Request Process

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add/update tests
5. Update documentation
6. Ensure all tests pass
7. Submit pull request with detailed description

### Code Review Checklist

- [ ] Code follows project style guidelines
- [ ] All tests pass
- [ ] Documentation is updated
- [ ] Error handling is implemented
- [ ] Logging is appropriate
- [ ] Performance considerations addressed
- [ ] Security considerations addressed

## Performance Considerations

### Optimization Tips

1. **Content Length**: Limit `max_content_length` based on needs
2. **Search Results**: Balance `max_search_results` vs. processing time
3. **Browser Management**: Use headless mode for production
4. **Caching**: Enable caching for repeated queries
5. **Rate Limiting**: Respect API rate limits with delays

### Monitoring

- Track processing times for each component
- Monitor memory usage during content extraction
- Log API call frequencies and failures
- Track relevance score distributions

## Security Considerations

1. **API Keys**: Never commit API keys to version control
2. **Input Validation**: Validate all user inputs
3. **Rate Limiting**: Implement proper rate limiting
4. **Error Messages**: Don't expose sensitive information in errors
5. **Dependencies**: Keep dependencies updated for security patches

## Future Enhancements

### Planned Features

1. **Additional LLM Providers**: Support for more LLM services
2. **Advanced Caching**: Redis-based caching system
3. **Async Processing**: Asynchronous content processing
4. **Custom Extractors**: Plugin system for custom content extractors
5. **Analytics Dashboard**: Web-based analytics and monitoring
6. **API Server**: REST API for integration with other services

### Extension Points

1. **Custom Search Engines**: Implement custom search engine interfaces
2. **Content Processors**: Add custom content processing pipelines
3. **Export Formats**: Support for additional export formats
4. **Authentication**: User authentication and authorization
5. **Rate Limiting**: Advanced rate limiting strategies

---

## Quick Run Commands

### **For First-Time Users:**
```bash
# 1. Setup environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 2. Create .env file with your API keys
echo "GOOGLE_API_KEY=your_key_here" > .env
echo "SERPAPI_API_KEY=your_key_here" >> .env

# 3. Run enhanced test
python test_enhanced_export.py
```

### **For Developers:**
```bash
# Run basic example
python example_modular.py

# Run enhanced test
python test_enhanced_export.py

# Run tests
pytest

# Interactive development
python
>>> from genai_search import create_genai_component
>>> component = create_genai_component()
>>> results = component.process_query("test query")
>>> component.close()
```

This developer guide provides a comprehensive overview of the GenAI Search Component architecture, development workflow, and enhanced features. For additional questions or contributions, please refer to the project repository or create an issue. 