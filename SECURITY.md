# Security Policy

## Reporting Security Issues

The OMSS project takes security seriously. We appreciate your efforts to responsibly disclose any security vulnerabilities you discover.

### Scope

This security policy applies to:

- The OMSS specification itself (spec/v1.0/\*)
- Documentation that could affect implementation security
- Examples that might encourage insecure practices
- Tooling and validation scripts in this repository

> [!NOTE]
> This policy covers the **specification**, not specific implementations. For security issues in OMSS-compliant backends or frontends, please contact those projects directly.

---

## Reporting a Vulnerability

**Please do NOT report security vulnerabilities through public GitHub issues.**

Instead, please report security vulnerabilities by:

### Preferred Method: GitHub Security Advisories

1. Navigate to the [Security Advisories](https://github.com/omss-spec/omss-spec/security/advisories) page
2. Click "Report a vulnerability"
3. Fill out the form with details about the vulnerability
4. Submit the report

---

## What to Include in Your Report

Please include as much of the following information as possible:

- **Type of vulnerability** (e.g., specification ambiguity leading to security issues)
- **Affected version(s)** of the OMSS specification
- **Location** in the specification (file, section, line number)
- **Description** of the vulnerability
- **Potential impact** if exploited
- **Suggested fix** or mitigation (if you have one)
- **Example scenario** demonstrating the issue

### Example Report Structure

```markdown
## Vulnerability Type

Specification ambiguity leading to authentication bypass

## Affected Version

OMSS v1.0

## Location

spec/v1.0/omss-v1.0.md, Section 5.2 "Authentication"

## Description

The specification states that authentication is "optional" but doesn't
clarify whether optional means "not required by spec" or "implementation
choice". This could lead backends to incorrectly assume no authentication
is needed for production deployments.

## Impact

Backends might deploy without authentication, exposing streaming sources
and user data publicly.

## Suggested Fix

Clarify that authentication is "optional in the specification" but
"strongly recommended for production" and add a security considerations
section.
```

---

## Response Timeline

- **Initial Response**: Within 72 hours (acknowledgment of report)
- **Assessment**: Within 7 days (initial evaluation and severity assessment)
- **Resolution Timeline**:
  - Critical: Within 14 days
  - High: Within 30 days
  - Medium: Within 60 days
  - Low: Next minor release

We will keep you informed of progress throughout the resolution process.

---

## Supported Versions

| Version | Supported          | Status      |
| ------- | ------------------ | ----------- |
| 1.0.x   | :white_check_mark: | Active      |
| < 1.0   | :x:                | Development |

We only accept security reports for released versions of the specification.

---

## Public Disclosure

- We follow **coordinated disclosure** practices
- Security vulnerabilities will be disclosed publicly only after:

  1. A fix has been released (or workaround documented)
  2. Affected implementations have had reasonable time to update
  3. At least 90 days have passed since the fix was released

- Reporters will be credited in:
  - Security advisories
  - CHANGELOG.md
  - Release notes

---

## Questions?

For questions about this security policy, please:

- Open a [GitHub Discussion](https://github.com/omss-spec/omss-spec/discussions)
- Review our [Contributing Guidelines](CONTRIBUTING.md)

**Thank you for helping keep OMSS secure!** 🔒
