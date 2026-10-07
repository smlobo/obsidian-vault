```mermaid
flowchart LR
    Input[Input] --> Process[Processing]
    Process --> Decision{Valid?}
    Decision -- Yes --> Output[Output]
    Decision -- No --> Error[Show error]
    Error --> Input
```
