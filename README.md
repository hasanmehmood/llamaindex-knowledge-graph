# Knowledge Graph Index with NebulaGraph and OpenAI LLM

This repository contains code to set up a Knowledge Graph Index using NebulaGraph as the storage backend and OpenAI's Language Model (LLM) for query processing.

## Setup

### Environment Configuration

1. **Load Environment Variables**: Ensure you have a `.env` file set up with required environment variables. Use `dotenv` to load them in your Python environment.

    ```python
    import os
    from dotenv import load_dotenv

    load_dotenv()
    ```

2. **Logging Configuration**: Configure the logging settings.

    ```python
    import logging
    import sys

    logging.basicConfig(
        stream=sys.stdout, level=logging.INFO
    )  # logging.DEBUG for more verbose output
    ```

3. **Import Dependencies**: Import necessary libraries and modules.

    ```python
    # Import necessary modules and libraries
    from llama_index import (
        KnowledgeGraphIndex,
        LLMPredictor,
        ServiceContext,
        SimpleDirectoryReader,
    )
    from llama_index.storage.storage_context import StorageContext
    from llama_index.graph_stores import NebulaGraphStore
    from llama_index.llms import OpenAI
    from IPython.display import Markdown, display
    ```

### NebulaGraph Setup

Ensure NebulaGraph (version 3.5.0 or newer) is set up with the following steps:

1. **Cluster Creation**: Create a NebulaGraph cluster using one of the following options:
   
   - **Docker Installation (Machines with Docker Installed)**:
     - Option 0: Run the command: `curl -fsSL nebula-up.siwei.io/install.sh | bash`
   
   - **Desktop Installation**:
     - Option 1: Install NebulaGraph Docker Extension from [Docker Hub](https://hub.docker.com/extensions/weygu/nebulagraph-dd-ext).
   
   - **Manual Setup via NebulaGraph Console**: If the above options are not applicable, manually create the NebulaGraph space, tags, edges, and indexes using the NebulaGraph console. Refer to the provided commands in the code comments.

2. **Environment Configuration**: Set NebulaGraph environment variables.

    ```python
    os.environ["NEBULA_USER"] = "root"
    os.environ["NEBULA_PASSWORD"] = "nebula"  # default is "nebula"
    os.environ["NEBULA_ADDRESS"] = "127.0.0.1:9669"  # assumed NebulaGraph installed locally
    ```

### Knowledge Graph Indexing

1. **Define Graph Structure**: Define the graph structure (space, tags, edges).

    ```python
    space_name = "llamaindex"
    edge_types, rel_prop_names = ["relationship"], ["relationship"]
    tags = ["entity"]
    ```

2. **Initialize NebulaGraph Store and Storage Context**:

    ```python
    graph_store = NebulaGraphStore(
        space_name=space_name,
        edge_types=edge_types,
        rel_prop_names=rel_prop_names,
        tags=tags,
    )
    storage_context = StorageContext.from_defaults(graph_store=graph_store)
    ```

3. **Load Data**: Load documents/data to be indexed.

    ```python
    documents = SimpleDirectoryReader("data").load_data()
    ```

4. **Create Knowledge Graph Index**:

    ```python
    kg_index = KnowledgeGraphIndex.from_documents(
        documents,
        storage_context=storage_context,
        max_triplets_per_chunk=10,
        service_context=service_context,
        space_name=space_name,
        edge_types=edge_types,
        rel_prop_names=rel_prop_names,
        tags=tags,
        include_embeddings=True,
    )
    ```

### Querying the Knowledge Graph

1. **Initialize Query Engine**:

    ```python
    from llama_index.query_engine import KnowledgeGraphQueryEngine

    query_engine = KnowledgeGraphQueryEngine(
        storage_context=storage_context,
        service_context=service_context,
        llm=llm,
        verbose=True,
    )
    ```

2. **Querying**:

    ```python
    response = query_engine.query(
        "your question on the data?",
    )
    display(Markdown(f"<b>{response}</b>"))
    ```

## Further Customization

- Adjust parameters and configurations in the code based on specific requirements or environment setup.
