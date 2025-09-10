---
issue: 5
title: RSS Parser Service
type: development
decomposed: 2025-09-10T13:05:45Z
estimated_hours: 10
---

# Development Guide: Issue #5

## Overview

Implement a robust RSS/Atom feed parsing service that can handle various feed formats and normalize them into a consistent internal structure. The service will integrate with the existing Next.js application using tRPC for API endpoints and Prisma for data persistence.

## Prerequisites

- [ ] Local task file reviewed: `.claude/epics/ai-info-aggregation/5.md`
- [ ] GitHub issue understood: `gh issue view 5`
- [ ] Development environment set up (`pnpm install` completed)
- [ ] Database schema understood (Prisma models for Source, Article, Category)

## Development Task List

### Core Implementation Tasks

#### Task 1: XML Parsing Library Setup and Feed Detection

**Description**: Set up XML parsing infrastructure and implement feed format detection
**Files to modify/create**:

- `ai-info/package.json` - Add xml parsing dependency (fast-xml-parser or xml2js)
- `ai-info/src/services/rss/types.ts` - Define TypeScript interfaces for RSS/Atom structures
- `ai-info/src/services/rss/detector.ts` - Implement RSS vs Atom format detection logic

**Context needed**:

- Understanding of RSS 2.0 and Atom 1.0 specification differences
- XML namespace handling for Atom feeds
- Root element detection patterns (<rss>, <feed>)

**Estimated Hours**: 2
**Priority**: High
**Dependencies**: None

#### Task 2: RSS 2.0 Parser Implementation

**Description**: Implement parser for RSS 2.0 format with proper field extraction
**Files to modify/create**:

- `ai-info/src/services/rss/parsers/rss-parser.ts` - RSS 2.0 specific parsing logic
- `ai-info/src/services/rss/parsers/index.ts` - Export parser implementations

**Context needed**:

- RSS 2.0 channel and item structure
- CDATA section handling
- Dublin Core namespace extensions
- Media RSS extensions (if applicable)

**Estimated Hours**: 3
**Priority**: High
**Dependencies**: Task 1

#### Task 3: Atom 1.0 Parser Implementation

**Description**: Implement parser for Atom 1.0 format with namespace handling
**Files to modify/create**:

- `ai-info/src/services/rss/parsers/atom-parser.ts` - Atom 1.0 specific parsing logic

**Context needed**:

- Atom 1.0 feed and entry structure
- XML namespace declarations
- Multiple link relations handling
- Author vs contributor handling

**Estimated Hours**: 3
**Priority**: High
**Dependencies**: Task 1

#### Task 4: Article Normalization Service

**Description**: Create service to normalize parsed articles into standardized format
**Files to modify/create**:

- `ai-info/src/services/rss/normalizer.ts` - Article normalization logic
- `ai-info/src/services/rss/schemas.ts` - Zod schemas for validated output

**Context needed**:

- Existing Article Prisma model structure
- Date/timestamp normalization patterns
- Content sanitization requirements
- Author field standardization

**Estimated Hours**: 1.5
**Priority**: High
**Dependencies**: Tasks 2, 3

#### Task 5: Main RSS Service Implementation

**Description**: Create main service class orchestrating parsing pipeline
**Files to modify/create**:

- `ai-info/src/services/rss/rss-service.ts` - Main service class with public API
- `ai-info/src/services/rss/index.ts` - Service exports

**Context needed**:

- Error handling patterns in existing codebase
- Encoding detection and conversion logic
- Malformed XML graceful handling
- Service instantiation patterns

**Estimated Hours**: 2
**Priority**: High
**Dependencies**: Tasks 1, 4

### Integration Tasks

#### Task 6: tRPC Endpoint Implementation

**Description**: Add tRPC procedures for RSS parsing functionality
**Files to modify/create**:

- `ai-info/src/pages/api/trpc/[trpc].ts` - Add RSS parsing procedures

**Context needed**:

- Existing tRPC procedure patterns
- Input validation with Zod schemas
- Error handling for tRPC procedures
- Authentication/authorization requirements (if any)

**Estimated Hours**: 1
**Priority**: Medium
**Dependencies**: Task 5

#### Task 7: Error Handling Framework Integration

**Description**: Implement comprehensive error handling for feed parsing
**Files to modify/create**:

- `ai-info/src/services/rss/errors.ts` - Custom error classes for RSS parsing
- Update main service to use error framework

**Context needed**:

- Existing error handling patterns in codebase
- Error logging strategies
- User-facing error message patterns
- Error categorization (network, parsing, validation)

**Estimated Hours**: 1.5
**Priority**: Medium
**Dependencies**: Task 5

## Task Dependencies

### Development Task Flow

1. **Setup Phase**: Task 1 (XML parsing setup)
2. **Core Parsing Phase**: Tasks 2, 3 (can be done in parallel)
3. **Normalization Phase**: Task 4 (depends on Tasks 2, 3)
4. **Service Integration**: Task 5 (depends on Task 4)
5. **API Integration**: Task 6 (depends on Task 5)
6. **Error Handling**: Task 7 (can be done in parallel with Task 6)

## Definition of Done

### Development Tasks Complete When:

- [ ] RSS 2.0 feeds parse successfully with all required fields extracted
- [ ] Atom 1.0 feeds parse successfully with all required fields extracted
- [ ] Malformed XML handled gracefully with descriptive error messages
- [ ] Encoding issues (UTF-8, ISO-8859-1) handled properly
- [ ] CDATA sections parsed correctly
- [ ] Timestamps normalized to consistent ISO format
- [ ] Article objects match expected schema structure
- [ ] tRPC endpoints created and functional
- [ ] Code follows existing patterns and TypeScript best practices
- [ ] No lint/type errors (`pnpm lint` passes)
- [ ] Basic functionality verified manually
- [ ] Ready for testing (see companion testing guide)

## Testing Readiness

- **Testing Guide**: `.claude/epics/ai-info-aggregation/5-test.md`
- **Test Dependencies**: Development tasks must be completed before testing begins
- **Parallel Work**: Testing team can prepare test cases and fixtures while development is in progress

## Development Notes

### Technical Considerations:

- **XML Library Choice**: Consider fast-xml-parser for performance vs xml2js for simplicity
- **Encoding Detection**: May need to inspect HTTP headers and XML declarations
- **Memory Management**: Large feeds should be parsed streaming if possible
- **Caching Strategy**: Consider caching parsed results to avoid re-parsing
- **Security**: Validate and sanitize all extracted content to prevent XSS

### Integration Points:

- **Database**: Service should work with existing Prisma Article model
- **tRPC**: Follow existing procedure patterns for consistency
- **Error Handling**: Align with application-wide error handling strategy

## Handoff to Testing

When development tasks are complete:

1. Notify testing team that RSS parsing service is implemented
2. Provide sample RSS/Atom feeds for testing
3. Share any implementation details affecting test strategy
4. Document any known limitations or edge cases discovered
5. Review testing guide together for alignment on test coverage
