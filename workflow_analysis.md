# MCP Server Workflow Analysis

## Overview
The `kv-extractor-mcp-server` is designed to extract structured key-value pairs from unstructured text. It uses a multi-stage pipeline combining spaCy for Named Entity Recognition (NER) and Large Language Models (LLMs) (specifically GPT-4.1-mini and GPT-4.1) for extraction, type inference, and validation.

## Core Pipeline (`extract_kv_pipeline`)
The main processing logic resides in the `extract_kv_pipeline` function in `server.py`.

### Step 0: Preprocessing (NER)
- **Function**: `extract_phrases_with_spacy_multilang`
- **Logic**: Uses `langdetect` to identify the language. Based on the language ('ja', 'en', 'zh-cn', 'zh-tw'), it loads the corresponding spaCy model to extract named entities.
- **Purpose**: Provides context-aware candidate phrases to the LLM to improve extraction accuracy.

### Step 1: Key-Value Extraction
- **Agent**: `agent_main` (GPT-4.1-mini)
- **Logic**: The LLM extracts key-value pairs from the input text, using the spaCy-extracted phrases as hints. The prompt (`build_kv_extraction_prompt`) guides the model to handle various formats.

### Step 2: Type Annotation
- **Agent**: `agent_main` (GPT-4.1-mini)
- **Logic**: The LLM infers the data type (e.g., `str`, `int`, `list`, `bool`) for each extracted key-value pair.

### Step 3: Type Evaluation
- **Agent**: `agent_eval` (GPT-4.1)
- **Logic**: A more capable model reviews the type annotations for correctness and context, fixing potential errors (e.g., distinguishing between a number and a string representation of a number).

### Step 4: Structuring & Validation
- **Agent**: `agent_eval` (GPT-4.1)
- **Logic**: The confirmed key-value-type triplets are structured into a `KVPayload` object. This step uses Pydantic for strict schema validation.

### Step 5: Type Normalization (Hybrid)
- **Function**: `normalize_types_v2`
- **Logic**:
    1.  **Static Rules**: Applies regex-based rules (`STATIC_NORMALIZATION_RULES`) to convert common patterns (dates, numbers, booleans) into Python native known types.
    2.  **LLM Fallback**: If static rules fail, `llm_normalize_value` uses the LLM to normalize the value based on the expected type.

### Step 6: Final Output Construction
- **Logic**: The normalized values are aggregated into a Python dictionary. Repeated keys are merged into lists.
- **Formatting**: The dictionary is serialized into the requested format:
    - `extract_json` -> JSON
    - `extract_yaml` -> YAML
    - `extract_toml` -> TOML (with special handling for nested structures)

## Architecture
- **Framework**: Built using `FastMCP` (fastmcp) and `pydantic-ai`.
- **Concurrency**: Operations like type normalization are processed concurrently using `asyncio.gather`.
- **Logging**: Configurable logging to an absolute file path.
