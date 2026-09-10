# claude-plugins

Reusable agent tooling for planning, executing and reviewing multi-step engineering work.

Add this repository as a plugin marketplace:

```sh
/plugin marketplace add awilliams-1995/claude-plugins
```

| Plugin | What it does |
|---|---|
| [`plan-execution`](plan-execution/) | Develops an idea into a spec and a reviewed plan, then executes it: per-step pre-flight against the tree as it actually is, delegated implementation, independent adversarial review, scoped fix verification, and a durable ledger that survives context compaction. |

```sh
/plugin install plan-execution@awilliams-1995-plugins
```

MIT licensed.
