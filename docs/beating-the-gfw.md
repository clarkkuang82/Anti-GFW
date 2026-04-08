# Beating the Great Firewall of China

## How the GFW Works

The Great Firewall (GFW) uses multiple layers of detection and blocking to restrict internet access. Understanding these mechanisms is essential to circumventing them.

### Detection Methods

| Method | Description |
|--------|-------------|
| **DNS Poisoning** | Returns fake DNS responses for blocked domains, redirecting or failing lookups |
| **IP Blacklisting** | Blocks traffic to known IP addresses of banned services |
| **Deep Packet Inspection (DPI)** | Inspects packet contents and metadata to identify protocol signatures (e.g., VPN, Tor) |
| **SNI Filtering** | Reads the Server Name Indication field in TLS handshakes to identify target domains |
| **Active Probing** | GFW sends its own connections to suspicious servers to fingerprint and confirm circumvention tools |
| **TCP/IP Header Analysis** | Detects anomalies in packet structure that indicate tunneling |
| **Bandwidth Throttling** | Slows connections that exhibit encrypted tunnel characteristics |
| **Connection Reset (RST injection)** | Injects TCP RST packets to tear down detected connections |

## Circumvention Strategies

### 1. Encrypted Proxy Protocols

Traditional VPNs (OpenVPN, IPSec) are easily detected by DPI because they have recognizable handshake patterns. Modern proxy protocols are designed to look like normal HTTPS traffic.

#### Shadowsocks

- Lightweight SOCKS5 proxy with encrypted traffic
- Uses symmetric encryption (AES-256-GCM, ChaCha20-Poly1305)
- Traffic looks like random bytes — no identifiable header
- Widely used but partially detected via traffic analysis (entropy detection, packet length patterns)

```
Client <--encrypted--> Shadowsocks Server (outside GFW) <--plain--> Internet
```

#### ShadowsocksR (SSR)

- Fork of Shadowsocks with added obfuscation plugins
- Protocol obfuscation makes traffic resemble HTTP or TLS
- Largely superseded by newer tools

#### V2Ray / Xray

- Multi-protocol platform supporting VMess, VLESS, Trojan, and more
- **VMess**: Encrypted protocol with UUID-based authentication
- **VLESS**: Lightweight version of VMess with less overhead
- Supports multiple transports: WebSocket, gRPC, HTTP/2, QUIC
- Can be fronted behind CDNs (Cloudflare) for additional protection

#### Trojan

- Designed to be indistinguishable from normal HTTPS (TLS 1.3) traffic
- Runs on port 443 and serves a real HTTPS website to non-authenticated visitors
- GFW active probes see a legitimate website; only clients with the correct password get proxy access
- Very resistant to active probing

```
Client --TLS 1.3--> Trojan Server (looks like normal HTTPS)
                     |-- authenticated --> proxy to internet
                     |-- unauthenticated --> serve real website
```

#### Hysteria / Hysteria2

- Based on QUIC (UDP) protocol
- Achieves high throughput by using a custom congestion control (Brutal)
- Effective against TCP-focused throttling
- Hysteria2 uses standard HTTP/3 authentication

### 2. Traffic Obfuscation

The goal is to make circumvention traffic indistinguishable from allowed traffic.

#### Domain Fronting

- Uses CDN infrastructure (e.g., Cloudflare, AWS CloudFront) to hide the true destination
- The outer TLS SNI shows a legitimate CDN domain; the inner HTTP Host header routes to the proxy
- GFW would have to block the entire CDN to stop it (collateral damage)
- Some CDN providers have restricted this technique

#### WebSocket/gRPC over CDN

- V2Ray/Xray can transport proxy traffic over WebSocket or gRPC
- Placing this behind a CDN like Cloudflare hides the origin server IP
- Traffic appears as normal HTTPS to a CDN — very hard to block without blocking the CDN entirely

#### TLS Camouflage

- Tools like **uTLS** mimic the TLS fingerprint of popular browsers (Chrome, Firefox, Safari)
- Prevents the GFW from distinguishing proxy TLS connections from normal browser traffic
- Used by Xray (XTLS), Clash, and others

#### Pluggable Transports (originally from Tor)

- **obfs4**: Makes traffic look like random noise
- **meek**: Tunnels through cloud provider frontends
- **Snowflake**: Uses WebRTC peer-to-peer connections through volunteer browsers

### 3. DNS Circumvention

Since DNS poisoning is the first line of blocking:

#### DNS over HTTPS (DoH)

- Encrypts DNS queries inside HTTPS requests
- Providers: Cloudflare (1.1.1.1), Google (8.8.8.8), Quad9
- GFW cannot see the DNS query contents

#### DNS over TLS (DoT)

