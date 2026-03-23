# Token Optimization Protocol — AutoNexus

Cost-aware execution to reduce token usage 30-50%.

## Strategies
1. ReasoningBank Retrieval (-15-32%): Reuse patterns if confidence > 0.7
2. Tier-1 Direct Edits (-10-15%): Route deterministic transforms to Edit tool
3. Result Caching (-5-10%): Cache verification and Obsidian query results
4. Prompt Compression (-5-10%): Only relevant file contents, summarize prior outputs
5. Batch Operations (-5-10%): Combine related operations into single agent calls

## Cost Tracking
Per-session: Tokens Used, Savings %, Tier breakdown, Cache hit rate.

## Obsidian Layout
Projects/{ProjectName}/Optimization/ -- token-log.md, routing-log.md, cache-stats.md

## Integration Points
Applied every phase and agent spawn. Reported via /autonexus:optimize.
