<required_reading>
**Read these reference files:**
- references/workers-ecosystem.md - For developer platform options
- references/security.md - For security feature options
- references/zero-trust.md - For enterprise/team features
</required_reading>

<process>

<step name="understand_project">
**Step 1: Understand the Project**

Ask about:
1. **Type** - Static site, SPA, API, full-stack app, blog, e-commerce?
2. **Traffic** - Expected visitors/requests per month?
3. **Security needs** - Public, authenticated users, sensitive data?
4. **Budget** - Free tier only, or willing to pay for features?
5. **Current setup** - Where is it hosted? What's the stack?
</step>

<step name="static_sites">
**Step 2a: Static Sites / SPAs**

**Recommended:**
- **Cloudflare Pages** (free) - Deploy from Git, automatic builds
- **R2** (free tier) - Store assets, images
- **Cache Everything** - Maximize CDN caching

**Optional:**
- **Turnstile** - Bot protection without CAPTCHAs
- **Web Analytics** - Privacy-focused analytics (free)

**Setup:**
1. Connect GitHub repo to Pages
2. Configure build command and output directory
3. Add custom domain
4. Enable "Cache Everything" page rule
</step>

<step name="apis_backends">
**Step 2b: APIs / Backends**

**Recommended:**
- **Workers** (free tier: 100k req/day) - API endpoints, middleware
- **D1** (free tier) - SQLite database for data storage
- **KV** (free tier) - Key-value for sessions, cache
- **Queues** - Background job processing

**Optional:**
- **Rate Limiting** - Protect API endpoints
- **API Shield** - API-specific security
- **Durable Objects** - Stateful Workers for real-time

**Setup:**
1. Create Worker for API logic
2. Bind D1 database for persistence
3. Use KV for caching/sessions
4. Add rate limiting rules
</step>

<step name="full_stack">
**Step 2c: Full-Stack Applications**

**Recommended:**
- **Pages** - Frontend hosting
- **Workers** - Backend API
- **D1** - Database
- **R2** - File uploads, media storage
- **SSL Strict** - End-to-end encryption

**Optional:**
- **Access** - Add authentication layer
- **Tunnel** - Secure connection to origin
- **Waiting Room** - Handle traffic spikes

**Setup:**
1. Deploy frontend to Pages
2. Create Workers for API routes
3. Set up D1 for database
4. Configure R2 for uploads
5. Enable Access for protected routes
</step>

<step name="ecommerce">
**Step 2d: E-commerce Sites**

**Recommended:**
- **SSL Strict** - Protect transactions
- **WAF** - Block common attacks
- **Rate Limiting** - Prevent abuse
- **Bot Management** - Block scrapers, credential stuffers
- **Cache Rules** - Cache static, bypass cart/checkout

**Optional:**
- **Argo Smart Routing** (paid) - Faster global routing
- **Load Balancing** (paid) - Multiple origins
- **Waiting Room** - Flash sale traffic

**Security checklist:**
1. SSL mode: Strict
2. Always Use HTTPS: On
3. Min TLS: 1.2
4. WAF: Enabled
5. Rate limiting on login/checkout
</step>

<step name="internal_tools">
**Step 2e: Internal Tools / Dashboards**

**Recommended:**
- **Access** (free for 50 users) - Zero Trust authentication
- **Tunnel** - Expose internal services without public IP
- **Gateway** - DNS filtering, malware blocking

**Setup:**
1. Create Access application
2. Configure identity provider (Google, GitHub, etc.)
3. Set up Tunnel to internal service
4. Add Access policies (who can access)
</step>

<step name="high_traffic">
**Step 2f: High Traffic Sites**

**Recommended:**
- **Cache Everything** - Minimize origin load
- **Tiered Cache** (Enterprise) - Regional caching
- **Argo Smart Routing** (paid) - Optimized routing
- **Load Balancing** (paid) - Distribute across origins
- **Waiting Room** - Queue during spikes

**Performance checklist:**
1. Cache level: Aggressive
2. Browser Cache TTL: High
3. Minification: All enabled
4. Brotli: Enabled
5. Early Hints: Enabled
6. HTTP/3: Enabled
</step>

<step name="provide_recommendation">
**Step 3: Provide Recommendation**

Format:

**Recommended Stack for [Project Type]**

**Essential (free):**
- [Service 1] - [Why]
- [Service 2] - [Why]

**Recommended (free tier available):**
- [Service 3] - [Why]
- [Service 4] - [Why]

**Optional (paid):**
- [Service 5] - [Why] - $X/month

**Configuration Checklist:**
1. [ ] Step 1
2. [ ] Step 2
3. [ ] Step 3

**Estimated monthly cost:** $X (or Free)
</step>

</process>

<service_matrix>
| Use Case | Pages | Workers | D1 | R2 | KV | Access | Tunnel |
|----------|-------|---------|----|----|----|---------| --------|
| Static Site | ✓ | | | ✓ | | | |
| SPA | ✓ | | | ✓ | | | |
| API | | ✓ | ✓ | | ✓ | | |
| Full-Stack | ✓ | ✓ | ✓ | ✓ | ✓ | ○ | |
| E-commerce | | | | ✓ | | | |
| Internal Tool | | | | | | ✓ | ✓ |
| Blog | ✓ | | | | | | |

✓ = Recommended, ○ = Optional
</service_matrix>

<free_tier_limits>
**Free Tier Limits (as of 2025):**
- **Workers:** 100,000 requests/day
- **Pages:** Unlimited sites, 500 builds/month
- **D1:** 5GB storage, 5M rows read/day
- **R2:** 10GB storage, 10M Class A ops/month
- **KV:** 100,000 reads/day, 1,000 writes/day
- **Access:** 50 users
- **Tunnel:** Unlimited
</free_tier_limits>

<success_criteria>
Recommendation is complete when:
- [ ] Project type understood
- [ ] Requirements gathered
- [ ] Appropriate services identified
- [ ] Free vs paid options explained
- [ ] Configuration checklist provided
- [ ] User understands next steps
</success_criteria>
