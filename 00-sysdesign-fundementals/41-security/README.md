## Security


### What is Security and Privacy?

**Security** protects systems from unauthorized access and attacks. **Privacy** ensures data is handled according to user expectations and regulations.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Security Pillars (CIA Triad)                  │
└─────────────────────────────────────────────────────────────────┘

        ┌─────────────────┐
        │ Confidentiality │  ← Only authorized access
        └────────┬────────┘
                 │
    ┌────────────┴────────────┐
    │                         │
    ▼                         ▼
┌─────────┐             ┌───────────┐
│Integrity│             │Availability│
└─────────┘             └───────────┘
    ↑                         ↑
    │                         │
Data not                 System accessible
tampered                 when needed
```

---

### What is Authentication?

**Authentication** verifies the identity of a user or system ("Who are you?").

```
┌─────────────────────────────────────────────────────────────────┐
│                    Authentication Flow                           │
└─────────────────────────────────────────────────────────────────┘

User                    Server                    Database
  │                        │                          │
  │── Username/Password ─▶│                          │
  │                        │── Lookup user ─────────▶│
  │                        │◀── Hashed password ─────│
  │                        │                          │
  │                        │ Hash(input) == stored?  │
  │                        │                          │
  │◀─── Auth token ───────│  (if match)              │
  │                        │                          │
```

**Authentication Factors:**

| Factor | Type | Example |
|--------|------|---------|
| **Something you know** | Knowledge | Password, PIN |
| **Something you have** | Possession | Phone, hardware key |
| **Something you are** | Inherence | Fingerprint, face |

**Multi-Factor Authentication (MFA):** Combines 2+ factors.

---

### What is Authorization?

**Authorization** determines what an authenticated user can do ("What can you access?").

```
┌─────────────────────────────────────────────────────────────────┐
│                    Authorization Flow                            │
└─────────────────────────────────────────────────────────────────┘

User (authenticated)              Server
        │                            │
        │── Request: DELETE /users ─▶│
        │                            │
        │                    Check permissions:
        │                    ┌──────────────────────┐
        │                    │ User role: "editor"  │
        │                    │ Required: "admin"    │
        │                    │ "editor" ⊄ "admin"   │
        │                    └──────────────────────┘
        │                            │
        │◀── 403 Forbidden ─────────│
        │                            │
```

**Common Authorization Models:**

| Model | Description | Use Case |
|-------|-------------|----------|
| **RBAC** | Role-Based Access Control | Most applications |
| **ABAC** | Attribute-Based (user, resource, context) | Complex policies |
| **ACL** | Access Control Lists per resource | File systems |
| **ReBAC** | Relationship-Based | Social networks |

---

### Authentication vs. Authorization

| Aspect | Authentication | Authorization |
|--------|----------------|---------------|
| **Question** | Who are you? | What can you do? |
| **When** | First (login) | After authentication |
| **Mechanism** | Passwords, tokens, biometrics | Roles, permissions, policies |
| **HTTP Code** | 401 Unauthorized | 403 Forbidden |
| **Example** | Login with username/password | Can user delete this file? |

```
                          Request Flow
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Authentication    │
                    │   "Who are you?"    │
                    └──────────┬──────────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
               ✗ Unknown             ✓ Known
                    │                     │
                    ▼                     ▼
            401 Unauthorized     ┌─────────────────────┐
                                 │   Authorization     │
                                 │  "Can you do this?" │
                                 └──────────┬──────────┘
                                            │
                                 ┌──────────┴──────────┐
                                 │                     │
                            ✗ Denied              ✓ Allowed
                                 │                     │
                                 ▼                     ▼
                          403 Forbidden           200 OK
```

---

### OAuth vs. JWT for Authentication

**OAuth 2.0** is an authorization framework. **JWT** is a token format. They serve different purposes but are often used together.

#### OAuth 2.0

```
┌─────────────────────────────────────────────────────────────────┐
│              OAuth 2.0 Authorization Code Flow                   │
└─────────────────────────────────────────────────────────────────┘

