# GenAI Search Component

A reusable Python component that combines the power of Large Language Models (LLMs) with web search capabilities to provide intelligent content discovery, extraction, and summarization.

## Features

- **Search Term Generation**: Uses LLM to generate optimized search terms from user queries
- **Web Search**: Performs web searches using Google SerpAPI
- **Content Extraction**: Extracts content from web pages using Selenium
- **Relevance Scoring**: Evaluates content relevance using LLM
- **Content Summarization**: Creates comprehensive summaries from relevant content with **source citations**
- **Multi-Provider Support**: Supports both Google Generative AI and Azure OpenAI
- **Configurable**: Highly customizable with various parameters
- **Enhanced Excel Export**: Professional styling with source citations and reference tables

## Workflow

1. **User Query** → Input query from user
2. **Search Term Generation** → LLM generates optimized search terms
3. **Web Search** → Google SerpAPI searches for relevant content
4. **Content Extraction** → Selenium extracts content from web pages
5. **Relevance Scoring** → LLM evaluates content relevance
6. **Content Summarization** → LLM creates comprehensive summary

## Installation

### Prerequisites

- Python 3.8+
- Chrome browser (for Selenium)
- ChromeDriver (automatically managed by Selenium)

### Install Dependencies

```bash
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file in your project root:

```env
# For Google Generative AI
GOOGLE_API_KEY=your_google_api_key_here

# For Azure OpenAI
AZURE_OPENAI_API_KEY=your_azure_openai_api_key_here
AZURE_OPENAI_ENDPOINT=your_azure_openai_endpoint_here
AZURE_OPENAI_DEPLOYMENT_NAME=your_deployment_name_here

# For Web Search (required for both providers)
SERPAPI_API_KEY=your_serpapi_key_here
```

## Quick Start

### Basic Usage

```python
from genai_search_component import create_genai_component

# Create component with Google Generative AI
component = create_genai_component(
    llm_provider="google",
    google_api_key="your_google_api_key",
    serpapi_api_key="your_serpapi_key"
)

# Process a query
results = component.process_query("What are the latest developments in quantum computing?")

# Get the summary
print(results['summary'])

# Clean up
component.close()
```

### Azure OpenAI Usage

```python
from genai_search_component import create_genai_component

# Create component with Azure OpenAI
component = create_genai_component(
    llm_provider="azure",
    azure_openai_api_key="your_azure_api_key",
    azure_openai_endpoint="your_azure_endpoint",
    azure_openai_deployment_name="your_deployment_name",
    serpapi_api_key="your_serpapi_key"
)

# Process a query
results = component.process_query("What are the best cybersecurity practices for 2024?")

print(results['summary'])
component.close()
```

## Configuration Options

### GenAISearchComponent Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `llm_provider` | str | "google" | "google" or "azure" |
| `google_api_key` | str | None | Google Generative AI API key |
| `azure_openai_api_key` | str | None | Azure OpenAI API key |
| `azure_openai_endpoint` | str | None | Azure OpenAI endpoint |
| `azure_openai_deployment_name` | str | None | Azure OpenAI deployment name |
| `serpapi_api_key` | str | None | SerpAPI key for web search |
| `max_search_results` | int | 10 | Maximum search results to process |
| `max_content_length` | int | 5000 | Maximum content length to extract |
| `relevance_threshold` | float | 0.7 | Minimum relevance score threshold |
| `headless_browser` | bool | True | Run browser in headless mode |

## API Reference

### GenAISearchComponent Methods

#### `process_query(user_query: str) -> Dict[str, Any]`

Main method that processes a user query through the complete workflow.

**Parameters:**
- `user_query` (str): The user's query

**Returns:**
- Dictionary containing:
  - `user_query`: Original query
  - `search_terms`: Generated search terms
  - `search_results`: Web search results
  - `extracted_contents`: Extracted content with relevance scores
  - `summary`: Final summary
  - `statistics`: Processing statistics

#### `generate_search_terms(user_query: str) -> List[str]`

Generates optimized search terms from a user query.

#### `web_search(search_terms: List[str]) -> List[SearchResult]`

Performs web search using the provided search terms.

#### `extract_content(search_results: List[SearchResult]) -> List[ExtractedContent]`

Extracts content from search result URLs.

#### `calculate_relevance_score(user_query: str, extracted_contents: List[ExtractedContent]) -> List[ExtractedContent]`

Calculates relevance scores for extracted content.

#### `summarize_content(user_query: str, relevant_contents: List[ExtractedContent]) -> str`

Creates a summary from relevant content.

#### `close()`

Cleans up resources (closes Selenium WebDriver).

## Data Structures

### SearchResult
```python
@dataclass
class SearchResult:
    title: str
    url: str
    snippet: str
    relevance_score: float = 0.0
