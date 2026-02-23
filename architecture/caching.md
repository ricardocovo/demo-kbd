# Caching & CDN Patterns

Purpose
- Guidance for application-level caching, distributed caches, and CDN usage.

Caching tiers
- In-process cache: for ephemeral, low-latency caches (e.g., local LRU cache) with short TTLs.
- Distributed cache: Redis or Memcached for cross-instance caching, locking, and pub/sub patterns.
- CDN: use for static asset delivery (images, JS/CSS) and edge caching of API responses when safe.

Cache invalidation
- Prefer cache-aside or write-through patterns; document invalidation rules clearly.
- Use versioned keys or tags to make broad invalidations safe and efficient.

Consistency
- Avoid using cache as the primary source of truth; design for eventual consistency and define tolerances for staleness.

Security
- Secure Redis with network controls and authentication; avoid exposing caching layers publicly.

Monitoring
- Track cache hit/miss rates and key churn; set alerts for low hit rates or high evictions.
