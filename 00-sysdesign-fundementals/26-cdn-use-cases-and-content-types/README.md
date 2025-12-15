## CDN Use Cases and Content Types


Understanding what content belongs on a CDN—and what doesn't—is crucial for system design interviews.

### Ideal CDN Content

| Content Type | Why CDN Excels | Example |
|-------------|----------------|---------|
| **Static Assets** | Immutable, highly cacheable | CSS, JS, images, fonts |
| **Video/Audio** | Large files, high bandwidth | Streaming, podcasts |
| **Software Downloads** | Predictable, large files | App installers, game patches |
| **Documents** | Read-heavy, rarely changes | PDFs, whitepapers |

### Content NOT Suited for CDN

| Content Type | Why CDN Struggles | Better Alternative |
|-------------|-------------------|-------------------|
| **Real-time Data** | Changes constantly | WebSocket, direct API |
| **Personalized Content** | Unique per user, uncacheable | Application server |
| **Transactional APIs** | Write operations, state changes | Origin server |
| **Sensitive Data** | Security/compliance concerns | Private network |

### Common CDN Use Cases

**1. Static Website Hosting**
```
User → CDN Edge → (HTML, CSS, JS, Images)
```
- Entire site served from edge
- Origin only needed for deployments
- Example: Marketing sites, documentation

**2. API Acceleration**
```
User → CDN Edge → (cached GET responses) or → Origin (POST/PUT)
```
- Cache GET requests with appropriate TTL
- Pass through mutations to origin
- Example: Product catalogs, public APIs

**3. Video Streaming**
```
User → CDN Edge → (video chunks/segments)
```
- Pre-positioned or pull-cached video files
- Adaptive bitrate streaming (HLS/DASH)
- Example: Netflix, YouTube, Twitch

**4. Software Distribution**
```
User → CDN Edge → (installers, updates, patches)
```
- Push model for predictable releases
- Critical for launch day traffic spikes
- Example: Steam, Windows Update, App Stores

---

