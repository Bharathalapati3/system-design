# Content Delivery Network (CDN)

A Content Delivery Network (CDN) is a geographically distributed network of edge servers that cache and deliver content to users from locations close to them. Using a CDN reduces latency, offloads traffic from your origin, and improves availability.

## Overview
- CDNs cache content (static and sometimes dynamic) at edge locations worldwide.
- On a cache hit, the CDN serves the response from the edge. On a cache miss, the CDN forwards the request to the origin server and may cache the response depending on headers and configuration.
- Modern CDNs also provide edge compute, request routing, TLS termination, WAF, rate limiting, analytics, and more.

## How CDNs work (high level)
1. User requests a resource.
2. DNS or Anycast routes the request to a nearby edge server.
3. The edge server checks its cache:
   - If cached and valid (hit): return from edge.
   - If missing or expired (miss): fetch from origin (or an upstream cache/shield), return to user, and optionally cache the response.
4. The CDN can be configured to cache based on headers, URL patterns, cookies, and custom cache keys.

## Caching strategies
- Cache-pull (On-demand): The first user request results in an origin fetch; the response is cached at the edge for subsequent users.
- Cache-push (Pre-warming): Origin proactively pushes content to edge nodes (useful for large or time-sensitive assets).
- Hybrid: Combine push for critical assets and pull for general content.

## Cache-control and headers
- The origin should control CDN caching with HTTP cache headers:
  - `Cache-Control: public, max-age=<seconds>` — browser and CDN TTL.
  - `Cache-Control: s-maxage=<seconds>` — CDN/Shared cache TTL (overrides max-age for shared caches).
  - `Cache-Control: private` — indicates responses should not be stored by shared caches (CDNs).
  - `Cache-Control: no-cache` / `no-store` — prevent caching or require revalidation.
  - `stale-while-revalidate=<seconds>` — serve stale content while revalidating in the background (reduces latency).
  - `stale-if-error=<seconds>` — serve stale content when origin is unreachable.
- `Expires` is legacy; prefer `Cache-Control`.
- `Vary` header affects cache keys (e.g., `Vary: Accept-Encoding`, `Vary: Cookie`).

## Cache keys and personalization
- Cache key determines when different requests map to the same cached response.
- Common cache key components: protocol, host, path, query-string parameters, selected headers, cookies.
- Avoid including user-specific cookies or Authorization in cache keys unless you intentionally serve personalized cached content.
- For personalization:
  - Use Edge logic to vary or assemble responses (edge compute or partial caching).
  - Use cacheable fragments or edge-side includes (ESI) when supported.

## Edge compute and dynamic content
- Modern CDNs (Cloudflare Workers, Fastly Compute, Lambda@Edge) can run code at the edge to:
  - Modify requests/responses
  - Generate dynamic content
  - Implement A/B tests, authentication checks, or personalization
- Even dynamic pages can be partially cached (e.g., cache shared sections and fill user-specific parts at render time).

## Invalidation, purging and cache lifecycle
- Invalidation (purge) options:
  - Time-based TTL expiry (let TTLs expire naturally).
  - Purge by URL or tag: send API calls to CDN to evict content immediately.
  - Soft purge vs hard purge (some CDNs return stale content while fetching fresh).
- Use cache tagging (where available) to purge groups of resources efficiently (e.g., `tag: product-123`).
- Purging can be rate-limited or costly; design cache-control and TTLs to minimize frequent purges.

## Origin shielding and multi-tier caches
- Origin shield or upstream cache reduces the load on origin by consolidating origin fetches through a single region/edge.
- Multi-tier caching (edge → regional → origin) can improve cache hit ratio and reduce origin bandwidth.

## Security, reliability and features
- TLS termination and HTTPS at the edge.
- DDoS mitigation, rate limiting, and Web Application Firewall (WAF).
- Access controls and signed URLs for private assets.
- Compression (gzip, brotli) and image optimizations at the edge.
- Logging and analytics — monitor cache hit ratio, origin bandwidth, latency.

## Performance and cost tradeoffs
- Short TTLs reduce staleness but increase origin load and bandwidth costs.
- Long TTLs lower origin load but can serve stale content unless you use revalidation strategies (`stale-while-revalidate`).
- Use a combination of long TTLs for most assets and targeted short TTLs or purges where freshness is critical.

## Best practices
- Set `s-maxage` for CDN TTLs and `max-age` for browsers when appropriate.
- Prefer `Cache-Control: public, s-maxage=3600, stale-while-revalidate=30` for cacheable content that tolerates short staleness.
- Avoid caching responses that contain user-specific data unless you separate shared and private content.
- Use consistent cache keys and avoid accidentally varying on headers that are not meaningful.
- Test and measure: monitor cache hit ratio, P95/P99 latency, and origin bandwidth.
- Implement origin shielding and CDN logging for observability.
- Plan and document your invalidation strategy to avoid surprises and costs.

## Example Cache-Control headers
- Static asset (long-lived):
  - `Cache-Control: public, max-age=31536000, immutable`
- Cache at CDN for 1 hour, browsers 5 minutes, allow stale while revalidate:
  - `Cache-Control: public, s-maxage=3600, max-age=300, stale-while-revalidate=30`
- Private or personalized:
  - `Cache-Control: private, max-age=0, no-cache`

## Vendors and features
- Popular CDN/edge offerings: Cloudflare (Workers), Fastly (Compute), AWS CloudFront (Lambda@Edge / CloudFront Functions), Akamai.
- Different vendors have different features for tagging, purging, edge compute, and configuration — consult vendor docs for advanced configs.
