---
issue: 5
title: RSS Parser Service
type: development
decomposed: 2025-09-10T15:00:28Z
estimated_hours: 8
---

# Development Guide: Issue #5

## Overview

Implement robust RSS/Atom feed parsing service that normalizes various feed formats into consistent internal structures. Integration with existing T3 stack (tRPC, Prisma, Zod) architecture.

## Prerequisites

- [ ] Local task file reviewed: `.claude/epics/ai-info-aggregation/5.md`
- [ ] Interface specifications defined: `.claude/epics/ai-info-aggregation/5-interface.md`
- [ ] GitHub issue understood: `gh issue view 5`
- [ ] Development environment set up
- [ ] Interface contracts available for implementation

## Development Task List

### Core Implementation Tasks

#### Task 1: Install RSS Parsing Dependencies

**Description**: Add required XML parsing library and types to package.json
**Files to modify/create**:

- `ai-info/package.json` - Add rss-parser dependency
- `ai-info/package.json` - Add @types/rss-parser dev dependency

**Context needed**:

- Choose between rss-parser (RSS-specific) vs fast-xml-parser (generic XML)
- Based on interface analysis, rss-parser is recommended for built-in normalization
- Ensure TypeScript types are available

**Implementation**:

```bash
cd ai-info
npm install rss-parser
npm install --save-dev @types/rss-parser
```

**Estimated Hours**: 0.5
**Priority**: High
**Dependencies**: None

#### Task 2: Create RSS Type Definitions

**Description**: Implement core type definitions and interfaces for RSS parsing
**Files to modify/create**:

- `ai-info/src/services/rss/types.ts` - RSS/Atom type definitions and schemas

**Context needed**:

- Follow existing TypeScript patterns in codebase
- Use Zod for validation schemas
- Match ParsedArticle interface from interface specification

**Implementation**:

```typescript
// Export all types defined in interface guide:
// - FeedFormat, ParsedArticle, ParsedFeedResult
// - FeedValidationResult, FeedValidationError
// - RSSParserService interface
```

**Estimated Hours**: 1
**Priority**: High
**Dependencies**: Task 1

#### Task 3: Create Feed Validation Schemas

**Description**: Implement Zod schemas for RSS/Atom feed structure validation
**Files to modify/create**:

- `ai-info/src/services/rss/validation.ts` - Feed validation schemas

**Context needed**:

- RSS 2.0 specification understanding
- Atom 1.0 specification understanding
- Zod schema patterns used in existing codebase

**Implementation**:

```typescript
// Implement RSSFeedSchema and AtomFeedSchema as defined in interface guide
// Include proper validation rules for required/optional fields
// Export type inference for TypeScript usage
```

**Estimated Hours**: 1.5
**Priority**: High
**Dependencies**: Task 2

#### Task 4: Implement RSS Parser Service

**Description**: Core RSS/Atom parsing logic with format detection and normalization
**Files to modify/create**:

- `ai-info/src/services/rss/parser.ts` - Main parser implementation

**Context needed**:

- rss-parser library usage patterns
- XML namespace handling for different feed formats
- Error handling patterns from existing codebase
- Encoding detection and conversion requirements

**Implementation**:

```typescript
export class RSSParser implements RSSParserService {
  async parseFeed(xmlContent: string): Promise<ParsedFeedResult>;
  detectFeedFormat(xmlContent: string): FeedFormat;
  validateFeed(xmlContent: string): FeedValidationResult;
}

// Key implementation details:
// - Use rss-parser for RSS feeds
// - Custom logic for Atom feed detection
// - Normalize timestamps to Date objects
// - Handle CDATA sections properly
// - Graceful error handling for malformed XML
```

**Estimated Hours**: 3
**Priority**: High
**Dependencies**: Task 3

#### Task 5: Implement Database Article Service

**Description**: Database operations for saving parsed articles to Prisma database
**Files to modify/create**:

- `ai-info/src/services/database/articles.ts` - Article database operations

**Context needed**:

- Existing Prisma client usage patterns in codebase
- Article model structure from schema.prisma
- Database transaction handling for batch operations
- Duplicate detection by article link

**Implementation**:

```typescript
export class ArticleService implements ArticleService {
  async saveArticles(
    articles: ParsedArticle[],
    sourceId: string,
  ): Promise<SaveArticlesResult>;
  async findExistingArticles(links: string[]): Promise<string[]>;
  async updateSourceLastFetched(sourceId: string): Promise<void>;
}

// Key implementation details:
// - Use prisma.article.createMany for batch inserts
// - Handle unique constraint violations gracefully
// - Update existing articles if content changes
// - Track save statistics for monitoring
```

**Estimated Hours**: 2
**Priority**: Medium
**Dependencies**: Task 2

#### Task 6: Create tRPC RSS Router

**Description**: Implement tRPC API endpoints for RSS feed operations
**Files to modify/create**:

