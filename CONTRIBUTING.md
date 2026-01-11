# Contributing to OMSS

Thank you for your interest in contributing to the Open Media Streaming Specification! This document provides guidelines for contributing to the specification.

## 🎯 Ways to Contribute

### 1. Documentation Improvements
- Fix typos or clarify ambiguous language
- Add examples or use cases
- Improve formatting or structure

### 2. Specification Issues
- Report ambiguities or inconsistencies
- Suggest clarifications
- Identify edge cases not covered

### 3. Feature Proposals
- Propose new optional features
- Suggest extensions to the specification
- Submit RFCs for major changes

### 4. Implementation Feedback
- Share experiences implementing OMSS
- Report real-world challenges
- Suggest practical improvements

## 📋 Before You Start

1. **Check existing issues** - Someone may have already reported it
2. **Read the specification** - Understand current design decisions
3. **Join discussions** - Ask questions before major work
4. **Start small** - Begin with documentation or minor fixes

## 🔄 Contribution Process

### For Minor Changes (Documentation, Examples)

1. Fork the repository
2. Create a feature branch (`git checkout -b improve-examples`)
3. Make your changes
4. Commit with clear messages (`git commit -m "Add movie response example"`)
5. Push to your fork (`git push origin improve-examples`)
6. Open a Pull Request

### For Specification Changes

1. **Open an issue first** - Discuss the change before implementation
2. Explain the problem you're solving
3. Propose a solution
4. Get feedback from maintainers and community
5. If approved, follow the minor changes process above

### For Major Changes (New Features, Breaking Changes)

1. **Open an RFC (Request for Comments)** in Discussions
2. Use the RFC template (see below)
3. Allow at least 2 weeks for community feedback
4. Address concerns and iterate on design
5. After consensus, create a draft specification
6. Submit Pull Request with the full proposal

## 📝 RFC Template

```markdown
# RFC: [Feature Name]

## Summary
Brief description of the proposed change (2-3 sentences)

## Motivation
Why is this change needed? What problem does it solve?

## Detailed Design
Technical details of the proposed solution

## Drawbacks
What are the downsides or limitations?

## Alternatives
What other approaches were considered?

## Backward Compatibility
Does this break existing OMSS v1.0 implementations?

## Open Questions
What needs to be resolved before implementation?
```

## ✅ Pull Request Guidelines

### PR Checklist

- [ ] Clear, descriptive title
- [ ] Description explains what and why
- [ ] Related issues linked (if any)
- [ ] Examples updated (if applicable)
- [ ] OpenAPI spec updated (if changing API)
- [ ] Documentation updated
- [ ] No breaking changes (unless v2.0)

### PR Description Template

```markdown
## Description
[What does this PR do?]

## Motivation
[Why is this change needed?]

## Changes
- [List specific changes]

## Related Issues
Closes #[issue number]
```

### Code of Conduct

All contributors must follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## 🔍 Review Process

1. **Maintainer Review** - At least one maintainer approval required
2. **Community Feedback** - Allow time for community input
3. **CI Checks** - All automated checks must pass
4. **Merge** - Maintainer merges after approval

## 🎨 Style Guidelines

### Markdown
- Use ATX-style headers (`#` not `===`)
- One sentence per line in prose
- Use fenced code blocks with language tags
- Reference other docs with relative links

### OpenAPI YAML
- 2-space indentation
- Clear, descriptive `operationId`s
- Include examples for all schemas
- Document all parameters

### JSON Examples
- 4-space indentation
- Use realistic example data
- Include comments (when helpful)
- Validate against schemas

## 🏷️ Issue Labels

- `bug` - Something is wrong in the spec
- `documentation` - Docs improvements
- `feature` - New feature proposal
- `question` - Need clarification
- `breaking-change` - Would affect v1.0 compatibility

## 🚀 Release Process

1. All changes merged to `dev`
2. Version bump in specification files
3. Update CHANGELOG.md
4. Create GitHub release with notes
5. Announce in Discussions

## 💬 Communication Channels

- **GitHub Issues** - Bug reports, specific problems
- **GitHub Discussions** - Questions, ideas, RFCs
- **Pull Requests** - Code/doc contributions

## 🙏 Recognition

All contributors will be:
- Listed in release notes
- Credited in CONTRIBUTORS.md
- Thanked in release announcements

## 📚 Resources

- [Specification v1.0](spec/v1.0/omss-v1.0.md)
- [Implementation Guide](docs/implementation-guide.md)
- [Architecture Overview](docs/architecture.md)

## ❓ Questions?

If you're unsure about anything:
1. Check existing documentation
2. Search closed issues
3. Ask in Discussions
4. Open an issue

Thank you for contributing to OMSS! 🎉
