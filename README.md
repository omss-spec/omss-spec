<div align="center">
  <img src="https://repository-images.githubusercontent.com/1132306794/6985c06b-8e53-41eb-9148-a81612fb6ac2" alt="OMSS Logo" />

# Open Media Streaming Specification (OMSS)

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](spec/v1.0/)
[![Standard](https://img.shields.io/badge/standard-OMSS-orange.svg)](https://omss.dev)

**OMSS** is an open standard that defines a unified REST API specification for streaming media backends. It enables any frontend (web, mobile, desktop) to seamlessly work with any OMSS-compliant backend.

</div>

> [!CAUTION]
> This is a draft specification and subject to change. We have not published a stable version yet.

## 🎯 What Problem Does OMSS Solve?

Today, every streaming backend has its own custom API format. Frontends must write custom integration code for each backend, creating:

- 🔴 Fragmentation across implementations
- 🔴 Duplicated integration effort
- 🔴 Difficult backend switching
- 🔴 Limited ecosystem growth

**OMSS standardizes** endpoint paths, request parameters, response schemas, and error handling so that:

- ✅ **Write once, work everywhere**: One frontend integration for all backends
- ✅ **User choice**: Easy to switch between backend providers
- ✅ **Ecosystem growth**: Clear standards encourage more implementations
- ✅ **Interoperability**: Any OMSS frontend + any OMSS backend = it just works

## 📋 Specification Overview

### Core Endpoints

```
GET /v1/movies/{tmdbId}                           # Get movie streaming sources
GET /v1/tv/{tmdbId}/seasons/{season}/episodes/{episode}  # Get TV episode sources
GET /v1/search?query={query}&type={type}          # Search for content (optional)
GET /                                             # API info and health check
```

### Key Features

- **Standardized Response Format**: Consistent JSON structure across all implementations
- **Multi-Source Support**: Returns multiple streaming sources per content item
- **Subtitle Support**: Standardized subtitle format and language codes
- **Error Handling**: Unified error response schema with clear error codes
- **Identifier Strategy**: TMDB as primary ID with support for IMDb, TVDB
- **Metadata Support**: Optional caching info, content metadata, source quality
- **Version Control**: URL-based versioning (`/v1/`, `/v2/`, etc.)

## 📖 Documentation

- **[Full Specification (v1.0)](spec/v1.0/omss-v1.0.md)** - Human-readable specification
- **[OpenAPI Spec (v1.0)](spec/v1.0/omss-v1.0.yaml)** - Machine-readable API definition
- **[Implementation Guide](docs/implementation-guide.md)** - How to build OMSS-compliant backends
- **[Architecture Overview](docs/architecture.md)** - Design principles and patterns
- **[Identifier Design](docs/identifier-design.md)** - ID strategy and best practices
- **[Migration Guide](docs/migration-guide.md)** - Upgrading between OMSS versions

## 🚀 Quick Start

### For Backend Developers

1. Read the [full specification](spec/v1.0/omss-v1.0.md)
2. Review [example responses](examples/)
3. Check the [reference implementation](https://github.com/omss-foundation/reference-implementation)
4. Implement required endpoints
5. Test compliance with [validation suite](https://github.com/omss-foundation/compliance-tests)

### For Frontend Developers

1. Review [OpenAPI specification](spec/v1.0/omss-v1.0.yaml)
2. See [example responses](examples/)
3. Use [client libraries](https://github.com/omss-foundation/client-libraries) (optional)
4. Connect to any OMSS-compliant backend

## 📦 Example Response

```json
{
  "status": "success",
  "media": {
    "id": "155",
    "type": "movie",
    "title": "The Dark Knight",
    "year": 2008,
    "identifiers": {
      "tmdb": "155",
      "imdb": "tt0468569"
    }
  },
  "sources": [
    {
      "id": "src_provider_001",
      "file": "https://example.com/stream.m3u8",
      "type": "hls",
      "quality": "1080p",
      "language": "en",
      "provider": "ExampleProvider"
    }
  ],
  "subtitles": [
    {
      "id": "sub_en_001",
      "url": "https://example.com/subs.vtt",
      "language": "en",
      "format": "vtt"
    }
  ]
}
```

## 🤝 Contributing

We welcome contributions! Please see:

- **[Contributing Guide](CONTRIBUTING.md)** - How to contribute
- **[Code of Conduct](CODE_OF_CONDUCT.md)** - Community guidelines
- **[GitHub Discussions](https://github.com/omss-foundation/specification/discussions)** - Ask questions, share ideas

### Ways to Contribute

- 📝 Improve documentation
- 🐛 Report issues or ambiguities in the spec
- 💡 Propose new features via RFCs
- 🔧 Build OMSS-compliant implementations
- 🌍 Translate documentation
- ⭐ Star the repo and spread the word

## 🏗️ OMSS Ecosystem

- **[Reference Implementation](https://github.com/omss-foundation/reference-implementation)** - Example backend
- **[Compliance Tests](https://github.com/omss-foundation/compliance-tests)** - Validate your implementation
- **[Client Libraries](https://github.com/omss-foundation/client-libraries)** - SDKs for various languages
- **[Backend Registry](https://github.com/omss-foundation/registry)** - Directory of OMSS backends
- **[Website](https://omss.dev)** - Documentation and guides

## 📊 Compliance

To be OMSS v1.0 compliant, a backend must:

- ✅ Implement required endpoints (`/v1/movies/{id}`, `/v1/tv/{id}/...`)
- ✅ Return responses matching the specified schemas
- ✅ Use standard error codes and format
- ✅ Support TMDB IDs as primary identifier

Optional features (recommended but not required):

- Search endpoint
- Caching metadata
- Multiple identifier support
- Authentication

## 📜 License

This specification is licensed under the [MIT License](LICENSE).

You are free to implement, extend, and distribute OMSS-compliant systems without restriction.