User          App              Auth Server         Resource Server
 │             │                    │                     │
 │── Login ──▶│                    │                     │
 │             │── Redirect ──────▶│                     │
 │◀────────────────── Login Page ──│                     │
 │── Credentials ─────────────────▶│                     │
 │◀───────── Authorization Code ───│                     │
 │             │◀── Code ──────────│                     │
 │             │                    │                     │
 │             │── Code + Secret ─▶│                     │
 │             │◀── Access Token ──│                     │
 │             │                    │                     │
 │             │── Request + Token ─────────────────────▶│
 │             │◀── Protected Data ──────────────────────│
 │◀── Data ───│                    │                     │
```

**OAuth 2.0 Roles:**
- **Resource Owner:** User
- **Client:** Application requesting access
- **Authorization Server:** Issues tokens
- **Resource Server:** Holds protected resources

#### JWT (JSON Web Token)

```
JWT Structure:
┌─────────────────────────────────────────────────────────────────┐
│  Header          Payload            Signature                    │
│  (algorithm)     (claims)           (verification)               │
│                                                                  │
│  eyJhbGc...  .  eyJzdWI...   .   SflKxwRJSM...                  │
│  ─────────      ───────────       ─────────────                  │
│  Base64URL      Base64URL         HMAC/RSA                       │
└─────────────────────────────────────────────────────────────────┘

Decoded:
{                          {
  "alg": "HS256",            "sub": "user123",
  "typ": "JWT"               "name": "John",
}                            "exp": 1735689600,
                             "roles": ["admin"]
                           }
```

**JWT Properties:**
- Self-contained (no database lookup needed)
- Stateless (server doesn't store sessions)
- Signed (tamper-evident)
- Optionally encrypted

#### Comparison

| Aspect | OAuth 2.0 | JWT |
|--------|-----------|-----|
| **What it is** | Authorization framework | Token format |
| **Purpose** | Delegated access | Encode claims |
| **Stateful?** | Can be either | Stateless |
| **Revocation** | Easy (invalidate at server) | Hard (until expiry) |
| **Used for** | Third-party access, SSO | Session tokens, API auth |

**Common Pattern:** OAuth issues JWTs as access tokens.

---

### What is Encryption?

**Encryption** transforms data into unreadable format, reversible only with a key.

```
┌─────────────────────────────────────────────────────────────────┐
│                    Symmetric Encryption                          │
└─────────────────────────────────────────────────────────────────┘

Same key for encryption and decryption:

   Plaintext          Key              Ciphertext
  ┌─────────┐     ┌─────────┐        ┌─────────────┐
  │ "Hello" │ ──▶ │ AES-256 │ ────▶  │ "xK9#mP..." │
  └─────────┘     └─────────┘        └─────────────┘
                       │
                  ┌────┴────┐
                  │Same Key │
                  └────┬────┘
                       │
  ┌─────────┐     ┌─────────┐        ┌─────────────┐
  │ "Hello" │ ◀── │ AES-256 │ ◀────  │ "xK9#mP..." │
  └─────────┘     └─────────┘        └─────────────┘
   Plaintext          Key              Ciphertext

Examples: AES, ChaCha20
Use: Data at rest, fast bulk encryption
```

```
┌─────────────────────────────────────────────────────────────────┐
│                   Asymmetric Encryption                          │
└─────────────────────────────────────────────────────────────────┘

Key pair: Public key (encrypt) + Private key (decrypt)

Sender                                          Receiver
┌────────────────────┐                    ┌────────────────────┐
│ Has: Bob's public  │                    │ Has: Private key   │
│      key           │                    │      Public key    │
└─────────┬──────────┘                    └─────────┬──────────┘
          │                                         │
          │ Encrypt with                           │
          │ Bob's public key                       │
          ▼                                         ▼
   ┌─────────────┐                          ┌─────────────┐
   │ Ciphertext  │ ────── Network ────────▶ │ Ciphertext  │
   └─────────────┘                          └──────┬──────┘
                                                   │
                                            Decrypt with
                                            private key
                                                   │
                                                   ▼
                                            ┌───────────┐
                                            │ Plaintext │
                                            └───────────┘

