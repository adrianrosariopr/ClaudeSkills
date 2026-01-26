<overview>
Performance optimization in Cloudflare: caching, Page Rules, Speed settings, and optimization features.
</overview>

<caching>

<cache_levels>
| Level | Description |
|-------|-------------|
| No Query String | Cache ignores query strings |
| Ignore Query String | Cache based on URL only |
| Standard | Cache with query strings |
| Aggressive | Cache everything possible |
</cache_levels>

<what_gets_cached>
**Default cacheable:**
- Images (jpg, png, gif, webp, svg)
- CSS, JavaScript
- Fonts (woff, woff2, ttf)
- Static files (pdf, zip, etc.)

**Not cached by default:**
- HTML (dynamic by default)
- API responses
- Anything with Set-Cookie
- POST/PUT/DELETE requests
</what_gets_cached>

<cache_headers>
**CF-Cache-Status header values:**
| Status | Meaning |
|--------|---------|
| HIT | Served from cache |
| MISS | Fetched from origin, now cached |
| BYPASS | Cache bypassed |
| DYNAMIC | Not eligible (HTML, API) |
| EXPIRED | Cache expired, refetching |
| STALE | Serving stale while revalidating |
| REVALIDATED | Cache validated with origin |
</cache_headers>

<cache_rules>
**Cache Rules** (recommended over Page Rules):
```
# Cache HTML for static site
Match: hostname eq "example.com"
Cache eligibility: Eligible for cache
Edge TTL: 1 day
Browser TTL: 4 hours
```
</cache_rules>

<api_commands>
```bash
# Purge everything
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"purge_everything":true}'

# Purge specific files
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"files":["https://example.com/file.js","https://example.com/style.css"]}'

# Purge by tag (Enterprise)
curl -X POST "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/purge_cache" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"tags":["product-123"]}'
```
</api_commands>
</caching>

<speed_settings>

<setting name="minification">
**Auto Minify:** Remove whitespace from CSS, JS, HTML.

```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/minify" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":{"css":"on","html":"on","js":"on"}}'
```
</setting>

<setting name="brotli">
**Brotli compression:** Better than gzip, ~15-20% smaller.

```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/brotli" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'
```
</setting>

<setting name="early_hints">
**Early Hints (103):** Preload assets before HTML arrives.

```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/early_hints" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'
```
</setting>

<setting name="http3">
**HTTP/3 (QUIC):** Faster connections, especially on mobile.

```bash
curl -X PATCH "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings/http3" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" \
  -H "Content-Type: application/json" \
  -d '{"value":"on"}'
```
</setting>

<setting name="rocket_loader">
**Rocket Loader:** Defer JavaScript loading (can break some sites).

Use with caution - test thoroughly.
</setting>

<setting name="polish">
**Polish (Pro+):** Image optimization.
- Lossless: No quality loss
- Lossy: Smaller files, slight quality reduction
- WebP: Auto-convert to WebP
</setting>

<setting name="mirage">
**Mirage (Pro+):** Lazy-load images, responsive images.
</setting>

</speed_settings>

<page_rules>
**Note:** Page Rules are being replaced by separate Rules products (Cache Rules, Redirect Rules, etc.)

<common_rules>
```
# Cache Everything for static site
URL: example.com/*
Setting: Cache Level = Cache Everything
Setting: Edge Cache TTL = 1 month

# Bypass cache for admin
URL: example.com/admin/*
Setting: Cache Level = Bypass

# Force HTTPS
URL: http://*example.com/*
Setting: Always Use HTTPS

# Forwarding URL (redirect)
URL: old.example.com/*
Setting: Forwarding URL (301) = https://new.example.com/$1
```
</common_rules>
</page_rules>

<optimization_checklist>
**Free tier optimizations:**
- [ ] SSL mode: Full (Strict)
- [ ] Always Use HTTPS: On
- [ ] Auto Minify: CSS, JS, HTML
- [ ] Brotli: On
- [ ] HTTP/3: On
- [ ] Early Hints: On
- [ ] Browser Cache TTL: 4+ hours
- [ ] Cache Level: Standard or Aggressive
- [ ] 0-RTT Connection Resumption: On

**Pro tier additions:**
- [ ] Polish: Lossy + WebP
- [ ] Mirage: On
- [ ] Argo Smart Routing

**Check current settings:**
```bash
# Get all zone settings
curl -s "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/settings" \
  -H "X-Auth-Email: Arosario@invisionnaire.com" \
  -H "X-Auth-Key: 996056aa77525d4f41f4cd46ef520900ef15f" | jq '.result[] | {id, value}'
```
</optimization_checklist>

<paid_features>

<feature name="argo">
**Argo Smart Routing:** Routes around congestion, 30%+ faster.
~$5/month + $0.10/GB
</feature>

<feature name="tiered_cache">
**Tiered Cache (Enterprise):** Regional caching layer reduces origin load.
</feature>

<feature name="load_balancing">
**Load Balancing:** Distribute traffic across multiple origins.
$5/month base + $0.50/500k DNS queries
</feature>

<feature name="rate_limiting">
**Advanced Rate Limiting:** More rules, more actions.
$0.05 per 10,000 good requests
</feature>

</paid_features>
