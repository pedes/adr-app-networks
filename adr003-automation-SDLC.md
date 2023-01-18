# Automation Pipeline


```mermaid
flowchart TD
    Design[Anypoint CLI Governance] --> A
    A[Start] --> B{Governance 70% Rules/Best Practices?}
    B -->|Yes| C[OK]
    C --> D[Rethink]
    D --> B
    B ---->|No| E[End]
```
