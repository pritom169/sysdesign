## CDN Interview Checklist


Quick reference for system design interviews involving CDN.

| Topic | Key Points to Mention |
|-------|----------------------|
| **What is CDN** | Geographically distributed cache, reduces latency, offloads origin |
| **Origin vs Edge** | Origin = source of truth; Edge = cached copies near users |
| **Push vs Pull** | Pull = on-demand (websites); Push = proactive (video, downloads) |
| **Cache Invalidation** | TTL, purge API, versioned URLs (best practice) |
| **Cache Shield** | Mid-tier cache protecting origin from thundering herd |
| **When NOT to use** | Real-time data, personalized content, write operations |

### Common Interview Questions

**"When would you use a CDN?"**
- Static assets (images, CSS, JS)
- Global user base requiring low latency
- High read-to-write ratio content
- Video streaming or large file downloads
- Protecting origin from traffic spikes

**"How do you handle cache invalidation?"**
- Versioned URLs for static assets (`app.a1b2c3.js`)
- Short TTL + purge API for dynamic content
- Cache shield to prevent origin stampede
- Accept eventual consistency for non-critical content

**"Push vs Pull CDN—when to use which?"**
- Pull: Unknown access patterns, storage cost sensitive, frequently changing
- Push: Predictable content, latency critical, origin unreliable
- Hybrid: Most real systems use both based on content type

**"Design a video streaming system"**
- Push CDN for video segments (pre-positioned)
- Adaptive bitrate streaming (HLS/DASH)
- Regional cache shields
- Origin in object storage (S3/GCS)
- Consider: encoding pipeline, DRM, live vs VOD

**"How would you reduce CDN costs?"**
- Increase cache hit ratio (longer TTL, better cache keys)
- Compress content (save 60-80% bandwidth)
- Use cache shield (reduce origin egress)
- Committed use discounts with CDN provider
- Monitor for cache-busting query strings

### CDN Selection Criteria

| Factor | Considerations |
|--------|----------------|
| **PoP Coverage** | Where are your users? Match CDN's edge locations |
| **Features** | Edge compute, video optimization, security features |
| **Pricing Model** | Bandwidth vs requests, egress costs, commit discounts |
| **Origin Support** | S3, custom origin, multi-origin failover |
| **Invalidation** | Purge speed, API availability, cost per purge |
| **Analytics** | Real-time logs, hit ratio metrics, latency data |

### Popular CDN Providers

| Provider | Strengths | Best For |
|----------|-----------|----------|
| **Cloudflare** | Security, free tier, edge compute | General purpose, security-focused |
| **CloudFront** | AWS integration, Lambda@Edge | AWS-native applications |
| **Akamai** | Largest network, enterprise features | Enterprise, media streaming |
| **Fastly** | Real-time purge, VCL customization | Dynamic content, API acceleration |
| **Google Cloud CDN** | GCP integration, global load balancing | GCP-native applications |

---

