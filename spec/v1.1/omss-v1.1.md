# OMSS v1.1 Specification

**Open Media Streaming Specification**

Version: 1.1.0
Status: Final
Released: June 9th, 2026
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
10. [Versioning and Compatibility](#10-versioning-and-compatibility)
11. [Complete Examples](#11-complete-examples)
12. [License](#12-license)

---

## 1. Introduction

### 1.1 Purpose

OMSS (Open Media Streaming Specification) v1.1 defines a unified REST API standard for streaming media backends. It enables any frontend application to work seamlessly with any OMSS-compliant backend through standardized endpoints for retrieving streaming sources.

### 1.2 Problem Statement

Modern streaming backends scrape data from various providers with inconsistent formats:

- **No standardized metadata**: Files may be MP4, HLS or MKV without clear type information.
- **Missing quality information**: Bitrate, resolution, and codec data are often unavailable.
- **Language ambiguity**: Audio/subtitle language tracks may be unlabeled.
- **Ephemeral URLs**: Signed URLs with expiration times require special handling.
- **Cross-origin restrictions**: Sources might require proxy handling for CORS and access control.

**OMSS standardizes streaming sources** while acknowledging these real-world constraints.

### 1.3 Design Philosophy

- **Source-focused**: OMSS is about standardizing streaming sources, not media metadata.
- **Pragmatic**: Designed for backends that scrape third-party providers with incomplete data.
- **Progressive disclosure**: Required fields are minimal; optional fields provide enhanced functionality.
- **HTTP-first**: Uses HTTP status codes, headers, and semantics correctly.

### 1.4 Scope

**In Scope:**

- REST API endpoint definitions for source retrieval.
- Standardized source/subtitle schemas.
- Error handling patterns.

**Out of Scope:**

- Access control/authentication — implement that yourself. (Personal Note: I would not recommend hosting such a backend publicly anyway, since it brings risk and costs with it. For personal use you do not need these.)
- Media metadata (ratings, descriptions, posters, etc.) — use the TMDB API directly.
- Search functionality — use the TMDB API.
- Provider-specific scraping logic — implement that yourself.
- Video player implementation — whichever player you use.

### 1.5 TMDB Integration

OMSS uses **TMDB (The Movie Database) as the primary identifier** for all media content:

- **Frontend responsibility**: Fetch media metadata (title, year, poster, description, ratings) from the TMDB API.
- **OMSS responsibility**: Provide streaming sources and subtitles for TMDB IDs.
- **Clean separation**: OMSS focuses exclusively on source standardization.

That is because currently it seems that TMDB is the only decent API provider that is free, has a large database and a stable API.
Supporting identifiers from multiple providers like anilist, trakt, or Sport Events etc. would add complexity and since these API's aren't really stable, also add a lot of maintenance work for backend developers, without really providing significant benefits to the end users.

---

## 2. Core Principles

### 2.1 HTTP-First Design

- **Primary signal**: HTTP status codes indicate success or failure.
- **Secondary signal**: The response body provides details.
- **Standard headers**: Use `Cache-Control`, `ETag`, `Retry-After`, etc.

### 2.2 Unknown Data Handling

Backends MUST handle incomplete provider data gracefully:

- **Missing metadata**: Omit optional fields rather than returning null or empty values.
- **Quality and language inference**: Attempt to infer quality and language from filenames, manifests, resolution, or bitrate.
- **Type detection**: Analyze file extension, MIME type, or content headers.

### 2.3 Identifier Strategy

- **Primary**: TMDB (The Movie Database) IDs.
- **Required**: All media MUST have a TMDB ID.

### 2.4 Versioning

- **URL-based**: `/v1/`, `/v2/` in the path.
- **Semantic versioning**: `MAJOR.MINOR.PATCH`.
- **Breaking changes**: Require a new major version.

---

## 3. API Conventions

### 3.1 Base URL

```text
https://api.example.com/
```

All endpoints are relative to this base URL. From here on, all absolute paths are meant to be relative, starting from the web root.
Clients should support non-root deployments, so the base URL can be configured as needed (e.g., `https://api.example.com/omss/`), but for simplicity, the specification assumes a root deployment.

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

OMSS v1.1 supports only JSON responses.

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
    "version": "1.1.0",
    "status": "operational",
    "endpoints": {
        "movie": "/v1/movies/{id}",
        "tv": "/v1/tv/{id}/seasons/{s}/episodes/{e}"
    },
    "spec": "omss",
    "note": "Here some description, motivation, disclaimer or anything else. Here you could also link your suggested OMSS frontend.",
    "media": {
        "movies": [123, 456],
        "tv": [
            {
                "id": 1396,
                "seasons": [
                    {
                        "season": 1,
                        "episodes": [1, 2, 3]
                    },
                    {
                        "season": 2,
                        "episodes": [1, 2, 3, 4, 5]
                    }
                ]
            }
        ]
    },
    "providers": [
        { "id": "alpha", "name": "Provider Alpha", "capabilities": ["movies", "tv"] },
        { "id": "beta", "name": "Provider Beta", "capabilities": ["movies"] }
    ]
}
```

**Required Fields:**

- `name` (string): Implementation name.
- `version` (string): Semantic version **of the OMSS implementation** (e.g., `1.1.0`).
- `status` (string): API status — `operational`, `degraded`, or `down` (e.g. maintenance).
- `endpoints` (object): Functions mapped to paths.
    - `movie` (string): Path to the movie endpoint. MUST contain the `{id}` placeholder where the frontend will insert the TMDB ID.
    - `tv` (string): Path to the TV show endpoint. MUST contain the `{id}`, `{s}` and `{e}` placeholders where the frontend will insert the TMDB ID, season and episode respectively.
- `spec` (string): When implementing OMSS, this MUST be `"omss"`.
- `media` (object): field for backends to provide a list of available media.
    - `movies` (array of integers OR string: '_' as const): List of TMDB movie IDs available in this backend or `'_'` if the backend supports all movies.
    - `tv` (array of objects or string: '_' as const): List of TMDB TV series IDs with available seasons and episodes, or `'_'`if the backend supports all TV series. You can use`'\*'` inside of the tv object to mark a specific series and/or season as available. Each object contains:
        - `id` (integer): TMDB series ID.
        - `seasons` (array of objects): List of seasons with available episodes. Each object contains:
            - `season` (integer): Season number.
            - `episodes` (array of integers): List of episode numbers available in this season.
- `providers` (array of objects): List of providers this backend provides. Each object contains:
    - `id` (string): Unique provider identifier (MUST BE URL SAFE).
    - `name` (string): Human-readable provider name.
    - `capabilities` (array of strings): List of media types the provider supports any of `movies`, `tv` and/or `subtitles`.

**Optional Fields:**

- `note` (string): A custom message by the backend.

### 4.2 Movie Sources

**Purpose**: Retrieve movie streaming sources by TMDB ID.

```http
GET /v1/movies/{id}?platform=web
GET /v1/movies/{id}?platform=native
GET /v1/movies/{id}?platform=web&provider={provider_id}
```

**Path Parameters:**

- `id` (string, required): TMDB movie ID.

**Query Parameters:**

- `platform` (string, optional): Target platform for source optimization. One of `web`, `native`. Default is `web`. **This will change the response structure!**
- `provider` (string, optional): Specific provider to fetch sources from. MUST be a valid provider ID. The result will stay the same, but the backend shall only return sources from the specified provider.

**Example Request:**

```http
GET /v1/movies/155?platform=web
GET /v1/movies/155?platform=native
GET /v1/movies/155?platform=web&provider=alpha
GET /v1/movies/155?provider=alpha
```

**Response: 200 OK** (see [6.1 Success Response](#61-success-response))

### 4.3 TV Episode Sources

**Purpose**: Retrieve TV episode streaming sources.

```http
GET /v1/tv/{id}/seasons/{s}/episodes/{e}?platform=web
GET /v1/tv/{id}/seasons/{s}/episodes/{e}?platform=native
GET /v1/tv/{id}/seasons/{s}/episodes/{e}?platform=web&provider={provider_id}
```

**Path Parameters:**

- `id` (string, required): TMDB series ID.
- `s` (integer, required): Season number (≥ 0).
- `e` (integer, required): Episode number (≥ 1).

**Query Parameters:**

- `platform` (string, optional): Target platform for source optimization. One of `web`, `native`. Default is `web`. **This will change the response structure!**
- `provider` (string, optional): Specific provider to fetch sources from. MUST be a valid provider ID. The result will stay the same, but the backend shall only return sources from the specified provider.

**Example Requests:**

```http
GET /v1/tv/1396/seasons/1/episodes/1?platform=web
GET /v1/tv/1396/seasons/5/episodes/14?platform=native
GET /v1/tv/1396/seasons/1/episodes/1?platform=web&provider=alpha
```

**Response: 200 OK** (see [6.1 Success Response](#61-success-response))

### 4.4 Proxy Endpoint (deprecated)

In previous versions of OMSS, the spec used to dictate the proxy behaviour. Starting v1.1, that isn't the case anymore. It is up to the backend to decide how to/if it wants to implement the proxy.

### 4.5 Refresh Endpoint

**Purpose**: Unified endpoint to refresh a response by its id.

```http
POST /v1/refresh/{id}
```

**Path Parameters:**

- `id` (string, required): The ID of the response to refresh.

**Response Object Schema:**

```json
{
    "status": "OK"
}
```

**Example Request:**

```http
POST /v1/refresh/cf6c3c2d-17be-4a5a-9488-bf12e70dca5a
```

**Response: 200 OK**

If the `id` is valid the response will always be `200 OK`, since the backend should simply remove the cache entry and immediatley invalidate the id and will try to fetch a new source the next time the frontend requests it. If it is invalid, see [7.3 Standard Error Codes](#73-standard-error-codes).

---

## 5. Request Specifications

### 5.1 Headers

**Required:**

- `Accept: application/json; charset=utf-8`

### 5.2 Query Parameters

**Validation Rules:**

- Query parameters are **lowercase**.
- Boolean parameters: `true` or `false` (lowercase).

### 5.3 Path Parameters

**Validation Rules:**

- `id`: String (numeric characters ONLY, treated as a string).
- `s`: Non-negative integer (0–∞ range).
- `e`: Positive integer (≥ 1).

---

## 6. Response Specifications

### 6.1 Success Response

Both the movie and TV episode endpoints share this response structure, which includes arrays of Source and Subtitle objects, as well as any diagnostics about data quality or scraping issues.

The response structure will stay the same regardless of the `provider` query parameter, as it only returns sources from the specified provider. If the `provider` parameter is not specified, sources from all providers will be returned. If the specified provider does not return any sources, the response shall throw a `NO_SOURCES_AVAILABLE` error (see [7.4.1 Media Exists, No Sources](#741-media-exists-no-sources)). If the `provider` parameter is specified but does not match any known provider, the response shall throw a `PROVIDER_NOT_FOUND` error (see [7.3 Standard Error Codes](#73-standard-error-codes)).

However, the `platform` query parameter **will** change the structure of the [6.2 Source Objects](#62-source-object) and [6.3 Subtitle Objects](#63-subtitle-object) in the response, as described in their respective sections.

**HTTP Status: 200 OK**

```json
{
    "id": "bdfa40a7-a468-461c-8563-7a0c165f252c",
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

- `id` (string): Unique response identifier (e.g. UUID v4).
- `sources` (array): Array of Source objects (at least one required. if no source found, see [7.4.1](#741-media-exists-no-sources)).
- `subtitles` (array): Array of Subtitle objects (may be empty).
- `diagnostics` (array): Non-fatal warnings/errors about data quality/inference and provider issues (may be empty).
- `expiresAt` (string, optional): ISO 8601 timestamp when sources expire (for signed/temporary URLs).

---

### 6.2 Source Object

**Purpose**: Represents a streaming source with quality and technical metadata.

**Usage**: If the source object is found in the sources array, it means that the source is streamable and can be used directly in video players. If it is found in the downloads array, it means that the source is NOT streamable and should be treated as a direct download link. Clients should not attempt to use download links as streaming sources, as they may quite literally break your backend, because your backend might try to download the whole file into it's memory before sending it to the client, which can cause out-of-memory errors and crashes.

**Platform**:

The structure of the Source Object can differ based on the `platform` query parameter in the request. **For `web`, the `url` must be a playable streaming link (e.g., HLS, MP4) that can be used directly in HTML5 video players (i.e. without CORS or any other restrictions)**. **For `native`, the `url` has to be the upstream URL and a new `headers` (object) field will be added**. This allows native player implementations that can handle CORS and custom headers to access the original streaming URL from the provider, while web clients will receive a proxied/optimized URL that is accessible without additional headers or CORS restrictions in browsers.

Default: `web`.

**?platform=web Example**

```json
{
    "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
    "url": "https://localhost:3000/playable/stream/endpoint.m3u8",
    "streamable": true,
    "type": "hls",
    "quality": "FHD",
    "audioTracks": ["Original", "English", "Spanish"],
    "provider": {
        "id": "provider_123",
        "name": "Example Provider"
    }
}
```

**?platform=native&provider=provider_123 Example**

```json ?platform=native
{
    "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
    "url": "https://upstream.provider.com/locked/stream/endpoint.m3u8",
    "headers": {
        "Referer": "https://upstream.provider.com/watch/locked/stream",
        "more": "custom headers as needed"
    },
    "streamable": true,
    "type": "hls",
    "quality": "FHD",
    "audioTracks": ["Original", "English", "Spanish"],
    "provider": {
        "id": "provider_123",
        "name": "Example Provider"
    }
}
```

#### 6.2.1 Required Fields

- **`id`** (string): Unique source identifier (e.g. UUID v4).
- **`url`** (string): For `web`: A string representing the URL to the streaming source that can be used directly in HTML5 video players without CORS issues. For `native`: A string representing the original URL to the streaming source from the provider, which may require CORS handling or custom headers (provided in the `headers` field).
- **`headers`** (object, only for `native` platform): Key-value pairs of HTTP (and non-standard HTTP) headers that should be included when accessing the `url` for native player implementations that can handle CORS and custom headers. For `web` platform, this field is not included, and the provided `url` must be accessible without additional headers or CORS restrictions without additional headers.

- **`streamable`** (boolean): Indicates if the source is streamable (true) or a direct download link (false). If false, clients must treat the URL as a download link rather than a streaming source. Download links CANNOT be used as streaming sources. ([see Range Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Range#:~:text=A%20server%20that%20doesn%27t%20support%20range%20requests%20may%20ignore%20the%20Range%20header%20and%20return%20the%20whole%20resource%20with%20a%20200%20status%20code.))

- **`type`** (string): Source type, one of:
    - `hls` — HTTP Live Streaming (M3U8).
    - `mp4` — MP4 file.
    - `mkv` — MKV file.

- **`quality`** (string): Video quality. One of 8K, 4K, 2K, FHD, HD, SD, `Auto`. default/unknown --> `Auto`
  Quality should be estimated based on available metadata such as bitrate, resolution, or inferred from the filename or manifest. If no quality information is available, use `Auto`. For example, 320p --> SD, 720p --> HD, 1080p --> FHD, 2160p --> 4K, 4320p --> 8K.

- **`audioTracks`** (array of strings): Human-readable language name. default/unknown --> `Original`

- **`provider`** (object): Information about the upstream provider.
    - `id` (string): Unique provider identifier.
    - `name` (string): Human-readable provider name.

#### 6.2.2 Handling Unknown Source Data

Backends must handle missing or incomplete provider data to the best of their ability, while ensuring that the response remains valid and usable for clients.

#### 6.2.3 Multi-Audio Handling

It is the client's responsibility to handle multiple audio tracks. The backend should provide an `audioTracks` array with human-readable language names.
For backends: If the language cannot be determined, use `"Original"`.

---

### 6.3 Subtitle Object

**Purpose**: Represents subtitle/caption tracks.

**Platform**:

The structure of the Subtitle Object can differ based on the `platform` query parameter in the request.
**For `web`, the `url` must be a direct link to a subtitle file (e.g., VTT, SRT) that can be used directly in HTML5 track elements without CORS issues**. **For `native`, the `url` has to be the upstream URL and a new `headers` (object) field will be added**. This allows native player implementations that can handle CORS and custom headers to access the original subtitle URL from the provider, while web clients will receive a proxied/optimized URL that is accessible without additional headers or CORS restrictions in browsers.
Default: `web`.

**?platform=web Example**

```json
{
    "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
    "url": "https://localhost:3000/playable/subtitles/endpoint.vtt",
    "label": "English",
    "format": "vtt",
    "provider": {
        "id": "provider_123",
        "name": "Example Provider"
    }
}
```

**?platform=native&provider=provider_123 Example**

```json
{
    "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
    "url": "https://provider.com/subs/blocked/abc1234.vtt",
    "headers": {
        "Referer": "https://provider.com/download/abcd1234",
        "more": "custom headers as needed"
    },
    "label": "English",
    "format": "vtt",
    "provider": {
        "id": "provider_123",
        "name": "Example Provider"
    }
}
```

#### 6.3.1 Required Fields

- **`id`** (string): Unique subtitle identifier (e.g. UUID v4).

- **`url`** (string): For `web`: A string representing the URL to the subtitle file that can be used directly in HTML5 track elements without CORS issues. For `native`: A string representing the original URL to the subtitle file from the provider, which may require CORS handling or custom headers (provided in the `headers` field).

- **`headers`** (object, only for `native` platform): Key-value pairs of HTTP (and non-standard HTTP) headers that should be included when accessing the `url` for native player implementations that can handle CORS and custom headers. For `web` platform, this field is not included, and the provided `url` must be accessible without additional headers or CORS restrictions without additional headers.

- **`label`** (string): Human-readable language name for the subtitle track. default/unknown --> `Unknown`

- **`format`** (string): Subtitle format, one of:
    - `vtt` — WebVTT (preferred for web; default).
    - `srt` — SubRip.

- **`provider`** (object): Information about the upstream provider.
    - `id` (string): Unique provider identifier.
    - `name` (string): Human-readable provider name.

#### 6.3.3 Handling Unknown Subtitle Data

Backends must handle missing or incomplete subtitle data to the best of their ability, while ensuring that the response remains valid and usable for clients.

---

### 6.4 Warning/Error Objects in Success Response

**Purpose**: Report non-response-fatal issues encountered during scraping. If `source` is an empty string, it means the warning applies to the entire response.

**Description**: A non-response-fatal warning/error is an issue encountered during scraping that does not prevent the backend from returning a successful response. These warnings/errors provide additional context about potential data quality issues or provider-specific scraping failures (for example: `"Provider [name] failed to respond"` would be a non-response-fatal error).

> [!NOTE]
> If the `provider` parameter is specified in the request, but the backend fails to fetch sources from that specific provider, the response will not include a warning/error object in the `diagnostics` array, but will instead return a `NO_SOURCES_AVAILABLE` error response (see [7.4.1 Media Exists, No Sources](#741-media-exists-no-sources)).

```json
{
    "code": "PROVIDER_ERROR",
    "message": "Provider [name] failed to respond",
    "source": "provider_123",
    "severity": "error"
}
```

**Fields:**

- **`code`** (string): Machine-readable warning/error code.
- **`message`** (string): Human-readable description.
- **`source`** (string): Identifier of the source of the issue (e.g., provider ID). If empty, applies to the whole response.
- **`severity`** (string): One of:
    - `warning` — Non-fatal issue.
    - `error` — Fatal issue.

#### 6.4.1 Standard Warning/Error Codes

| Error/Warning Code | Description                                          | Severity |
| ------------------ | ---------------------------------------------------- | -------- |
| `PROVIDER_ERROR`   | An error occurred while fetching from the provider.  | error    |
| `PARTIAL_SCRAPE`   | A provider did respond, but not to it's full extent. | warning  |

---

## 7. Error Handling

### 7.1 Error Response Format

**All non-2xx responses MUST include:**

```json
{
    "error": {
        "code": "INVALID_TMDB_ID",
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

- **`error.details`** (any): Additional context about the error. Frontends should not display this.

### 7.3 Standard Error Codes

| Error Code               | HTTP Status | Description                              |
| ------------------------ | ----------- | ---------------------------------------- |
| `INVALID_TMDB_ID`        | 400         | Invalid (i.e., fictional ID's) TMDB ID   |
| `INVALID_PARAMETER`      | 400         | Invalid query/path parameter.            |
| `MISSING_PARAMETER`      | 400         | Required parameter missing.              |
| `INVALID_SEASON`         | 400         | Season number out of valid range.        |
| `INVALID_EPISODE`        | 400         | Episode number invalid.                  |
| `INVALID_RESPONSE_ID`    | 400         | Invalid responseId format.               |
| `PROVIDER_NOT_FOUND`     | 404         | No provider found with the specified ID. |
| `RESPONSE_ID_NOT_FOUND`  | 404         | responseId not found for refresh.        |
| `NO_SOURCES_AVAILABLE`   | 404         | Media exists but no sources were found.  |
| `ENDPOINT_NOT_FOUND`     | 404         | Invalid endpoint path.                   |
| `METHOD_NOT_ALLOWED`     | 405         | HTTP method not supported.               |
| `INTERNAL_ERROR`         | 500         | Unexpected server error.                 |
| `UNSUPPORTED_MEDIA_TYPE` | 415         | Unsupported Content-Type in request.     |

> [!NOTE]
> `INVALID_TMDB_ID` should be used, when the TMDB ID is valid in format but does not correspond to any media (e.g., `99999999`). For IDs that are not even valid in format (e.g., `abc123`), `INVALID_PARAMETER` should be used instead.

### 7.4 Edge Cases

#### 7.4.1 Media Exists, No Sources

**HTTP Status: 404 Not Found**

When the TMDB ID is valid but no streaming sources are available for that media, the backend MUST return a `404 Not Found` status with a `NO_SOURCES_AVAILABLE` error code. This indicates that while the media exists, the backend was unable to find any streamable sources for it (applies to a failure when fetching sources from a specific provider).

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
        "code": "INVALID_PARAMETER",
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
        "code": "INVALID_TMDB_ID",
        "message": "No media found with the provided TMDB ID"
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
- When the refresh endpoint is called, the cache, aswell as the id, for that specific source should be invalidated immediately.
- For large/public backends, consider caching with a CDN and browser caching headers.

### 8.5 Performance Recommendations

**For Backends:**

- Implement request coalescing for concurrent identical requests.
- Use connection pooling for upstream providers.
- Implement **circuit breakers** for failing providers.

**For Clients:**

- Respect cache headers and ETags.
- Prefetch sources before playback (for example at the end of a episode).
- Use conditional requests (`If-None-Match`).

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
    - Reject injection attempts.

3. **Headers**:
    - Enforce size limits (to prevent header overflow attacks).

---

## 10. Versioning and Compatibility

### 10.1 Semantic Versioning

OMSS follows Semantic Versioning 1.0.0:

```text
MAJOR.MINOR.PATCH
  1 . 1 . 0
```

- **MAJOR**: Breaking changes (incompatible API changes).
- **MINOR**: Backward-compatible new features.
- **PATCH**: Backward-compatible bug fixes/clarifications.

### 10.2 URL Versioning

Version is in the URL path:

```text
/v1/movies/155   # Version 1.x
/v2/movies/155   # Version 2.x (future)
```

### 10.4 Deprecation Policy

This is best effort, but especially in the beginning, this might not be followed strictly, since the spec is still evolving and breaking changes are expected. However, once the spec stabilizes, the following deprecation policy should be followed:

1. **Announce** at least 6 months before removal.
2. **Headers**: Deprecated endpoints return:

    ```text
    Deprecation: true
    Sunset: Wed, 01 Jul 2026 23:59:59 GMT
    ```

3. **Documentation**: Migration guides are provided.
4. **Support**: Deprecated features are supported for ≥ 1 major version.
5. **Removal**: Only in a new major version.

### 10.5 Client Compatibility Expectations

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

## 11. Complete Examples

### 11.1 Movie Request — Web Platform (Default)

**Request**

```http
GET /v1/movies/155
Accept: application/json
```

Since the `platform` query parameter is omitted, the response uses the default `web` structure. All `url` fields are accessible without additional headers or CORS restrictions directly in browsers without requiring custom headers or proxy handling by the client.

For examples when requesting a specific provider, see above in the [4.2 Movie Sources](#42-movie-sources) section.

**Response: 200 OK**

```json
{
    "id": "bdfa40a7-a468-461c-8563-7a0c165f252c",
    "expiresAt": "2026-01-11T20:56:00Z",
    "sources": [
        {
            "id": "cf6c3c2d-17be-4a5a-9488-bf12e70dca5a",
            "url": "https://api.example.com/playable/source/1/master.m3u8",
            "streamable": true,
            "type": "hls",
            "quality": "4K",
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
}
```

---

### 11.2 TV Episode Request — Native Platform

**Request**

```http
GET /v1/tv/1396/seasons/1/episodes/1?platform=native
Accept: application/json
```

For the `native` platform, the backend returns upstream URLs directly and includes the required request headers for native player implementations.

**Response: 200 OK**

```json
{
    "id": "ad2e6d83-ef14-42de-9f3d-88d4d0c60b6f",
    "expiresAt": "2026-01-11T21:15:00Z",
    "sources": [
        {
            "id": "f54cdb9f-f7dd-4f53-8456-d31a94f27ef1",
            "url": "https://upstream.provider.com/stream/master.m3u8",
            "headers": {
                "Referer": "https://upstream.provider.com/watch/1396-1-1",
                "User-Agent": "ExampleNativePlayer/1.0"
            },
            "streamable": true,
            "type": "hls",
            "quality": "HD",
            "audioTracks": ["Original", "English", "German"],
            "provider": {
                "id": "provider_streamhub",
                "name": "StreamHub"
            }
        }
    ],
    "subtitles": [
        {
            "id": "ee7d8e7f-8f6f-4d66-8c71-9d5f50a2df17",
            "url": "https://subs.provider.com/1396-s1-e1-en.vtt",
            "headers": {
                "Referer": "https://subs.provider.com"
            },
            "label": "English",
            "format": "vtt",
            "provider": {
                "id": "provider_streamhub",
                "name": "StreamHub"
            }
        }
    ],
    "diagnostics": []
}
```

---

### 11.3 Media Exists but No Sources Available

**Request**

```http
GET /v1/movies/999999
Accept: application/json
```

The TMDB ID is valid, but the backend could not find any playable or downloadable sources.

**Response: 404 Not Found**

```json
{
    "error": {
        "code": "NO_SOURCES_AVAILABLE",
        "message": "No streaming sources found for TMDB ID: 999999"
    },
    "traceId": "f4f44c59-6e53-4f5d-a53d-c6d6cbeff5f"
}
```

---

### 11.4 Invalid TMDB ID Format

**Request**

```http
GET /v1/movies/not-a-number
Accept: application/json
```

TMDB IDs must contain numeric characters only.

**Response: 400 Bad Request**

```json
{
    "error": {
        "code": "INVALID_PARAMETER",
        "message": "TMDB ID must contain numeric characters only",
        "details": {
            "parameter": "id",
            "value": "not-a-number"
        }
    },
    "traceId": "fcd0ef6c-54b4-49e5-9f67-b5aefcb76f9e"
}
```

---

### 11.5 Unsupported Accept Header

**Request**

```http
GET /v1/movies/155
Accept: application/xml
```

OMSS v1.1 supports JSON responses only.

**Response: 406 Not Acceptable**

```json
{
    "error": {
        "code": "NOT_ACCEPTABLE",
        "message": "Only application/json responses are supported"
    },
    "traceId": "78fa61ab-5b89-4984-ae91-78382f9242d8"
}
```

---

### 11.6 Refresh Endpoint — Success

**Request**

```http
POST /v1/refresh/cf6c3c2d-17be-4a5a-9488-bf12e70dca5a
Accept: application/json
```

The backend invalidates the cached source immediately. The next request for this media will trigger a fresh scrape.

**Response: 200 OK**

```json
{
    "status": "OK"
}
```

---

### 11.7 Refresh Endpoint — Unknown ID

**Request**

```http
POST /v1/refresh/unknown-id
Accept: application/json
```

The provided response ID does not exist or has already expired.

**Response: 404 Not Found**

```json
{
    "error": {
        "code": "RESPONSE_ID_NOT_FOUND",
        "message": "No cached response found for the provided ID"
    },
    "traceId": "1f5dff55-b0ec-4a74-87ec-b5b6e8bc52a7"
}
```

---

### 11.8 Root Endpoint

**Request**

```http
GET /
Accept: application/json
```

The root endpoint provides general information about the OMSS backend implementation and available endpoints.

**Response: 200 OK**

```json
{
    "name": "Example OMSS Implementation",
    "version": "1.1.0",
    "status": "operational",
    "spec": "omss",
    "note": "OMSS backend is running",
    "endpoints": {
        "movie": "/v1/movies/{id}",
        "tv": "/v1/tv/{id}/seasons/{s}/episodes/{e}"
    },
    "media": {
        "movies": "*",
        "tv": "*"
    },
    "providers": [
        {
            "id": "provider_alpha",
            "name": "Provider Alpha",
            "capabilities": ["movies", "tv"]
        },
        {
            "id": "provider_beta",
            "name": "Provider Beta",
            "capabilities": ["movies"]
        }
    ]
}
```

---

### 11.9 Versioned Root Endpoint with Limited Media Availability

**Request**

```http
GET /v1
Accept: application/json
```

This example shows a backend that exposes only a limited set of available movies and TV episodes.

**Response: 200 OK**

```json
{
    "name": "Limited Example Backend",
    "version": "1.1.0",
    "status": "operational",
    "spec": "omss",
    "endpoints": {
        "movie": "/v1/movies/{id}",
        "tv": "/v1/tv/{id}/seasons/{s}/episodes/{e}"
    },
    "note": "This backend exposes only selected media.",
    "media": {
        "movies": [123, 456],
        "tv": [
            {
                "id": 1396,
                "seasons": [
                    {
                        "season": 1,
                        "episodes": [1, 2, 3]
                    },
                    {
                        "season": 2,
                        "episodes": [1, 2, 3, 4, 5]
                    }
                ]
            }
        ]
    },
    "providers": [
        {
            "id": "provider_alpha",
            "name": "Provider Alpha",
            "capabilities": ["movies", "tv"]
        },
        {
            "id": "provider_beta",
            "name": "Provider Beta",
            "capabilities": ["movies"]
        }
    ]
}
```

---

## 12. License

This specification is licensed under the MIT License.