Examples: RSA, ECC
Use: Key exchange, digital signatures
```

**Encryption Types:**

| Type | Key | Speed | Use Case |
|------|-----|-------|----------|
| **Symmetric** | Same key both sides | Fast | Bulk data |
| **Asymmetric** | Public/private pair | Slow | Key exchange, signatures |
| **Hybrid** | Asymmetric for key, symmetric for data | Best of both | TLS/HTTPS |

**Encryption Layers:**

| Layer | Protects | Example |
|-------|----------|---------|
| **In transit** | Network traffic | TLS/HTTPS |
| **At rest** | Stored data | AES disk encryption |
| **End-to-end** | Data even from service provider | Signal, WhatsApp |

---

### What are DDoS Attacks?

**Distributed Denial of Service (DDoS)** overwhelms a system with traffic from many sources, making it unavailable.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DDoS Attack Structure                         │
└─────────────────────────────────────────────────────────────────┘

           Attacker
              │
              │ Commands
              ▼
    ┌─────────────────────┐
    │   Command & Control │
    │       Server        │
    └─────────┬───────────┘
              │
    ┌─────────┼─────────────────────────┐
    │         │         │         │     │
    ▼         ▼         ▼         ▼     ▼
  ┌───┐     ┌───┐     ┌───┐     ┌───┐ ┌───┐
  │Bot│     │Bot│     │Bot│     │Bot│ │Bot│  ← Compromised machines
  └─┬─┘     └─┬─┘     └─┬─┘     └─┬─┘ └─┬─┘
    │         │         │         │     │
    └────┬────┴────┬────┴────┬────┴──┬──┘
         │         │         │       │
         ▼         ▼         ▼       ▼
    ┌─────────────────────────────────────┐
    │            Target Server            │
    │         💀 OVERWHELMED              │
    └─────────────────────────────────────┘
```

**Attack Types:**

| Layer | Attack Type | Method |
|-------|-------------|--------|
| **Network (L3/4)** | Volumetric | UDP flood, ICMP flood |
| **Network (L3/4)** | Protocol | SYN flood, Ping of Death |
| **Application (L7)** | Application | HTTP flood, Slowloris |

**Common Attacks:**

```
SYN Flood:
┌────────┐         ┌────────┐
│Attacker│         │ Target │
└────┬───┘         └────┬───┘
     │                   │
     │── SYN (fake IP) ─▶│
     │                   │── SYN-ACK (to fake IP, no response)
     │── SYN (fake IP) ─▶│
     │                   │── SYN-ACK (waiting...)
     │── SYN (fake IP) ─▶│
     │       ...         │── SYN-ACK (waiting...)
     │                   │
     │  Target's connection table fills up
     │  Legitimate connections rejected
```

**Mitigation Strategies:**

| Strategy | How It Works |
|----------|--------------|
| **Rate limiting** | Cap requests per IP/user |
| **CDN/Edge network** | Absorb traffic at edge, cache responses |
| **Anycast** | Distribute attack across global PoPs |
| **WAF** | Filter malicious L7 traffic |
| **Blackholing** | Drop all traffic to target IP (last resort) |
| **CAPTCHA** | Verify human users |
| **SYN cookies** | Stateless SYN-ACK, no half-open connections |

```
DDoS Mitigation Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Traffic                                                        │
│      │                                                           │
│      ▼                                                           │
│  ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌────────┐  │
│  │   CDN     │───▶│   WAF     │───▶│   Rate    │───▶│ Origin │  │
│  │  (Edge)   │    │           │    │  Limiter  │    │ Server │  │
│  └───────────┘    └───────────┘    └───────────┘    └────────┘  │
│       │                │                 │                       │
│    Absorbs          Filters           Throttles                  │
│    volume           attacks           excess                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---


