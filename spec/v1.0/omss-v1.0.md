# OMSS v1.0 Specification

**Open Media Streaming Specification**

Version: 1.0.0  
Status: Stable  
Released: January 11, 2026  
License: MIT

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Core Principles](#2-core-principles)
3. [API Conventions](#3-api-conventions)
4. [Endpoints](#4-endpoints)
5. [Request Specifications](#5-request-specifications)
6. [Response Specifications](#6-response-specifications)
7. [Error Handling](#7-error-handling)
8. [Caching and Performance](#8-caching-and-performance)
9. [Security Considerations](#9-security-considerations)
10. [Compliance Requirements](#10-compliance-requirements)
11. [Versioning and Compatibility](#11-versioning-and-compatibility)
12. [Provider Data Handling](#12-provider-data-handling)

---

## 1. Introduction

### 1.1 Purpose

OMSS (Open Media Streaming Specification) defines a unified REST API standard for streaming media backends. It enables any frontend application to work seamlessly with any OMSS-compliant backend through standardized endpoints for retrieving streaming sources.

### 1.2 Problem Statement

Modern streaming backends scrape data from various providers with inconsistent formats:

- **No standardized metadata**: Files may be MP4, HLS, or DASH without clear type information.
- **Missing quality information**: Bitrate, resolution, and codec data are often unavailable.
- **Language ambiguity**: Audio/subtitle language tracks may be unlabeled.
- **Ephemeral URLs**: Signed URLs with expiration times require special handling.
- **Cross-origin restrictions**: Sources require proxy handling for CORS and access control.

**OMSS standardizes streaming sources** while acknowledging these real-world constraints.

### 1.3 Design Philosophy

- **Source-focused**: OMSS is about standardizing streaming sources, not media metadata.
- **Pragmatic**: Designed for backends that scrape third-party providers with incomplete data.
- **Progressive disclosure**: Required fields are minimal; optional fields provide enhanced functionality.
- **HTTP-first**: Uses HTTP status codes, headers, and semantics correctly.
- **Proxy-enabled**: All sources route through the OMSS proxy endpoint to handle provider restrictions.

### 1.4 Scope

**In Scope:**

- REST API endpoint definitions for source retrieval.
- Standardized source/subtitle schemas.
- Proxy endpoint for source access.
- Error handling patterns.
- Caching strategies.

**Out of Scope:**

- Access control/authentication — implement that yourself. (you are no fun😔. Personal Note: I would not recommend hosting such a backend publicly anyway, since it brings risk and costs with it. For personal use you do not need these.)
- Media metadata (ratings, descriptions, posters, etc.) — use the TMDB API directly.
- Search functionality — use the TMDB API.
- Provider-specific scraping logic — implement that yourself.
- Video player implementation — whichever player you use.

### 1.5 TMDB Integration

OMSS uses **TMDB (The Movie Database) as the primary identifier** for all media content:

- **Frontend responsibility**: Fetch media metadata (title, year, poster, description, ratings) from the TMDB API.
- **OMSS responsibility**: Provide streaming sources and subtitles for TMDB IDs.
- **Clean separation**: OMSS focuses exclusively on source standardization.

---

## 2. Core Principles

### 2.1 HTTP-First Design

- **Primary signal**: HTTP status codes indicate success or failure.
- **Secondary signal**: The response body provides details.
- **Standard headers**: Use `Cache-Control`, `ETag`, `Retry-After`, etc.

### 2.2 Proxy-First Architecture

- **All sources route through the proxy**: Every `url` field uses the OMSS proxy endpoint.
- **CORS handling**: The proxy manages cross-origin requests.

The Proxy also has to rewrite the manifest files (HLS/DASH) to ensure all sub-resources also go through the proxy.

### 2.3 Unknown Data Handling

Backends MUST handle incomplete provider data gracefully:

- **Missing metadata**: Omit optional fields rather than returning null or empty values.
- **Quality and language inference**: Attempt to infer quality and language from filenames, manifests, resolution, or bitrate.
- **Type detection**: Analyze file extension, MIME type, or content headers.

### 2.4 Identifier Strategy

- **Primary**: TMDB (The Movie Database) IDs.
- **Required**: All media MUST have a TMDB ID.

### 2.5 Versioning

- **URL-based**: `/v1/`, `/v2/` in the path.
- **Semantic versioning**: `MAJOR.MINOR.PATCH`.
- **Breaking changes**: Require a new major version.

---

## 3. API Conventions

### 3.1 Base URL

```text
https://api.example.com/v1
```

All endpoints are relative to this base URL. From here on, all absolute paths are meant to be relative, starting from the web root.

### 3.2 Content Type

- **Request**: `application/json; charset=utf-8`
- **Response**: `application/json; charset=utf-8`

### 3.3 Character Encoding

UTF-8 MUST be used for all text data.

### 3.4 Date/Time Format

ISO 8601 format MUST be used:

```text
2026-01-11T19:56:00Z
2026-01-11T19:56:00+01:00
```

### 3.5 Naming Conventions

- **Fields**: camelCase (e.g., `expiresAt`, `mimeType`).
- **Enums**: lowercase (e.g., `hls`, `mp4`, `vtt`).
- **Error codes**: SCREAMING_SNAKE_CASE (e.g., `INVALID_ID`, `NOT_FOUND`).

### 3.6 Content Negotiation

OMSS v1.0 supports only JSON responses.

**Supported**:

- `Accept: application/json` → 200 OK with JSON
- `Accept: */*` → 200 OK with JSON (default)

**Unsupported**:

- `Accept: application/xml` → Throws a 406 Not Acceptable error. [See 7.3 Standard Error Codes](#73-standard-error-codes)

### 3.7 Preflight Requests

OMSS backends MUST handle OPTIONS requests for CORS preflight correctly.

**Example:**

```http
OPTIONS /v1/movies/155
```

**Response: 204 No Content**

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, OPTIONS
Access-Control-Allow-Headers: Content-Type, Accept
```

---

## 4. Endpoints

### 4.1 Home

**Purpose**: General information about the backend.

```http
GET /
GET /v1
```

**Response: 200 OK**

```json
{
    "name": "Example OMSS Implementation",
    "version": "1.0.0",
    "status": "operational",
    "endpoints": {
        "movie": "/v1/movies/{id}",
        "tv": "/v1/tv/{id}/seasons/{s}/episodes/{e}",
        "proxy": "/v1/proxy/?data={encoded_data}"
    },
    "spec": "omss",
    "note": "Here some description, motivation, disclaimer or anything else. Here you could also link your suggested OMSS frontend."
}
```

**Required Fields:**

- `name` (string): Implementation name.
- `version` (string): Semantic version of the OMSS implementation (e.g., `1.0.0`).
- `status` (string): API status — `operational`, `degraded`, or `down` (e.g. maintenance).
- `endpoints` (object): Functions mapped to paths.
    - `movie` (string): Path to the movie endpoint. MUST contain the `{id}` placeholder where the frontend will insert the TMDB ID.
    - `tv` (string): Path to the TV show endpoint. MUST contain the `{id}`, `{s}` and `{e}` placeholders where the frontend will insert the TMDB ID, season and episode respectively.
    - `proxy` (string): Path to the proxy endpoint. MUST contain the `{encoded_data}` placeholder where the frontend will insert the URI-encoded data object.
- `spec` (string): When implementing OMSS, this MUST be `"omss"`.

**Optional Fields:**

- `note` (string): A custom message.

### 4.2 Movie Sources

**Purpose**: Retrieve movie streaming sources by TMDB ID.

```http
GET /v1/movies/{id}
```

**Path Parameters:**

- `id` (string, required): TMDB movie ID.

**Example Request:**

```http
GET /v1/movies/155
```

**Response: 200 OK** (see [6.1 Success Response](#61-success-response))

### 4.3 TV Episode Sources

**Purpose**: Retrieve TV episode streaming sources.

```http
GET /v1/tv/{id}/seasons/{s}/episodes/{e}
```

**Path Parameters:**

- `id` (string, required): TMDB series ID.
- `s` (integer, required): Season number (≥ 0).
- `e` (integer, required): Episode number (≥ 1).

**Example Requests:**

```http
GET /v1/tv/1396/seasons/1/episodes/1
GET /v1/tv/1396/seasons/5/episodes/14
```

**Response: 200 OK** (see [6.1 Success Response](#61-success-response))

### 4.4 Proxy Endpoint

**Purpose**: Unified proxy endpoint for streaming sources, subtitles, and manifests through the OMSS backend, since almost all providers have CORS or other restrictive security logic in place. This endpoint will make a request to `url` with the provided `headers` and stream the response back to the client.

The Proxy also has to rewrite the manifest files (HLS/DASH) to ensure all sub-resources also go through the proxy.

**Important Note:** To send the `data` object to the backend you have to use **`encodeURIComponent()`** (not `encodeURI()`).

```http
GET /v1/proxy?data={encoded_data}
```

**Query Parameters:**

- `data` (string, required): URL-encoded JSON proxy request object (use `encodeURIComponent()`, NOT `encodeURI()`).

**Proxy Request Object Schema:**

```json
{
    "url": "https://provider.com/stream/abcd1234.mp4",
    "headers": {
        "Referer": "https://provider.com/watch/abcd1234",
        "more": "custom headers as needed"
    }
}
```

**Example Request:**

```http
GET /v1/proxy?data={encodeURIComponent(data)}
```

**Response: 200 OK or 206 Partial Content**

Returns the actual streaming content with appropriate headers:

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.apple.mpegurl
Content-Length: 524288
Cache-Control: public, max-age=3600
Access-Control-Allow-Origin: *

[streaming content]
```

### 4.5 Refresh Endpoint

**Purpose**: Unified endpoint to refresh a source by its unique ID.

```http
GET /v1/refresh/{responseId}
```

**Path Parameters:**

- `responseId` (string, required): The unique ID of the source to refresh.

**Response Object Schema:**

```json
{
    "status": "OK"
}
```

**Example Request:**

```http
GET /v1/refresh/cf6c3c2d-17be-4a5a-9488-bf12e70dca5a
```

**Response: 200 OK**

If the `responseId` is valid the response will always be `200 OK`, since the backend should simply remove the cache entry and will try to fetch a new source the next time the frontend requests it. If it is invalid, see [7.3 Standard Error Codes](#73-standard-error-codes).

---

## 5. Request Specifications

### 5.1 Headers

**Required:**

- `Accept: application/json; charset=utf-8`

### 5.2 Query Parameters

**Validation Rules:**

- Query parameters are case-sensitive.
- Boolean parameters: `true` or `false` (lowercase).

### 5.3 Path Parameters

**Validation Rules:**

- `id`: String (numeric characters ONLY, treated as a string).
- `s`: Non-negative integer (0–99 typical range).
- `e`: Positive integer (≥ 1).

---

## 6. Response Specifications

### 6.1 Success Response

**HTTP Status: 200 OK**

```json
{
    "responseId": "bdfa40a7-a468-461c-8563-7a0c165f252c",
    "expiresAt": "2026-01-11T20:56:00Z",
    "sources": [
        // 6.2 Source Objects here
    ],
    "subtitles": [
        // 6.3 Subtitle Objects here
    ],
    "diagnostics": [
        // 6.4 Warning/Error Objects here
    ]
}
```

**Required Top-Level Fields:**

- `responseId` (string): Unique response identifier (e.g. UUID v4).
- `sources` (array): Array of Source objects (at least one required. if no source found, see [7.4.1](#741-media-exists-no-sources)).
- `subtitles` (array): Array of Subtitle objects (may be empty).
- `diagnostics` (array): Non-fatal warnings/errors about data quality/inference and provider issues (may be empty).
- `expiresAt` (string, optional): ISO 8601 timestamp when sources expire (for signed/temporary URLs).

---

### 6.2 Source Object

**Purpose**: Represents a streaming source with quality and technical metadata.

```json
{
    "url": "https://localhost:3000/v1/proxy/?data=encodeURIComponent({\"url\":\"https://provider.com/stream/abcd1234.m3u8\",\"headers\":{\"Referer\":\"https://provider.com/watch/abcd1234\"}})",
    "type": "hls",
    "quality": "1080p",
    "audioTracks": [
        {
            "language": "en",
            "label": "English"
        },
        {
            "language": "es",
            "label": "Spanish"
        },
        {
            "language": "de",
            "label": "German"
        }
    ],
    "provider": {
        "id": "provider_123",
        "name": "Example Provider"
    }
}
```

#### 6.2.1 Required Fields

- **`url`** (string): Proxy endpoint URL for this source. Must contain the encoded data object.
    - MUST be in the format: `/v1/proxy/?data={encoded_data}`.
    - Example:  
      `/v1/proxy/?data=%7B%0D%0A%20%20%20%20%22url%22%3A%20%22https%3A%2F%2Fprovider%2Ecom%2Fstream%2Fabcd1234%2Emp4%22%2C%0D%0A%20%20%20%20%22headers%22%3A%20%7B%0D%0A%20%20%20%20%20%20%20%20%22Referer%22%3A%20%22https%3A%2F%2Fprovider%2Ecom%2Fwatch%2Fabcd1234%22%2C%0D%0A%20%20%20%20%20%20%20%20%22more%22%3A%20%22custom%20headers%20as%20needed%22%0D%0A%20%20%20%20%7D%0D%0A%7D`

- **`type`** (string): Source type, one of:
    - `hls` — HTTP Live Streaming (M3U8).
    - `dash` — MPEG-DASH (MPD).
    - `mp4` — MP4 file.
    - `mkv` — MKV file.
    - `webm` — WebM file.
    - `embed` — Embedded player (iframe). Note: This is not recommended for OMSS compliance, but can be used as a last resort. Avoid using embed and use direct streaming links instead. For frontend developers: Try to prevent users from using embedded players, since they usually have ads, trackers, and other unwanted content. Backend developers: Do your best to avoid using embed links. If you must return an embed link, go as far down the embed stack as possible to get the cleanest link.

- **`quality`** (string): Video quality. There is no standardized list of qualities.

- **`audioTracks`** (array): Array of audio track objects. Default is English (`en`) if unknown.
    - Each audio track object contains:
        - `language` (string): Audio language (ISO 639-1 code).
        - `label` (string): Human-readable language name.

- **`provider`** (object): Information about the upstream provider.
    - `id` (string): Unique provider identifier.
    - `name` (string): Human-readable provider name.

#### 6.2.2 Handling Unknown Source Data

**When scraping providers with incomplete metadata:**

##### Type Detection Order

1. **File extension**:
    - `.m3u8` → `hls`
    - `.mpd` → `dash`
    - `.mp4` → `mp4`
    - `.mkv` → `mkv`
    - `.webm` → `webm`

2. **MIME type** from the HTTP `Content-Type` header.
3. **Content inspection** (first few bytes):
    - `#EXTM3U` → `hls`
    - `<?xml` with `MPD` → `dash`
    - Magic numbers for video containers.

4. **Default**: If unknown, use `embed` as a fallback.

##### Quality Inference

1. **Parse filename** for quality indicators:
    - Regex example: `/(\d{3,4}p|4K|8K|HD|SD|UHD)/i`.
    - Examples: `movie_1080p.mp4`, `stream_4k.m3u8`.

2. **Analyze resolution** from manifest or stream metadata:
    - HLS: `#EXT-X-STREAM-INF:RESOLUTION=1920x1080`.
    - DASH: `<Representation width="1920" height="1080">`.

3. **If unknown**: `quality` MUST be provided, so use `"unknown"`.

##### Language Detection

1. **Parse from filename**:
    - Patterns: `movie.en.mp4`, `stream_eng.m3u8`, `video-english.mp4`.

2. **Extract from manifest metadata**:
    - HLS: `#EXT-X-MEDIA:LANGUAGE="en"`.
    - DASH: `<Representation lang="en">`.

3. **If unknown**: Set the `language` field to `en` (English) by default.

#### 6.2.3 Multi-Audio Handling

**For HLS/DASH with native multi-audio**:
The file returned by the `url` field contains multiple audio tracks. The `audioTracks` array describes each track.

> Seperate audio tracks are not supported in this version of the specification.

**Player Responsibility**:
Frontend players handle audio track switching using manifest metadata.

---

### 6.3 Subtitle Object

**Purpose**: Represents subtitle/caption tracks.

```json
{
    "url": "http://localhost:3000/v1/proxy?data=encodeURIComponent({\"url\":\"https://provider.com/subs/abc1234.vtt\",\"headers\":{\"Referer\":\"https://provider.com/download\"}})",
    "label": "English",
    "format": "vtt"
}
```

#### 6.3.1 Required Fields

- **`url`** (string): Proxy endpoint path for this subtitle.
    - MUST be in the format: `/v1/proxy?data={encoded_data}`.
    - Example:  
      `/v1/proxy?data=%7B%0D%0A%20%20%20%20%22url%22%3A%20%22https%3A%2F%2Fprovider%2Ecom%2Fsub%2Fabc1234%2Evtt%22%2C%0D%0A%20%20%20%20%22headers%22%3A%20%7B%0D%0A%20%20%20%20%20%20%20%20%22Referer%22%3A%20%22https%3A%2F%2Fprovider%2Ecom%2Fdownload%2Fabcd1234%22%2C%0D%0A%20%20%20%20%20%20%20%20%22more%22%3A%20%22custom%20headers%20as%20needed%22%0D%0A%20%20%20%20%7D%0D%0A%7D`

- **`label`** (string): Human-readable language name.
    - Examples: `English`, `Spanish`, `Deutsch`, `日本語`.
    - Should match the language code.
    - Backends should attempt to infer this from filename or content, without adding extra metadata or ads.
    - If unknown, use `"Unknown"`.

- **`format`** (string): Subtitle format, one of:
    - `vtt` — WebVTT (preferred for web; default).
    - `srt` — SubRip.
    - `ass` — Advanced SubStation Alpha.
    - `ssa` — SubStation Alpha.
    - `ttml` — Timed Text Markup Language.

#### 6.3.3 Handling Unknown Subtitle Data

**When scraping providers with incomplete subtitle metadata:**

##### Label Generation

1. **Parse from filename**:
    - Patterns: `subs.en.vtt`, `subtitle_english.srt`, `movie.spa.vtt`.
    - Common codes: `eng`, `en`, `spa`, `es`, etc.

2. **If unknown**: Use `"Unknown"` as the label.

##### Format Detection

1. **File extension**:
    - `.vtt` → `vtt`.
    - `.srt` → `srt`.
    - `.ass` → `ass`.
    - `.ssa` → `ssa`.

2. **If unknown**: Use `vtt` as the default format.

---

### 6.4 Warning/Error Objects in Success Response

**Purpose**: Report non-response-fatal issues encountered during scraping. If `field` is an empty string, it means the warning applies to the entire response.

**Description**: A non-response-fatal warning/error is an issue encountered during scraping that does not prevent the backend from returning a successful response. These warnings/errors provide additional context about potential data quality issues or provider-specific scraping failures (for example: `"Provider [name] failed to respond"` would be a non-response-fatal error).

```json
{
    "code": "QUALITY_INFERRED",
    "message": "Video quality was inferred from filename",
    "field": "sources[0].quality",
    "severity": "warning"
}
```

**Fields:**

- **`code`** (string): Machine-readable warning/error code.
- **`message`** (string): Human-readable description.
- **`field`** (string, optional): JSON path to the affected field.
- **`severity`** (string): One of:
    - `info` — Informational notice.
    - `warning` — Non-fatal issue.
    - `error` — Fatal issue.

#### 6.4.1 Standard Warning/Error Codes

| Error/Warning Code        | Description                                                |
| ------------------------- | ---------------------------------------------------------- |
| `QUALITY_INFERRED`        | Video quality was inferred.                                |
| `LANGUAGE_INFERRED`       | Audio language was inferred.                               |
| `TYPE_INFERRED`           | Source type was inferred.                                  |
| `SUBTITLE_LABEL_INFERRED` | Subtitle label was inferred.                               |
| `PROVIDER_ERROR`          | Provider request timed out.                                |
| `PARTIAL_SCRAPE`          | Some providers failed. This applies to the whole response. |

---

## 7. Error Handling

### 7.1 Error Response Format

**All non-2xx responses MUST include:**

```json
{
    "error": {
        "code": "NOT_FOUND",
        "message": "No media found with the provided TMDB ID",
        "details": {
            "parameter": "id",
            "value": "99999999"
        }
    },
    "traceId": "6081fa12-6613-4d1a-8757-aa0767413d24"
}
```

**Required Fields:**

- **`error.code`** (string): Machine-readable error code.
- **`error.message`** (string): Human-readable error description.
- **`traceId`** (string): Unique identifier for error tracking/debugging.

**Optional Fields:**

- **`error.details`** (object): Additional context about the error. Frontends should not display this.

### 7.3 Standard Error Codes

| Error Code               | HTTP Status | Description                             |
| ------------------------ | ----------- | --------------------------------------- |
| `INVALID_TMDB_ID`        | 400         | Invalid TMDB ID format.                 |
| `INVALID_PARAMETER`      | 400         | Invalid query/path parameter.           |
| `MISSING_PARAMETER`      | 400         | Required parameter missing.             |
| `INVALID_SEASON`         | 400         | Season number out of valid range.       |
| `INVALID_EPISODE`        | 400         | Episode number invalid.                 |
| `INVALID_RESPONSE_ID`    | 400         | Invalid responseId format.              |
| `RESPONSE_ID_NOT_FOUND`  | 404         | responseId not found for refresh.       |
| `NO_SOURCES_AVAILABLE`   | 404         | Media exists but no sources were found. |
| `ENDPOINT_NOT_FOUND`     | 404         | Invalid endpoint path.                  |
| `METHOD_NOT_ALLOWED`     | 405         | HTTP method not supported.              |
| `INTERNAL_ERROR`         | 500         | Unexpected server error.                |
| `UNSUPPORTED_MEDIA_TYPE` | 415         | Unsupported Content-Type in request.    |

### 7.4 Edge Cases

#### 7.4.1 Media Exists, No Sources

**HTTP Status: 404 Not Found**

When the TMDB ID is valid but no streaming sources are available:

```json
{
    "error": {
        "code": "NO_SOURCES_AVAILABLE",
        "message": "No streaming sources found for TMDB ID: 155"
    },
    "traceId": "abc-123"
}
```

#### 7.4.2 Invalid TMDB ID

**HTTP Status: 400 Bad Request**

```json
{
    "error": {
        "code": "INVALID_TMDB_ID",
        "message": "TMDB ID must be numeric",
        "details": {
            "parameter": "tmdbId",
            "value": "not-a-number"
        }
    },
    "traceId": "abc-123"
}
```

#### 7.4.3 TMDB ID Not Found

**HTTP Status: 404 Not Found**

```json
{
    "error": {
        "code": "NOT_FOUND",
        "message": "No media found with TMDB ID: 99999999"
    },
    "traceId": "abc-123"
}
```

---

## 8. Caching and Performance

### 8.1 Caching Strategies

OMSS does not mandate specific caching strategies, but recommends best practices for both backends and clients to optimize performance.

Backends should implement caching strategies to improve performance and reduce load on upstream providers, and also to help avoid IP bans.

Recommended strategies include:

- In-memory caching (e.g. Redis, Memcached).
- Cache sources and subtitles separately.
- Cache sources for around 2 hours (7,200 seconds) by default.
- Cache subtitles for longer durations (e.g. 24 hours).
- When the refresh endpoint is called, the cache for that specific source should be invalidated immediately.
- For large/public backends, consider caching with a CDN and browser caching headers.
- Do NOT cache proxy responses.

### 8.5 Performance Recommendations

**For Backends:**

- Implement request coalescing for concurrent identical requests.
- Use connection pooling for upstream providers.
- Implement **circuit breakers** for failing providers.
- Pre-fetch sources for popular content.

**For Clients:**

- Respect cache headers and ETags.
- Implement exponential backoff for retries.
- Prefetch sources before playback.
- Monitor `expiresAt` and refresh proactively.
- Use conditional requests (`If-None-Match`).
- Implement client-side caching of source lists.

---

## 9. Security Considerations

### 9.1 HTTPS Requirement

- **Production**: All endpoints MUST use HTTPS.
- **Development**: HTTP is allowed for `localhost` only.

### 9.2 Input Validation

Backends MUST validate:

1. **Path parameters**:
    - `id`: Alphanumeric, max 20 characters.
    - `s`: Integer, 0–99.
    - `e`: Integer, 1–9,999.
    - `responseId`: Alphanumeric with hyphens/underscores, max 128 characters.

2. **Query parameters**:
    - Reject SQL injection attempts.

3. **Headers**:
    - Enforce size limits (to prevent header overflow attacks).

### 9.3 Proxy Security

It is recommended that you implement SSRF safeguards: validate that upstream URLs are not internal/private IPs.

---

## 10. Compliance Requirements

### 10.1 Required for OMSS v1.0 Compliance

A backend MUST implement:

1. **Core endpoints**:
    - `GET /v1/movies/{tmdbId}`
    - `GET /v1/tv/{tmdbId}/seasons/{s}/episodes/{e}`
    - `GET /v1/proxy?data={encoded_data}`
    - `GET /`, `/v1`, or `/v1/health`

2. **Source object** with required fields:
    - `url` (as proxy path), `type`, `quality`, `audioTracks`, `provider`.

3. **Subtitle object** with required fields:
    - `url` (as proxy path), `label`, `format`.

4. **Error responses** for all non-2xx responses with:
    - `error.code`, `error.message`, `traceId`.

5. **HTTP status codes**:
    - 200 for success.
    - 400 for bad requests.
    - 404 for not found.
    - 500 for server errors.

6. **Proxy routing**: All `url` fields MUST use proxy paths.
7. **HTTPS** in production.

---

## 11. Versioning and Compatibility

### 11.1 Semantic Versioning

OMSS follows Semantic Versioning 2.0.0:

```text
MAJOR.MINOR.PATCH
  1 . 0 . 0
```

- **MAJOR**: Breaking changes (incompatible API changes).
- **MINOR**: Backward-compatible new features.
- **PATCH**: Backward-compatible bug fixes/clarifications.

### 11.2 URL Versioning

Version is in the URL path:

```text
/v1/movies/155   # Version 1.x
/v2/movies/155   # Version 2.x (future)
```

### 11.4 Deprecation Policy

1. **Announce** at least 6 months before removal.
2. **Headers**: Deprecated endpoints return:

    ```text
    Deprecation: true
    Sunset: Wed, 01 Jul 2026 23:59:59 GMT
    ```

3. **Documentation**: Migration guides are provided.
4. **Support**: Deprecated features are supported for ≥ 1 major version.
5. **Removal**: Only in a new major version.

### 11.5 Client Compatibility Expectations

Clients MUST:

1. **Ignore unknown fields** in responses.
2. **Use HTTP status codes** as the primary signal.
3. **Handle new error codes** gracefully.
4. **Support redirect** responses (3xx).

Clients SHOULD:

1. **Version lock** to a specific OMSS version (`/v1/`).
2. **Upgrade proactively** when new versions are released.
3. **Monitor deprecation headers**.
4. **Implement graceful degradation** for missing required fields.

---

## 12. Provider Data Handling

### 12.1 Real-World Constraints

Backends scrape data from third-party providers with:

- **No standardization**: Each provider has custom formats.
- **Incomplete metadata**: Missing quality, language and other information.
- **Inconsistent naming**: Files like `unknown.mp4`, `stream`.
- **Dynamic URLs**: Signed/expiring URLs requiring refresh.
- **Unreliable availability**: Sources disappear without notice.

---

## 13. Complete Examples

### 13.1 Movie Request - Success with Multiple Sources

**Request**:

```http
GET /v1/movies/155 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```json
{
    "responseId": "bdfa40a7-a468-461c-8563-7a0c165f252c",
    "expiresAt": "2026-01-15T18:00:00Z",
    "sources": [
        {
            "id": "src_001",
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%7D",
            "type": "hls",
            "quality": "1080p",
            "audioTracks": [{ "language": "en", "label": "English" }],
            "provider": { "id": "prov_1", "name": "Provider One" }
        }
    ],
    "subtitles": [
        {
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fsub.vtt%22%7D",
            "label": "English",
            "format": "vtt"
        }
    ],
    "diagnostics": []
}
```

### 13.2 TV Episode Request - Multiple Audio Tracks

**Request**:

```http
GET /v1/tv/1396/seasons/1/episodes/1 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```json
{
    "responseId": "e7f2c8d1-9a3b-4c5e-8f1d-2a6b9c4e7f3a",
    "expiresAt": "2026-01-15T20:30:00Z",
    "sources": [
        {
            "id": "src_tv_001",
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fstreaming.provider.com%2Fbreaking-bad%2Fs01e01.m3u8%22%2C%22headers%22%3A%7B%22Referer%22%3A%22https%3A%2F%2Fprovider.com%22%7D%7D",
            "type": "hls",
            "quality": "1080p",
            "audioTracks": [
                { "language": "en", "label": "English" },
                { "language": "es", "label": "Spanish" },
                { "language": "de", "label": "German" }
            ],
            "provider": { "id": "prov_streaming", "name": "Streaming Provider" }
        },
        {
            "id": "src_tv_002",
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn2.example.com%2Fbb-s01e01-720p.mp4%22%7D",
            "type": "mp4",
            "quality": "720p",
            "audioTracks": [{ "language": "en", "label": "English" }],
            "provider": { "id": "prov_cdn2", "name": "CDN Provider 2" }
        }
    ],
    "subtitles": [
        {
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fsubs.provider.com%2Fbb-s01e01-en.vtt%22%7D",
            "label": "English",
            "format": "vtt"
        },
        {
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fsubs.provider.com%2Fbb-s01e01-es.srt%22%7D",
            "label": "Spanish",
            "format": "srt"
        }
    ],
    "diagnostics": [
        {
            "code": "QUALITY_INFERRED",
            "message": "Quality for source 'src_tv_002' was inferred from filename",
            "field": "sources.quality",
            "severity": "warning"
        }
    ]
}
```

### 13.3 Movie Request - Partial Scrape with Diagnostics

**Request**:

```http
GET /v1/movies/550 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```json
{
    "responseId": "3f8a9c2e-6d4b-4a1c-9e7f-5b3d8c1a6e4f",
    "expiresAt": "2026-01-15T19:45:00Z",
    "sources": [
        {
            "id": "src_550_001",
            "url": "http://localhost:3000/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Ffast-cdn.net%2Ffight-club.m3u8%22%2C%22headers%22%3A%7B%22User-Agent%22%3A%22Mozilla%2F5.0%22%7D%7D",
            "type": "hls",
            "quality": "unknown",
            "audioTracks": [{ "language": "en", "label": "English" }],
            "provider": { "id": "fast_cdn", "name": "Fast CDN" }
        }
    ],
    "subtitles": [],
    "diagnostics": [
        {
            "code": "PROVIDER_ERROR",
            "message": "Provider 'Backup CDN' failed to respond within timeout",
            "field": "",
            "severity": "error"
        },
        {
            "code": "PARTIAL_SCRAPE",
            "message": "Only 1 of 3 providers returned results",
            "field": "",
            "severity": "warning"
        },
        {
            "code": "QUALITY_INFERRED",
            "message": "Could not determine quality from manifest or filename",
            "field": "sources.quality",
            "severity": "warning"
        },
        {
            "code": "LANGUAGE_INFERRED",
            "message": "Audio language defaulted to English",
            "field": "sources.audioTracks.language",
            "severity": "info"
        }
    ]
}
```

### 13.4 Error Response - Invalid TMDB ID

**Request**:

```http
GET /v1/movies/not-a-number HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
```

```json
{
    "error": {
        "code": "INVALID_TMDB_ID",
        "message": "TMDB ID must be numeric",
        "details": {
            "parameter": "id",
            "value": "not-a-number",
            "expected": "numeric string"
        }
    },
    "traceId": "a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d"
}
```

### 13.5 Error Response - No Sources Available

**Request**:

```http
GET /v1/movies/999999 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```http
HTTP/1.1 404 Not Found
Content-Type: application/json; charset=utf-8
```

```json
{
    "error": {
        "code": "NO_SOURCES_AVAILABLE",
        "message": "No streaming sources found for TMDB ID: 999999",
        "details": {
            "parameter": "id",
            "value": "999999",
            "providersChecked": 5,
            "allProvidersFailed": true
        }
    },
    "traceId": "f7e8d9c0-1a2b-3c4d-5e6f-7a8b9c0d1e2f"
}
```

### 13.6 Error Response - Invalid Season/Episode

**Request**:

```http
GET /v1/tv/1396/seasons/99/episodes/500 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```http
HTTP/1.1 400 Bad Request
Content-Type: application/json; charset=utf-8
```

```json
{
    "error": {
        "code": "INVALID_EPISODE",
        "message": "Episode number is out of valid range for this season",
        "details": {
            "tmdbId": "1396",
            "season": 99,
            "episode": 500,
            "maxSeason": 5,
            "maxEpisodeForSeason": 16
        }
    },
    "traceId": "8b9c0d1e-2f3a-4b5c-6d7e-8f9a0b1c2d3e"
}
```

### 13.7 Proxy Request - HLS Manifest

**Request**:

```http
GET /v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%2C%22headers%22%3A%7B%22Referer%22%3A%22https%3A%2F%2Fprovider.com%22%7D%7D HTTP/1.1
Host: api.example.com
Accept: application/vnd.apple.mpegurl
```

**Decoded data parameter**:

```json
{
    "url": "https://cdn.example.com/stream.m3u8",
    "headers": {
        "Referer": "https://provider.com"
    }
}
```

**Response**:

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.apple.mpegurl
Cache-Control: public, max-age=300
X-Proxy-Cache: MISS
```

```m3u8
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1920x1080
/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%2C%22headers%22%3A%7B%22Referer%22%3A%22https%3A%2F%2Fprovider.com%22%7D%7D
#EXT-X-STREAM-INF:BANDWIDTH=2500000,RESOLUTION=1280x720
/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%2C%22headers%22%3A%7B%22Referer%22%3A%22https%3A%2F%2Fprovider.com%22%7D%7D
#EXT-X-STREAM-INF:BANDWIDTH=1000000,RESOLUTION=854x480
/v1/proxy?data=%7B%22url%22%3A%22https%3A%2F%2Fcdn.example.com%2Fstream.m3u8%22%2C%22headers%22%3A%7B%22Referer%22%3A%22https%3A%2F%2Fprovider.com%22%7D%7D
```

### 13.8 Home/Health Endpoint

**Request**:

```http
GET /v1 HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

```json
{
    "name": "My OMSS Backend",
    "version": "1.0.0",
    "status": "operational",
    "endpoints": {
        "movie": "/v1/movies/{id}",
        "tv": "/v1/tv/{id}/seasons/{s}/episodes/{e}",
        "proxy": "/v1/proxy/?data={encoded_data}"
    },
    "spec": "omss",
    "note": "Community-powered streaming backend. Visit https://github.com/mybackend for more info. Suggested frontend: https://omss-frontend.example.com"
}
```

### 13.9 Refresh Endpoint - Success

**Request**:

```http
GET /v1/refresh/cf6c3c2d-17be-4a5a-9488-bf12e70dca5a HTTP/1.1
Host: api.example.com
Accept: application/json
```

**Response**:

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

```json
{
    "status": "OK"
}
```
