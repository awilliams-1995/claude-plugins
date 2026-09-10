# claude-plugins

Reusable agent tooling, shared across projects. Add this directory as a local plugin marketplace:

```sh
/plugin marketplace add ~/Developer/claude-plugins
```

| Plugin | What it does |
|---|---|
| [`plan-execution`](plan-execution/) | Executes a large multi-step implementation plan: per-step pre-flight, delegated implementation, independent adversarial review, scoped fix verification, and a durable ledger that survives compaction. |