- `ai-info/src/server/api/routers/rss.ts` - tRPC router with RSS procedures
- `ai-info/src/server/api/root.ts` - Add RSS router to app router

**Context needed**:

- Existing tRPC patterns in codebase
- Input validation with Zod schemas
- Error handling with tRPC error types
- HTTP client for fetching feeds from URLs

**Implementation**:

```typescript
export const rssRouter = createTRPCRouter({
  fetchRSS: publicProcedure
    .input(fetchRSSInputSchema)
    .mutation(async ({ input }) => {
      // Fetch RSS feed from URL and parse
    }),

  parseRSS: publicProcedure
    .input(parseRSSInputSchema)
    .mutation(async ({ input }) => {
      // Parse provided XML content
    }),

  validateRSS: publicProcedure
    .input(parseRSSInputSchema)
    .query(async ({ input }) => {
      // Validate RSS feed structure
    }),
});
```

**Estimated Hours**: 2
**Priority**: High
**Dependencies**: Task 4, Task 5

### Configuration & Setup Tasks

#### Task 7: Error Handling Implementation

**Description**: Implement comprehensive error handling following project patterns
**Files to modify/create**:

- `ai-info/src/services/rss/errors.ts` - RSS-specific error types

**Context needed**:

- Existing error handling patterns in codebase
- tRPC error handling approach
- User-friendly error messages for different failure modes

**Implementation**:

```typescript
export class RSSParsingError extends Error {
  constructor(
    message: string,
    public cause?: Error,
  ) {}
}

export class FeedValidationError extends Error {
  constructor(
    message: string,
    public errors: ValidationError[],
  ) {}
}

// Error categories:
// - Network errors (feed URL not accessible)
// - Parsing errors (malformed XML)
// - Validation errors (invalid feed structure)
// - Database errors (save failures)
```

**Estimated Hours**: 1
**Priority**: Medium
**Dependencies**: Task 4

## Task Dependencies

### Development Task Flow

1. **Setup Phase**: Task 1 (dependencies) → Task 2 (types) → Task 3 (validation)
2. **Core Implementation**: Task 4 (parser) → Task 5 (database) → Task 6 (tRPC)
3. **Polish Phase**: Task 7 (error handling)

### Parallel Opportunities

- Task 5 (database service) can be developed in parallel with Task 4 (parser)
- Task 7 (error handling) can be developed alongside Task 4-6

## Definition of Done

### Development Tasks Complete When:

- [ ] RSS parser successfully handles RSS 2.0 and Atom 1.0 feeds
- [ ] Malformed XML gracefully handled with informative errors
- [ ] Feed validation works for structure and required fields
- [ ] Article metadata extracted correctly (title, description, link, pubDate, author)
- [ ] Encoding issues handled (UTF-8, ISO-8859-1, etc.)
- [ ] Timestamps normalized to consistent Date format
- [ ] CDATA sections parsed properly
- [ ] Articles saved to database with duplicate detection
- [ ] tRPC endpoints functional and properly validated
- [ ] Error handling comprehensive and user-friendly
- [ ] Code follows existing patterns and conventions
- [ ] No lint/type errors
- [ ] Ready for testing (see companion testing guide)

## Testing Coordination

- **Testing Guide**: `.claude/epics/ai-info-aggregation/5-test.md`
- **Test-Driven Development**: Tests written based on interface specifications, not implementation
- **Parallel Work**: Testing team can write tests using interface contracts while development proceeds
- **Continuous Integration**: Tests provide immediate feedback during development

## Development Notes

### RSS vs Atom Considerations

- **RSS 2.0**: Uses `<channel><item>` structure, dates in RFC 2822 format
- **Atom 1.0**: Uses `<feed><entry>` structure, dates in ISO 8601 format
- **Detection Logic**: Check root element name (`rss` vs `feed`) and namespace declarations
- **Normalization**: Convert all dates to JavaScript Date objects, standardize field names

### Error Handling Strategy

- **Network Errors**: Retry with exponential backoff, timeout after 30 seconds
- **XML Parsing**: Catch library errors, provide line number if available
- **Validation Errors**: Return detailed field-level errors for debugging
- **Database Errors**: Log for monitoring, return user-friendly messages

### Performance Considerations

- **Memory Usage**: Stream large feeds instead of loading entire content in memory
- **Database Efficiency**: Use batch operations, avoid N+1 queries
- **Caching**: Consider caching parsed feeds for repeated requests (future enhancement)

### Encoding Handling

- **Detection**: Use charset from HTTP Content-Type header
- **Conversion**: Handle common encodings (UTF-8, UTF-16, ISO-8859-1)
- **Fallback**: Default to UTF-8 if encoding detection fails

## Handoff to Testing

When development tasks are complete:

1. Notify testing team that implementation is ready
2. Provide sample RSS/Atom feeds for testing
3. Share any implementation details that affect testing approach
4. Review error scenarios and edge cases discovered during development
