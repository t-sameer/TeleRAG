# TeleRAG-4 Simplified Architecture (PPT Landscape)

> **Instructions for PPT:** Copy and paste the Mermaid code below into a Mermaid visualizer (like [mermaid.live](https://mermaid.live) or a native PPT Mermaid add-in), then export as an image or insert directly into your slide. It is specifically designed to flow left-to-right (LR) to maximize the use of widescreen landscape space.

```mermaid
flowchart TB
    %% Stacked 3-Tier Architecture for PPT Presentation
    
    subgraph Top ["1. Input & Routing Layer"]
        direction LR
        UI[Streamlit Interface<br/>Q&A / Log Upload] --> Router{Semantic Router}
    end

    subgraph Mid ["2. Processing Pipelines"]
        direction LR
        subgraph Ret ["Retrieval Pipeline"]
            DB[(Vector & Keyword Indices<br/>3GPP / O-RAN)] --> Search[Hybrid Search +<br/>Cross-Encoder]
        end
        subgraph Logic ["Reasoning Pipeline"]
            RCA[RCA Chain-of-Thought<br/>Engine]
        end
    end

    subgraph Bottom ["3. Generation & Output Layer"]
        direction LR
        LLM[Gemma 4 Generator<br/>w/ Vision] --> Eval{CRAG &<br/>Self-RAG}
        Eval -->|Pass| Out[Final Answer +<br/>Citations]
    end

    %% Cross-layer flows
    Router -->|Text Query| DB
    Router -->|Anomaly Logs| RCA
    
    Search --> LLM
    RCA --> LLM
    
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
* **Compact 3-Tier Layout:** Restructured from a flat Left-to-Right flow into a stacked Top-to-Bottom structure to better utilize vertical slide space.
* **Flattened Retrieval:** The 4-layer retrieval (BGE, RRF, Cross-Encoder, CRAG grader) is compressed into a single "Search" box.
* **Abstracted Data Sources:** 3GPP specs, ORAN-Bench, and TeleQnA are grouped under a single "Vector & Keyword Indices" database icon.
* **Unified Generation:** Context assembly, citation enforcement, and vision processing are merged into the central "Gemma 4 Generator" node.
