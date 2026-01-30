# TLS Termination vs TLS Passthrough — And the Risky Middle Ground

When you deploy a proxy, load balancer, API gateway, or ingress in front of services, you’re making a decision that is far more than technical:

> **Where does TLS actually end?**

That single choice defines:

* Your **trust boundaries**
* Your **security model**
* Your **routing capabilities**
* Your **observability**
* Your **compliance posture**
* Your **blast radius during an incident**

Most discussions frame this as:

> TLS Termination **vs** TLS Passthrough

But in real systems, there’s a third pattern that appears constantly:

> **TLS termination at the proxy, then plaintext internally**

Let’s walk through all three — and, more importantly, the *thinking process* behind choosing.

---

# 1️⃣ The Two “Clean” Models

## 🔒 TLS Passthrough (End-to-End TLS)

The proxy does **not** decrypt traffic.

**Client → Backend = one TLS session**

The proxy forwards encrypted bytes. It operates mostly at **Layer 4 (TCP)**.

### What the proxy can see:

* IP/port
* TLS handshake metadata (like SNI hostname)
* Connection stats

### What it cannot see:

* URLs
* Headers
* Payload
* HTTP status codes

This is a **secure pipe**, not an application-aware component.

## 🔓 TLS Termination (Edge Termination / Offload)

The proxy **decrypts** TLS from the client.

Now you have two legs:

1. **Client → Proxy**: TLS
2. **Proxy → Backend**:

   * HTTP (plaintext), or
   * New TLS session (re-encryption)

The proxy now operates at **Layer 7** and understands the application protocol.

It’s no longer “just networking” — it becomes part of your **application platform**.

---

# 2️⃣ The Real Architectural Question

This decision is not “performance vs security.”

It is:

> **Do you want your proxy to be a transport device or an application control point?**

| Role of Proxy            | You Should Lean Toward |
| ------------------------ | ---------------------- |
| Network plumbing         | TLS Passthrough        |
| Security + routing brain | TLS Termination        |

---

# 3️⃣ What You Gain with TLS Termination

Once TLS ends at the proxy, it can see HTTP/gRPC/etc. That unlocks major capabilities.

## 🧠 Smart Routing

* Path-based routing (`/api`, `/admin`)
* Header-based routing
* Canary deployments
* A/B testing
* API versioning

Passthrough cannot do this.

## 🛡 Application Security at the Edge

* WAF
* Bot filtering
* Rate limiting per endpoint or user
* JWT/OAuth validation
* mTLS client cert validation
* Request normalization

With passthrough, the proxy is blind to attacks at the HTTP layer.

## 📊 Observability

Termination enables:

* Per-endpoint latency
* HTTP status codes
* Error rates
* Trace header injection
* Detailed logs

Passthrough gives you TCP metrics, not application visibility.

## ⚙ Traffic Optimization

* Compression
* Caching
* Header rewrites
* Connection pooling to backends

Termination turns your proxy into a **smart application-aware edge**.

---

# 4️⃣ What You Give Up When You Terminate

### ❗ The proxy becomes a high-value target

It now sees decrypted:

* Credentials
* Tokens
* Personal data
* Business payloads

Compromise the proxy, and you expose everything.

### ❗ More operational responsibility

You now manage:

* Certificates at the proxy
* PKI integration
* mTLS to backends (if used)

Your proxy is now part of your **security infrastructure**, not just networking.

---

# 5️⃣ What You Gain with TLS Passthrough

## 🔐 True End-to-End Encryption

Only the client and service can see the data.

## 🧱 Reduced Trust in Infrastructure

The proxy never sees plaintext.

## 📜 Simpler compliance story

“Traffic is encrypted in transit” is easier when no middlebox decrypts it.

## 🧬 Direct identity

With mTLS, the backend sees the real client cert — not a proxy-asserted identity.

---

# 6️⃣ What You Lose with Passthrough

| Capability             | Passthrough |
| ---------------------- | ----------- |
| Path-based routing     | ❌           |
| WAF                    | ❌           |
| JWT validation at edge | ❌           |
| HTTP metrics           | ❌           |
| API gateway behavior   | ❌           |

Your proxy becomes secure, but **blind**.

---

# 7️⃣ The Middle Ground: Termination + Plaintext Internal

This is extremely common:

**Client → Proxy: TLS**
**Proxy → Backend: HTTP**

It’s simple. Backends don’t need certs. Historically it improved performance.

But this model **moves your trust boundary**.

You’ve now said:

> “Inside the network, traffic doesn’t need encryption.”

That assumption is increasingly unsafe.

## 🚨 Risks of Plaintext Internal Traffic

### 1. Lateral movement = data exposure

If an attacker compromises:

* A VM
* A container
* A pod
* A monitoring agent

They may read internal traffic:

* JWTs
* Session cookies
* API keys
* PII
* Payment data

TLS protected you from the internet — not from the breach after entry.

### 2. Header spoofing risk

Backends often trust proxy headers:

* `X-Forwarded-For`
* `X-Forwarded-Proto`
* Identity headers

If services are reachable internally via HTTP, an attacker can call them directly and spoof headers unless tightly restricted.

### 3. Weak zero-trust posture

Modern security assumes:

> The internal network is hostile.

Plaintext internal traffic contradicts that assumption.

### 4. Compliance pressure

Auditors increasingly ask:

> “Is traffic encrypted between internal services?”

“It's internal” is no longer a strong answer.

---

# 8️⃣ When Plaintext Internally *Might* Be Acceptable

Only with strong context:

* Backend services are only reachable from the proxy
* Strict network isolation
* Low sensitivity data
* Transitional architecture
* Compensating controls (firewalls, header stripping, monitoring)

Even then, it’s a **risk acceptance**, not best practice.

---

# 9️⃣ The Modern Default Pattern

Most modern systems choose:

### 🔐 TLS Termination + Re-encrypt

Client → Proxy: TLS
Proxy → Backend: TLS

You get:

* L7 control
* WAF and routing
* Observability
* Encryption in transit internally

### 🔐 Even better: mTLS internally

Proxy → Backend uses mutual TLS. Backend only accepts trusted clients.

This protects against:

* Lateral movement
* Rogue workloads
* Header spoofing

---

# 🔟 Final Decision Framework

| Requirement                       | Best Fit                            |
| --------------------------------- | ----------------------------------- |
| L7 routing/security/observability | TLS Termination                     |
| Strict end-to-end confidentiality | TLS Passthrough                     |
| Modern secure platform            | Terminate + Re-encrypt (often mTLS) |
| Legacy/simple internal network    | Terminate + Plaintext (with risk)   |

---

# Final Takeaway

TLS termination gives you **power and visibility**.
TLS passthrough gives you **isolation and simplicity**.

Termination + plaintext internally gives you **convenience**, but weakens internal security and should be justified, not assumed.

If the data would be a major incident if leaked,
**don’t let it travel unencrypted — even inside your network.**

Your TLS architecture is really your **trust architecture**.
