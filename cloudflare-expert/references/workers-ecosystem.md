<overview>
Cloudflare's developer platform: Workers, Pages, KV, R2, D1, Queues, and more.
</overview>

<workers>
**Serverless JavaScript/TypeScript at the edge.**

<basics>
- Runs in 300+ locations worldwide
- Sub-millisecond cold starts
- V8 isolates (not containers)
- Free tier: 100,000 requests/day
</basics>

<use_cases>
- API endpoints
- Request/response modification
- A/B testing
- Authentication
- Rate limiting
- Redirects
- Edge-side rendering
</use_cases>

<simple_example>
```javascript
export default {
  async fetch(request, env, ctx) {
    return new Response("Hello World!");
  },
};
```
</simple_example>

<api_commands>
```bash
# List Workers
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/workers/scripts" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Deploy Worker
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/workers/scripts/my-worker" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/javascript" \
  --data 'export default { fetch() { return new Response("Hi") } }'

# Create route
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/workers/routes" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"pattern":"example.com/api/*","script":"my-worker"}'
```
</api_commands>
</workers>

<pages>
**Static site and full-stack hosting.**

<features>
- Git integration (GitHub, GitLab)
- Automatic builds and deploys
- Preview deployments for PRs
- Functions (Workers at /functions)
- Free unlimited bandwidth
</features>

<supported_frameworks>
Next.js, Nuxt, Astro, SvelteKit, Remix, Gatsby, Hugo, Jekyll, Eleventy, and any static generator.
</supported_frameworks>

<api_commands>
```bash
# List Pages projects
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/pages/projects" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Get project details
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/pages/projects/project-name" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# List deployments
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/pages/projects/project-name/deployments" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</api_commands>
</pages>

<kv>
**Key-Value storage at the edge.**

<characteristics>
- Eventually consistent (reads may be stale)
- Great for: config, feature flags, cached data
- Not for: frequently updated data, counters
- Free tier: 100,000 reads/day, 1,000 writes/day
</characteristics>

<api_commands>
```bash
# List KV namespaces
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/storage/kv/namespaces" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Create namespace
curl -X POST "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/storage/kv/namespaces" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"title":"my-namespace"}'

# Write key
curl -X PUT "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/storage/kv/namespaces/${NS_ID}/values/my-key" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  --data "my-value"

# Read key
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/storage/kv/namespaces/${NS_ID}/values/my-key" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"
```
</api_commands>
</kv>

<r2>
**S3-compatible object storage. Zero egress fees.**

<features>
- S3 API compatible
- No egress charges (huge cost savings)
- Automatic replication
- Public bucket option
- Free tier: 10GB storage
</features>

<api_commands>
```bash
# List buckets
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/r2/buckets" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Create bucket
curl -X POST "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/r2/buckets" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-bucket"}'
```
</api_commands>
</r2>

<d1>
**SQLite database at the edge.**

<features>
- SQL database (SQLite)
- Automatic replication
- Read replicas worldwide
- Free tier: 5GB, 5M reads/day
</features>

<api_commands>
```bash
# List databases
curl -s "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/d1/database" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f"

# Create database
curl -X POST "https://api.cloudflare.com/client/v4/accounts/498725db75d32e48d269120f12bdf2e6/d1/database" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"name":"my-database"}'
```
</api_commands>
</d1>

<queues>
**Message queues for background processing.**

<use_cases>
- Background jobs
- Event processing
- Decoupling services
- Retry logic
</use_cases>
</queues>

<durable_objects>
**Stateful serverless for real-time apps.**

<use_cases>
- WebSocket servers
- Collaborative editing
- Game state
- Rate limiting with precision
- Coordination between requests
</use_cases>
</durable_objects>

<ai>
**Inference API for ML models.**

<models>
- Text generation (Llama, Mistral)
- Image generation (Stable Diffusion)
- Embeddings
- Speech recognition
- Translation
</models>
</ai>

<vectorize>
**Vector database for AI/ML.**

<use_cases>
- Semantic search
- RAG (Retrieval Augmented Generation)
- Recommendation systems
- Similarity matching
</use_cases>
</vectorize>
