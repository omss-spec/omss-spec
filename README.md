<div align="center">
  
  <img src="https://repository-images.githubusercontent.com/1132306794/d7750bdf-7892-4b60-88e5-0fc8abcc112c" alt="OMSS Logo" />

# Open Media Streaming Specification

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.1.0-green.svg)](/spec/v1.1/omss-v1.1.md)
[![Specification](https://img.shields.io/badge/spec-OMSS-orange.svg)](https://github.com/omss-spec/omss-spec)

**A standardized REST API specification for streaming media backends**

[Specification](/spec/v1.1/omss-v1.1.md) · [OpenAPI](/spec/v1.1/omss-v1.1.yml) · [Contributing](CONTRIBUTING.md) · [Discussions](https://github.com/omss-spec/omss-spec/discussions)

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

**Current Version:** [OMSS v1.1.0](/spec/v1.1/omss-v1.1.md) (released May 26th, 2026)

### Core Endpoints

```http
GET /v1/movies/{id}?platform=native&provider=<provider_id>    # Movie streaming sources
GET /v1/tv/{id}/seasons/{s}/episodes/{e}?platform=web         # TV episode streaming sources
POST /v1/refresh/{id}                                         # Invalidate cache for sources
GET / or /v1                                                  # Health check
```

### Key Features

- **TMDB-based identifiers** — Movies and TV shows identified by TMDB IDs
- **Multi-source support** — Multiple streaming providers per content item
- **Multi-language audio** — Audio tracks with language codes and labels
- **Subtitle support** — VTT, SRT, ASS, SSA formats with language labels
- **Quality metadata** — Resolution info (FHD, HD, etc.)
- **Diagnostic reporting** — Warnings for partial scrapes, and errors
- **Expiration tracking** — `expiresAt` timestamp for cache management
- **Standardized errors** — Machine-readable error codes with trace IDs
- **URL versioning** — `/v1/` prefix for future compatibility

---

## 🚀 Quick Start

### For Backend Developers

**Implement an OMSS-compliant backend:**

Either do it yourself or [use the @omss/framework](https://github.com/omss-spec/framework)

1. Read the [full specification](/spec/v1.1/omss-v1.1.md)
2. Review the [OpenAPI spec](/spec/v1.1/omss-v1.1.yml)
3. Implement the required endpoints
4. Return responses matching the schemas

### For Frontend Developers

**Integrate with any OMSS backend:**

Either do it yourself or [use the @omss/sdk](https://github.com/omss-spec/sdk)

1. Review the [OpenAPI specification](/spec/v1.1/omss-v1.1.yml)
2. Point your app to an OMSS backend URL
3. Call `/v1/movies/{id}` or `/v1/tv/{id}/seasons/{s}/episodes/{e}`
4. Parse `SourceResponse` and extract `sources` array
5. Play your content!

**Frontend benefits:**

- One integration works with all OMSS backends
- Users can configure/switch backends without app updates
- Standardized error handling across backends

---

## 📦 Example Response

**Request:**

```http
GET /v1/movies/155?platform=web HTTP/1.1
Host: api.example.com
```

**Response:**

````json
{
    "id": "bdfa40a7-a468-461c-8563-7a0c165f252c",
    "expiresAt": "2026-01-11T20:56:00Z",
    "sources": [
        {
            "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
            "url": "https://api.example.com/playable/source/1/master.m3u8",
            "streamable": true,
            "type": "hls",
            "quality": "4k",
            "audioTracks": ["Original", "English"],
            "provider": {
                "id": "provider_alpha",
                "name": "Provider Alpha"
            }
        },
        {
            "id": "2b39e8b4-cf6c-4c89-88cf-6fbb55e0f4d3",
            "url": "https://api.example.com/playable/source/2/video.mp4",
            "streamable": false,
            "type": "mp4",
            "quality": "FHD",
            "audioTracks": ["English"],
            "provider": {
                "id": "provider_beta",
                "name": "Provider Beta"
            }
        }
    ],
    "subtitles": [
        {
            "id": "ec9cf9b0-ff1f-4e69-80cb-ef28bb4f6db8",
            "url": "https://api.example.com/subtitles/en-155.vtt",
            "label": "English",
            "format": "vtt",
            "provider": {
                "id": "provider_alpha",
                "name": "Provider Alpha"
            }
        }
    ],
    "diagnostics": [
        {
            "code": "PARTIAL_SCRAPE",
            "message": "Server 1 of Alpha failed during scraping",
            "source": "provider_alpha",
            "severity": "warning"
        }
    ]
}```

---

## 📋 Documentation

- **[OMSS v1.1 Specification](/spec/v1.1/omss-v1.1.md)** — Full human-readable spec
- **[OpenAPI Specification](/spec/v1.1/omss-v1.1.yml)** — Machine-readable API definition (Swagger/Redoc compatible)
- **[Contributing Guide](CONTRIBUTING.md)** — How to propose changes
- **[Code of Conduct](CODE_OF_CONDUCT.md)** — Community guidelines
- **[Security Policy](SECURITY.md)** — Reporting vulnerabilities

---

## 🏗️ Architecture

### How OMSS Decouples Frontends and Backends

````

┌─────────────────────────────────────────────────────────┐
│ OMSS v1.1 Standard │
│ (Open Specification Document) │
└─────────────────────────────────────────────────────────┘

          ┌───────────────────────────────┐
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

1. Open a [GitHub Issue](https://github.com/omss-spec/omss-spec/issues/new?template=rfc.yml) describing the problem
2. If there's consensus, fork the repo and make your changes in a new branch
3. Submit a pull request with spec changes
4. Maintainers review and merge if approved

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

### Can I add custom fields to responses?

Yes! OMSS clients MUST ignore unknown fields, so backends can add custom metadata. Just don't remove required fields.

### How do I handle missing quality/language metadata?

Use `"Auto"` for quality and default to `"Original"` for language.

### What about authentication?

OMSS v1.1 is authentication-agnostic. Backends can implement auth (API keys, OAuth, etc.) as needed, although we recommend not to publicly expose OMSS endpoints at all. In our view, OMSS backends should be private services used only by trusted frontends on your LAN or personal devices.
To host movie streaming backends publicly you risk abuse from unauthorized users leeching your bandwidth and resources.

---

## Note

This is **not** a piracy tool or a scraper framework per se. OMSS is a **specification** for how streaming backends should expose their data. That means, you can use OMSS to digitalize your own media collection YOU own.

---

<div align="center">

**[Read the Full Specification →](/spec/v1.1/omss-v1.1.md)**

Built with ❤️ by the OMSS Foundation

</div>
```
