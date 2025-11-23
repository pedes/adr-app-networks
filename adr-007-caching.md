ADR 001: REST API Caching Strategy Using MuleSoft Object Store
Context
Our MuleSoft integration layer handles numerous REST API calls to backend systems, resulting in:

High latency for frequently requested data that changes infrequently
Increased load on backend systems due to repetitive requests for the same data
Potential rate limiting issues with external APIs
Higher infrastructure costs due to unnecessary processing

Key constraints and requirements:

Need to reduce response times for read-heavy API operations
Must minimize backend system load while maintaining data consistency
Cache invalidation strategy must prevent stale data issues
Solution must be scalable across CloudHub workers
Must support both CloudHub and on-premises deployment models
Need visibility into cache performance metrics

The MuleSoft Object Store v2 provides a suitable distributed caching solution with TTL (Time-To-Live) capabilities, CloudHub compatibility, and persistence options.
Decision
We will implement a caching layer using MuleSoft Object Store v2 for REST API responses with the following approach:
Accepted Solution

Cache-Aside Pattern: Check cache before invoking backend APIs; populate cache on miss
Object Store v2 as the caching mechanism due to:

Native MuleSoft integration
Distributed cache across CloudHub workers
Built-in TTL support for automatic expiration
No additional infrastructure required


Cache Key Strategy: Composite keys combining endpoint path + query parameters + relevant headers
TTL-based Expiration: Configure TTL values based on data volatility (5 minutes to 24 hours)
Selective Caching: Cache only GET operations with 200 OK responses
Error Handling: On cache failures, bypass cache and invoke backend directly

Options Considered and Rejected

Redis/External Cache: Rejected due to additional infrastructure complexity and cost
In-Memory Cache: Rejected due to lack of distribution across workers in CloudHub
Database Caching: Rejected due to performance overhead and complexity
Write-Through Pattern: Rejected as most APIs are read-heavy and we don't control backend updates

Status
Accepted
Consequences
Positive

Performance: 70-90% reduction in response time for cached responses (typically <50ms vs 200-2000ms)
Scalability: Reduced backend load enables handling higher traffic volumes
Cost Efficiency: Lower CloudHub vCore utilization and reduced backend API call costs
Reliability: Cache provides limited fallback during backend outages (within TTL window)
Developer Experience: Reusable caching components promote consistency across APIs

Negative

Data Staleness: Cached data may be outdated until TTL expires (mitigated by appropriate TTL tuning)
Storage Costs: Object Store usage incurs costs (monitored through CloudHub metrics)
Complexity: Additional logic for cache key generation and invalidation
Debugging: Cached responses may complicate issue troubleshooting (requires cache clearing capability)
Object Store Limits: Subject to CloudHub Object Store quotas (10 GB per environment in base tier)

Risks and Mitigation

Risk: Stale data causing business issues → Mitigation: Conservative TTL values for critical data; implement cache refresh API
Risk: Object Store quota exhaustion → Mitigation: Monitoring alerts; selective caching of high-value endpoints only
Risk: Cache key collisions → Mitigation: Include all relevant parameters in key generation; use consistent hashing

Dependencies

Object Store v2 connector (included in Mule 4.x)
CloudHub Object Store service or on-premises Object Store configuration
Monitoring tools for cache hit/miss ratio tracking

Related Documents

MuleSoft Object Store v2 Documentation
CloudHub Object Store Limits and Quotas
API Design Guidelines - Performance Requirements
Infrastructure Architecture - CloudHub Standards
Monitoring and Observability Standards
