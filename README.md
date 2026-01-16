<div align="center">

# Open Media Streaming Specification (/spec/v1.0/omss)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](/spec/v1.0/omss-v1.0.md)
[![Specification](https://img.shields.io/badge/spec-OMSS-orange.svg)](https://github.com/omss-spec/omss-spec)

**A standardized REST API specification for streaming media backends**

[Specification](/spec/v1.0/omss-v1.0.md) · [OpenAPI](/spec/v1.0/omss-openapi.yml) · [Contributing](CONTRIBUTING.md) · [Discussions](https://github.com/omss-spec/omss-spec/discussions)

</div>

---

## 🎯 What is OMSS?

**OMSS (Open Media Streaming Specification)** is an open standard that defines how streaming backends expose movies, TV episodes, sources, and subtitles through a unified REST API.

### The Problem

Every streaming backend has custom API formats:

- ❌ Frontends rewrite integration code for each backend
- ❌ Users can't easily switch backends
- ❌ No interoperability between implementations
- ❌ Fragmented ecosystem slows innovation

### The Solution

**OMSS standardizes** endpoints, request/response formats, and error handling:

- ✅ **Write once, work everywhere** — One frontend works with any OMSS backend
- ✅ **User choice** — Switch backends without frontend changes
- ✅ **Ecosystem growth** — Clear standard encourages more implementations
- ✅ **True interoperability** — Any OMSS frontend + any OMSS backend = it just works

---

## 📖 Specification

**Current Version:** [OMSS v1.0.0](/spec/v1.0/omss-v1.0.md) (Released January 15, 2026)

### Core Endpoints

```http
GET /v1/movies/{id}                                 # Movie streaming sources
GET /v1/tv/{id}/seasons/{s}/episodes/{e}            # TV episode streaming sources
GET /v1/proxy?data={base64url_encoded_json}         # Proxy to upstream providers
GET /v1/refresh/{sourceId}                          # Invalidate cache for sources
GET / or /v1 or /v1/health                          # Health check
```

### Key Features

- **TMDB-based identifiers** — Movies and TV shows identified by TMDB IDs
- **Proxy-based delivery** — All source URLs routed through backend proxy for header injection
- **Multi-source support** — Multiple streaming providers per content item
- **Multi-language audio** — Audio tracks with language codes and labels
- **Subtitle support** — VTT, SRT, ASS, SSA formats with language labels
- **Quality metadata** — Resolution info (1080p, 720p, etc.) with inference support
- **Diagnostic reporting** — Warnings for partial scrapes, inferred metadata
- **Expiration tracking** — `expiresAt` timestamp for cache management
- **Standardized errors** — Machine-readable error codes with trace IDs
- **URL versioning** — `/v1/` prefix for future compatibility

---

## 🚀 Quick Start

### For Backend Developers

**Implement an OMSS-compliant backend:**

1. Read the [full specification](/spec/v1.0/omss-v1.0.md)
2. Review the [OpenAPI spec](/spec/v1.0/omss-openapi.yml)
3. Implement the 4 required endpoints
4. Return responses matching the schemas
5. Use proxy paths for all source/subtitle URLs

**Minimum compliance requirements:**

- ✅ `GET /v1/movies/{id}` returns `SourceResponse`
- ✅ `GET /v1/tv/{id}/seasons/{s}/episodes/{e}` returns `SourceResponse`
- ✅ `GET /v1/proxy?data=...` proxies upstream requests
- ✅ `GET /` or `/v1` or `/v1/health` returns backend info
- ✅ All errors return `ErrorResponse` with `code`, `message`, `traceId`
- ✅ HTTPS in production (HTTP allowed for localhost only)

### For Frontend Developers

**Integrate with any OMSS backend:**

1. Review the [OpenAPI specification](/spec/v1.0/omss-openapi.yml)
2. Point your app to an OMSS backend URL
3. Call `/v1/movies/{id}` or `/v1/tv/{id}/seasons/{s}/episodes/{e}`
4. Parse `SourceResponse` and extract `sources` array
5. Play sources using the `url` field (proxy paths)
6. Load `subtitles` if available

**Frontend benefits:**

- One integration works with all OMSS backends
- Users can configure/switch backends without app updates
- Standardized error handling across backends

---

## 📦 Example Response

**Request:**

```http
GET /v1/movies/155 HTTP/1.1
Host: api.example.com
```

**Response:**

```json
{
    "sourceId": "bdfa40a7-a468-461c-8563-7a0c165f252c",
    "expiresAt": "2026-01-15T18:00:00Z",
    "sources": [
        {
            "id": "src_001",
            "url": "/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%7D",
            "type": "hls",
            "quality": "1080p",
            "audioTracks": [
                {
                    "language": "en",
                    "label": "English",
                    "default": true
                }
            ],
            "provider": {
                "id": "prov_1",
                "name": "Provider One"
            }
        }
    ],
    "subtitles": [
        {
            "url": "/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fsub.vtt%22%7D",
            "label": "English",
            "format": "vtt"
        }
    ],
    "diagnostics": []
}
```

---

## 📋 Documentation

- **[OMSS v1.0 Specification](/spec/v1.0/omss-v1.0.md)** — Full human-readable spec
- **[OpenAPI Specification](/spec/v1.0/omss-openapi.yml)** — Machine-readable API definition (Swagger/Redoc compatible)
- **[Contributing Guide](CONTRIBUTING.md)** — How to propose changes
- **[Code of Conduct](CODE_OF_CONDUCT.md)** — Community guidelines
- **[Security Policy](SECURITY.md)** — Reporting vulnerabilities

---

## 🏗️ Architecture

### How OMSS Decouples Frontends and Backends

```
┌─────────────────────────────────────────────────────────┐
│                    OMSS v1.0 Standard                   │
│              (Open Specification Document)              │
└─────────────────────────────────────────────────────────┘
                          ▲
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │Backend A │    │Backend B │    │Backend C │ <-- Can be open or closed source
    │(scrapers)│    │(scrapers)│    │(scrapers)│     can have different implementations/technologies
    └──────────┘    └──────────┘    └──────────┘
          ▲               ▲               ▲
          └───────────────┼───────────────┘
                          │
          ┌───────────────┴───────────────┐
          │                               │
          ▼                               ▼
    ┌──────────┐                    ┌──────────┐
    │Frontend 1│                    │Frontend 2│ <-- Frontends can now work with any OMSS backend
    │ (Web UI) │                    │(Mobile)  │     since they all follow the same spec!!🥳
    └──────────┘                    └──────────┘
```

**Key Benefits:**

- Backends implement OMSS spec → any frontend works
- Frontends implement OMSS client → any backend works
- Users choose their preferred backend in app settings
- Innovation happens independently on both sides

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

### Ways to Contribute

- 📝 Improve documentation and clarify ambiguities
- 🐛 Report issues or unclear sections in the spec
- 💡 Propose new features via GitHub Discussions
- 🔧 Build OMSS-compliant backends or frontends
- 🌍 Translate documentation to other languages
- ⭐ Star the repo and share with others

### Proposing Changes

OMSS follows semantic versioning:

- **MAJOR** (2.0.0) — Breaking changes
- **MINOR** (1.1.0) — Backward-compatible features
- **PATCH** (1.0.1) — Clarifications and bug fixes

To propose changes:

1. Open a [GitHub Discussion](https://github.com/omss-spec/omss-spec/discussions) describing the problem
2. If there's consensus, create an issue with a detailed proposal
3. Submit a pull request with spec changes
4. Maintainers review and merge if approved

---

## 📊 Compliance

### Required for OMSS v1.0 Compliance

A backend **MUST** implement:

1. **Core endpoints:**
    - `GET /v1/movies/{id}`
    - `GET /v1/tv/{id}/seasons/{s}/episodes/{e}`
    - `GET /v1/proxy?data={encoded_data}`
    - `GET /`, `/v1`, or `/v1/health`

2. **Source object** with required fields:
    - `id`, `url` (proxy path), `type`, `quality`, `audioTracks`, `provider`

3. **Subtitle object** with required fields:
    - `url` (proxy path), `label`, `format`

4. **Error responses** for all non-2xx responses:
    - `error.code`, `error.message`, `traceId`

5. **HTTP status codes:**
    - `200` for success
    - `400` for bad requests
    - `404` for not found
    - `500` for server errors

6. **Proxy routing:** All `url` fields MUST use proxy paths
7. **HTTPS** in production (HTTP allowed for `localhost` only)

A frontend **MUST** use the OMSS v1.0 endpoints and response formats to ensure compatibility with any compliant backend.

---

## 🛠️ Ecosystem

There is a Registry of known OMSS-compliant implementations. [See the list here](REGISTRY.md).

**Want to add your implementation?** [Submit a PR](CONTRIBUTING.md) adding it to this list!

---

## 📜 License

This specification is licensed under the [MIT License](LICENSE).

You are free to implement, extend, and distribute OMSS-compliant systems without restriction.

---

## 🙋 FAQ

### Why TMDB IDs?

TMDB provides stable, comprehensive IDs for movies and TV shows. It's free, well-maintained, and widely adopted.

### Why proxy-based URLs?

Many streaming providers require specific headers (Referer, User-Agent). Proxying through the backend allows header injection without CORS issues.

### Can I add custom fields to responses?

Yes! OMSS clients MUST ignore unknown fields, so backends can add custom metadata. Just don't remove required fields.

### How do I handle missing quality/language metadata?

Use `"unknown"` for quality and default to `"en"` for language. Add diagnostics explaining inference.

### What about authentication?

OMSS v1.0 is authentication-agnostic. Backends can implement auth (API keys, OAuth, etc.) as needed, although we recommend not to publicly expose OMSS endpoints at all. In our view, OMSS backends should be private services used only by trusted frontends on your LAN or personal devices.
To host movie streaming backends publicly you risk abuse from unauthorized users leeching your bandwidth and resources.

---

<div align="center">

**[Read the Full Specification →](/spec/v1.0/omss-v1.0.md)**

Built with ❤️ by the OMSS Foundation

</div>
