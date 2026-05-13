# TeleRAG-4 Simplified Architecture (PPT Landscape)

> **Instructions for PPT:** Copy and paste the Mermaid code below into a Mermaid visualizer (like [mermaid.live](https://mermaid.live) or a native PPT Mermaid add-in), then export as an image or insert directly into your slide. It is specifically designed to flow left-to-right (LR) to maximize the use of widescreen landscape space.

```mermaid
flowchart LR
    %% Minimal Landscape Architecture for PPT Presentation
    
    subgraph S1 ["1. User Input"]
        UI[Streamlit Interface<br/>Q&A / Log Upload]
    end

    subgraph S2 ["2. Intelligent Routing"]
        Router{Semantic<br/>Router}
    end

    subgraph S3 ["3. Knowledge Retrieval"]
        DB[(Vector & Keyword Indices<br/>3GPP / O-RAN / Telecom)]
        Search[Hybrid Search +<br/>Cross-Encoder Rerank]
    end

    subgraph S4 ["4. Core Reasoning"]
        RCA[RCA Chain-of-Thought<br/>Engine]
        LLM[Gemma 4 Generator<br/>w/ Vision Encoder]
    end

    subgraph S5 ["5. Validation & Output"]
        Eval{CRAG &<br/>Self-RAG}
        Out[Final Answer +<br/>Source Citations]
    end

    %% Flow paths
    UI --> Router
    
    %% Split flows
    Router -->|Text Query| DB
    Router -->|Anomaly Logs| RCA
    
    %% Retrieval pipeline
    DB --> Search
    Search --> LLM
    
    %% Generation pipeline
    RCA --> LLM
    LLM --> Eval
    
    %% Evaluation loop
    Eval -->|Pass Faithfulness| Out
    Eval -.->|Fail Retry| DB
    
    %% Styling for presentation
    classDef default fill:#f9f9f9,stroke:#333,stroke-width:2px;
    classDef highlight fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    classDef logic fill:#fff3e0,stroke:#f57c00,stroke-width:2px;
    
    class UI,Out default;
    class Router,Eval logic;
    class DB,Search,RCA,LLM highlight;
```

### Key Differences from Actual Architecture (for talking points):
* **Flattened Retrieval:** The 4-layer retrieval (BGE, RRF, Cross-Encoder, CRAG grader) is compressed into a single "Search" box to save visual space.
* **Abstracted Data Sources:** 3GPP specs, ORAN-Bench, and TeleQnA are grouped under a single "Vector & Keyword Indices" database icon.
* **Unified Generation:** Context assembly, citation enforcement, and vision processing are merged into the central "Gemma 4 Generator" node.
* **Linear Flow:** Switched from Top-to-Bottom (`TB`) to Left-to-Right (`LR`) to perfectly fit a standard 16:9 presentation slide without squishing.
