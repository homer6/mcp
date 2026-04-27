The `.well-known` standard is thoroughly documented. Here's a comprehensive overview:

***

## What Is `.well-known`?

The `/.well-known/` URI prefix is a standardized path reserved at the root of any web origin — for example, `https://example.com/.well-known/` — that gives clients a predictable, consistent location to find site-wide metadata, service configuration, and policy information. Before this standard existed, every protocol had to invent its own arbitrary paths, leading to inconsistency, naming conflicts, and fragile discovery mechanisms. [en.wikipedia](https://en.wikipedia.org/wiki/Well-known_URI)

## Origin and History

The standard was first codified as **RFC 5785** in April 2010, authored by Mark Nottingham and Eran Hammer-Lahav. It was subsequently revised and replaced by **RFC 8615** in May 2019, also authored by Nottingham, which obsoletes RFC 5785 and expanded support to more URI schemes. The IANA Well-Known URIs registry was created on 2010-01-20 and is continuously updated — it was last updated on 2026-04-01. [datatracker.ietf](https://datatracker.ietf.org/doc/rfc8615/)

## How It Works

A well-known URI takes the form `/.well-known/<suffix>`, where the suffix is a short, descriptive name registered in the IANA registry. Crucially, it only exists at the **root of an origin** — `/blog/.well-known/security.txt` is not a valid well-known URI, only `/.well-known/security.txt` is. No default content is defined at `/.well-known/` itself; every sub-path must be explicitly registered and defined by a specification. [http](https://http.dev/well-known-uris)

## Supported URI Schemes

The mechanism is not universal across all URI schemes; a scheme must explicitly opt in. Currently supported schemes include: [http](https://http.dev/well-known-uris)

- **http / https** — the original and primary target
- **ws / wss** — WebSocket schemes for WebSocket-related metadata
- **coap / coaps** (and TCP/WebSocket variants) — the Constrained Application Protocol used in IoT devices [http](https://http.dev/well-known-uris)

## Registration Process

All well-known URI suffixes are managed by the **IANA Well-Known URIs Registry**. Adding a new entry follows a "Specification Required" review process, evaluated by designated experts (currently Mark Nottingham). Each registration must include: [iana](https://www.iana.org/assignments/well-known-uris)

- **URI suffix** — the path segment after `/.well-known/` (no forward slashes allowed, must conform to the `segment-nz` URI syntax production)
- **Change controller** — the responsible organization or individual
- **Reference** — the specification document defining the resource format
- **Status** — either *permanent* (stable specification, widely implemented) or *provisional* (less established, may be removed if unused) [http](https://http.dev/well-known-uris)

## Key Well-Known URIs by Category

### Security & Trust
| Suffix | Purpose |
|---|---|
| `acme-challenge` | ACME protocol domain validation for TLS certificates (e.g., Let's Encrypt)  [docs.20i](https://docs.20i.com/web-hosting/what-is-well-known) |
| `pki-validation` | HTTP-based domain validation for SSL certificate issuance  [docs.20i](https://docs.20i.com/web-hosting/what-is-well-known) |
| `security.txt` | Vulnerability disclosure policy with contact info, per RFC 9116  [http](https://http.dev/well-known-uris) |
| `mta-sts.txt` | Mail Transport Agent Strict Transport Security policy for SMTP TLS enforcement  [http](https://http.dev/well-known-uris) |
| `gpc.json` | Global Privacy Control signal indicating whether the site respects opt-out preferences  [http](https://http.dev/well-known-uris) |

### Identity & Authentication
| Suffix | Purpose |
|---|---|
| `openid-configuration` | OpenID Connect discovery document: lists authorization endpoints, supported scopes, signing algorithms  [help.akana](https://help.akana.com/content/current/cm/api_oauth/oauth_discovery/m_oauth_getOpenIdConnectWellknownConfiguration.htm) |
| `oauth-authorization-server` | OAuth 2.0 authorization server metadata (equivalent to `openid-configuration` for non-OIDC OAuth servers)  [http](https://http.dev/well-known-uris) |
| `webauthn` | Related origins list allowing passkeys registered on one origin to work across related origins  [iana](https://www.iana.org/assignments/well-known-uris) |
| `change-password` | Redirects to the site's actual password change page (used by password managers)  [http](https://http.dev/well-known-uris) |
| `webfinger` | Federated account discovery — accepts a resource query and returns a JSON Resource Descriptor  [http](https://http.dev/well-known-uris) |

### Mobile App Linking
| Suffix | Purpose |
|---|---|
| `assetlinks.json` | Android App Links (Google Digital Asset Links) — declares web/app associations for verified deep links  [iana](https://www.iana.org/assignments/well-known-uris) |
| `apple-app-site-association` | iOS Universal Links — associates a domain with Apple apps (not IANA-registered, but widely used)  [http](https://http.dev/well-known-uris) |

### Federation & Fediverse
| Suffix | Purpose |
|---|---|
| `nodeinfo` | Fediverse server metadata (software, protocols, usage stats for Mastodon, etc.)  [http](https://http.dev/well-known-uris) |
| `matrix` | Matrix federation server discovery  [iana](https://www.iana.org/assignments/well-known-uris) |
| `host-meta` / `host-meta.json` | Web Host Metadata bootstrap for WebFinger and other discovery protocols  [iana](https://www.iana.org/assignments/well-known-uris) |
| `api-catalog` | API discovery endpoint returning available APIs and documentation links (RFC 9727)  [iana](https://www.iana.org/assignments/well-known-uris) |

### IoT, Protocols & Miscellaneous
| Suffix | Purpose |
|---|---|
| `caldav` / `carddav` | CalDAV/CardDAV service discovery (calendar and contacts)  [iana](https://www.iana.org/assignments/well-known-uris) |
| `sbom` | Software Bill of Materials endpoint  [iana](https://www.iana.org/assignments/well-known-uris) |
| `did.json` | Decentralized Identifiers (DID) via the DID Web method  [iana](https://www.iana.org/assignments/well-known-uris) |
| `wot` | W3C Web of Things (IoT device discovery)  [iana](https://www.iana.org/assignments/well-known-uris) |
| `terraform.json` | HashiCorp Terraform remote service discovery  [iana](https://www.iana.org/assignments/well-known-uris) |
| `keybase.txt` | Keybase identity verification file  [iana](https://www.iana.org/assignments/well-known-uris) |

The IANA registry currently contains **over 100 registered entries**. [http](https://http.dev/well-known-uris)

## A Real-World Example: OpenID Connect Discovery

When an app needs to authenticate users via Google, it performs a `GET` request to `https://accounts.google.com/.well-known/openid-configuration`. The response is a JSON document listing every endpoint the OAuth/OIDC client needs — authorization, token exchange, user info, and the JWKS key set for signature verification — eliminating the need for hardcoded configuration. [authgear](https://www.authgear.com/post/well-known-openid-configuration)

## Security Considerations

Because `.well-known` paths are publicly accessible and routinely probed by automated scanners, several risks must be managed: [http](https://http.dev/well-known-uris)

- **Access control on shared hosting**: On multi-tenant hosts, write access to `/.well-known/` must be tightly restricted, as a malicious tenant could hijack certificate validation or publish fraudulent app-association files. [http](https://http.dev/well-known-uris)
- **Information disclosure**: Anything placed here is effectively public; sensitive data must never be served without encryption or access controls. [http](https://http.dev/well-known-uris)
- **Scope ambiguity**: A `.well-known` resource at `example.com` does not automatically apply to `sub.example.com`; each specification defines its own scoping rules. [http](https://http.dev/well-known-uris)
- **Content-Type discipline**: Resources should be served as `application/json` or `text/plain` (not `text/html`), and the `X-Content-Type-Options: nosniff` header should be sent to prevent MIME sniffing attacks. [http](https://http.dev/well-known-uris)
- **No redirects for some URIs**: `apple-app-site-association` and `assetlinks.json` must be served directly without redirects, or the verifying OS client (iOS/Android) will reject them. [http](https://http.dev/well-known-uris)

## Implementation

Serving a well-known file is straightforward — create a `.well-known/` directory in your web root and place the appropriate file(s) inside. For dynamic resources like `openid-configuration`, configure a route handler in your web framework. Many hosting environments and automated tools (like Certbot for Let's Encrypt) create and manage the directory automatically. In Nginx, a static `security.txt` can be served with: [servebolt](https://servebolt.com/help/technical-resources/what-the-well-known-folder-is-used-for/)

```nginx
location = /.well-known/security.txt {
    alias /var/www/example.com/.well-known/security.txt;
    default_type text/plain;
}
```

Applications behind a reverse proxy should ensure the proxy passes `/.well-known/` paths through to the backend rather than blocking or returning a 404. [http](https://http.dev/well-known-uris)

## Notable Trivia

`robots.txt`, `sitemap.xml`, `favicon.ico`, and `humans.txt` all live at the site root but are **not** well-known URIs — they predate the standard and are not part of the IANA registry. The well-known mechanism is specifically for machine-readable metadata that needs a predictable, standardized discovery location across any server. [http](https://http.dev/well-known-uris)