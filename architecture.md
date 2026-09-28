# Runtime path

One run, four possible outcomes. The sufficiency gate is the only thing that
decides between a teardown and a refusal.

```mermaid
flowchart TD
    A["Hermes cron job, 12:15 IST"] --> B["Hermes invokes LaunchLens skill"]
    B --> C["Step 1: Product Hunt leaderboard<br/>today, then yesterday, then weekly"]
    C -->|"every source failed"| F1["RUN FAILED message"]
    C -->|"top 5 candidates"| D["Step 2: read processed.log and refusals.log<br/>drop seen launches, recall memory notes"]
    D -->|"log unreadable"| F1
    D -->|"all candidates seen"| N1["NO NEW LAUNCHES message"]
    D -->|"new candidates"| S["Step 3: select one launch"]
    S --> H["Step 4: Hacker News Show page<br/>selected launch only"]
    H --> L["Step 5: product landing and pricing page"]
    L --> G{"Step 6: sufficiency gate<br/>MIN_OBSERVED_FIELDS of 4 GTM fields observed?"}
    G -->|"no"| R["REFUSAL output with reasons"]
    G -->|"yes"| T["TEARDOWN output"]
    R --> RL["Step 8: append refusals.log"]
    T --> TL["Step 8: append processed.log<br/>update memory note"]
    RL --> Z["Final message"]
    TL --> Z
    F1 --> Z
    N1 --> Z
    Z --> TG["Hermes scheduler delivers to Telegram"]
```

Notes:

- The gate threshold is `MIN_OBSERVED_FIELDS` in the Config block of `SKILL.md`.
  It is set to 3 of 4 fields.
- Logging happens before the final message because the skill cannot observe delivery.
  A log line means the output was produced, not that it arrived.
- The cron node reflects the author's own machine. The job definition is in
  `hermes-cron-job.example.json`.