- Encrypts DNS on port 853
- Easier for GFW to block the port, so DoH is generally preferred

#### Encrypted Client Hello (ECH)

- Encrypts the SNI field in TLS handshakes (previously visible in plaintext)
- Prevents SNI-based filtering
- Requires both client and server support
- Still being deployed; Cloudflare supports it

### 4. Multi-Hop and Relay Architectures

#### Relay Chains

```
Client --> Relay (inside China) --> Proxy Server (outside China) --> Internet
```

- Use a domestic relay server to forward traffic to the actual proxy abroad
- The domestic relay sees only encrypted traffic
- Adds latency but increases resilience

#### Tor with Bridges

- Standard Tor is blocked; entry node IPs are known
- **Bridges** are unlisted Tor entry nodes
- Combined with pluggable transports (obfs4, Snowflake, meek) for additional obfuscation
- Slow but provides strong anonymity

### 5. Emerging Approaches

#### REALITY (Xray)

- Next generation of TLS camouflage
- Borrows the TLS certificate of a legitimate website without needing to own it
- Server presents a real site's certificate during handshake; GFW probes see a genuine TLS connection
- Client authenticates via a short ID and public key
- No need for your own domain or certificate — very low setup cost

#### Encrypted Client Hello (ECH) + CDN

- When fully deployed, ECH will encrypt the SNI, making domain-based blocking impossible without blocking entire IP ranges
- Combined with CDN hosting, this could make per-site blocking infeasible

#### QUIC/HTTP3-based Tunneling

- UDP-based protocols are harder for middleboxes to inspect and reset
- Hysteria2 and tuic exploit this

#### Satellite Internet

- Starlink and similar LEO satellite constellations bypass terrestrial infrastructure entirely
- Not dependent on undersea cables or border routers
- Currently restricted in China but relevant as the technology spreads

## Architecture Recommendations

### For Individual Users

```
Recommended Stack:
  Xray (VLESS + REALITY) or Trojan-Go
  + CDN fronting (Cloudflare)
  + DoH for DNS
  + uTLS fingerprint mimicry
```

- Use VLESS + REALITY on Xray for the strongest anti-detection
- Fall back to Trojan behind Cloudflare CDN if REALITY is blocked
- Always use encrypted DNS (DoH)
- Keep client software updated — GFW detection evolves constantly

### For Developers Building Tools

1. **Assume active probing** — your server must respond like a normal web server to unauthenticated requests
2. **Avoid protocol fingerprints** — no fixed magic bytes, predictable packet sizes, or identifiable handshakes
3. **Support multiple transports** — if one transport is blocked, users can switch to another
4. **Mimic real browser TLS** — use uTLS or equivalent to match browser fingerprints
5. **Leverage existing infrastructure** — CDNs, cloud functions, and websockets are hard to block without collateral damage
6. **Rotate IPs and domains** — assume any single IP or domain will eventually be discovered
7. **Minimize metadata leakage** — packet sizes, timing patterns, and connection frequency can all be analyzed

### Server Deployment Checklist

- [ ] Server runs on port 443 (standard HTTPS)
- [ ] Serves a real website to unauthenticated visitors
- [ ] TLS certificate is valid (Let's Encrypt or REALITY)
- [ ] TLS fingerprint matches a common browser
- [ ] DNS resolves through encrypted channel
- [ ] IP is not on known blacklists (test with a Chinese IP)
- [ ] Fallback transport configured (e.g., WebSocket + CDN)
- [ ] Regular IP rotation or CDN fronting in place

## Tool Comparison

| Tool | Protocol | Anti-DPI | Anti-Active-Probing | Speed | Setup Difficulty |
|------|----------|----------|---------------------|-------|-----------------|
| Shadowsocks | Custom | Medium | Low | Fast | Easy |
| V2Ray (VMess) | VMess | High | Medium | Fast | Medium |
| Xray (VLESS+REALITY) | VLESS | Very High | Very High | Fast | Medium |
| Trojan-Go | Trojan | Very High | Very High | Fast | Medium |
| Hysteria2 | QUIC | High | High | Very Fast | Easy |
| Tor + obfs4 | Tor | High | High | Slow | Easy |
| Naive Proxy | HTTP/2 | Very High | High | Fast | Medium |

## Key Principles

1. **Defense in depth** — combine multiple techniques (encrypted DNS + TLS camouflage + CDN fronting)
2. **Look like everyone else** — the best camouflage is traffic that looks exactly like millions of other HTTPS connections
3. **Assume compromise** — any single technique will eventually be detected; always have fallbacks
4. **Minimize attack surface** — the less your server reveals about itself, the harder it is to fingerprint
5. **Stay current** — the GFW is actively researched and updated; circumvention tools must evolve continuously