```

### ExtractedContent
```python
@dataclass
class ExtractedContent:
    url: str
    title: str
    content: str
    relevance_score: float
    word_count: int
```

## Examples

### Example 1: Research Assistant

```python
from genai_search_component import create_genai_component

component = create_genai_component(
    llm_provider="google",
    max_search_results=8,
    relevance_threshold=0.6
)

# Research a topic
results = component.process_query("What are the environmental impacts of electric vehicles?")

print("Research Summary:")
print(results['summary'])

print(f"\nSources consulted: {results['statistics']['total_extracted_contents']}")
print(f"Relevant sources: {results['statistics']['relevant_contents']}")

component.close()
```

### Example 2: Content Analysis

```python
from genai_search_component import create_genai_component

component = create_genai_component(
    llm_provider="azure",
    max_search_results=5,
    relevance_threshold=0.8
)

# Analyze content relevance
results = component.process_query("Latest developments in machine learning")

# Show detailed results
for content in results['extracted_contents']:
    if content['relevance_score'] >= 0.8:
        print(f"Highly Relevant: {content['title']}")
        print(f"Score: {content['relevance_score']:.2f}")
        print(f"URL: {content['url']}")
        print("-" * 50)

component.close()
```

### Example 3: Batch Processing

```python
from genai_search_component import create_genai_component
import json

component = create_genai_component(
    llm_provider="google",
    max_search_results=3
)

queries = [
    "Python best practices",
    "Data science trends 2024",
    "Cloud computing benefits"
]

all_results = {}

for query in queries:
    print(f"Processing: {query}")
    results = component.process_query(query)
    all_results[query] = results

# Save results
with open("batch_results.json", "w") as f:
    json.dump(all_results, f, indent=2, default=str)

component.close()
```

### Example 4: Enhanced Excel Export with Source Citations

The enhanced Excel exporter provides professional styling and source citations:

```python
from genai_search.exporters import ExcelExporter

# Export with enhanced styling and citations
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

print(f"Enhanced Excel file saved: {xlsx_path}")
```

**Enhanced Excel Features:**
- **Professional Styling**: Color-coded headers, borders, and alternating row colors
- **Source Citations**: Inline citations in summaries using [Source X] format
- **Reference Tables**: Complete source reference information with URLs and metadata
- **Citation Highlighting**: Visual indicators for cited content
- **Structured Layout**: Organized worksheets with clear sections and titles
- **Auto-formatting**: Automatic column width adjustment and text wrapping

## Error Handling

The component includes comprehensive error handling:

- **Missing API Keys**: Clear error messages for missing credentials
- **Network Issues**: Graceful handling of web search failures
- **Content Extraction**: Fallback mechanisms for failed extractions
- **LLM Errors**: Robust error handling for LLM API calls

## Performance Considerations

- **Rate Limiting**: Built-in delays to respect API rate limits
- **Content Length**: Configurable content length limits
- **Browser Management**: Automatic WebDriver cleanup
- **Memory Management**: Efficient processing of large content

## Troubleshooting

### Common Issues

1. **ChromeDriver not found**: Install Chrome browser and ensure it's in PATH
2. **API key errors**: Verify your API keys are correct and have proper permissions
3. **Selenium errors**: Ensure Chrome browser is installed and up to date
4. **Network timeouts**: Check your internet connection and API service status

### Debug Mode

Enable debug logging:

```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
1. Check the troubleshooting section
2. Review the examples
3. Open an issue on GitHub

## Changelog

### Version 1.0.0
- Initial release
- Support for Google Generative AI and Azure OpenAI
- Complete workflow implementation
- Comprehensive error handling
- Extensive documentation and examples 