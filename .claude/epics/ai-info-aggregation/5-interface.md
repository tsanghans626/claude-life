---
issue: 5
title: RSS Parser Service
type: interface
decomposed: 2025-09-10T15:00:28Z
estimated_hours: 2
---

# Interface Guide: Issue #5

## Overview

Define interfaces and contracts for RSS/Atom feed parsing service that normalizes various feed formats into consistent internal structures for downstream processing. This service integrates with the existing T3 stack (tRPC, Prisma, Zod) architecture.

## Prerequisites

- [ ] Local task file reviewed: `.claude/epics/ai-info-aggregation/5.md`
- [ ] GitHub issue understood: `gh issue view 5`
- [ ] Existing codebase patterns analyzed
- [ ] Dependencies and integration points identified

## Interface Design Tasks

### Core Interface Definitions

#### Interface 1: RSS Parser Service

**Description**: Core service for parsing RSS/Atom feeds and extracting article metadata
**Files to create**:

- `ai-info/src/services/rss/types.ts` - RSS/Atom type definitions and schemas
- `ai-info/src/services/rss/parser.ts` - Parser service interface definition

**Interface Specification**:

```typescript
// Feed format detection
export type FeedFormat = "rss" | "atom" | "unknown";

// Normalized article output structure
export interface ParsedArticle {
  title: string;
  description: string | null;
  link: string;
  publishedAt: Date | null;
  author: string | null;
  guid: string | null;
  content: string | null;
}

// Parser service interface
export interface RSSParserService {
  parseFeed(xmlContent: string): Promise<ParsedFeedResult>;
  detectFeedFormat(xmlContent: string): FeedFormat;
  validateFeed(xmlContent: string): FeedValidationResult;
}

// Parser result types
export interface ParsedFeedResult {
  format: FeedFormat;
  title: string;
  description: string | null;
  link: string | null;
  articles: ParsedArticle[];
  lastBuildDate: Date | null;
}

// Validation result types
export interface FeedValidationResult {
  isValid: boolean;
  format: FeedFormat;
  errors: FeedValidationError[];
}

export interface FeedValidationError {
  code: string;
  message: string;
  path?: string;
}
```

**Behavior Contract**:

- **Input validation**: XML content must be valid UTF-8 string, handle encoding detection
- **Output guarantees**: Always returns ParsedFeedResult with normalized articles array
- **Error conditions**: Throws RSSParsingError for malformed XML, returns validation errors for invalid feeds
- **Side effects**: No side effects - pure parsing function

**Dependencies**: XML parsing library (rss-parser or fast-xml-parser)
**Estimated Hours**: 1.5
**Priority**: High

#### Interface 2: tRPC RSS Procedures

**Description**: tRPC API endpoints for RSS feed operations
**Files to create**:

- `ai-info/src/server/api/routers/rss.ts` - tRPC router with RSS procedures

**Interface Specification**:

```typescript
// tRPC input schemas
export const fetchRSSInputSchema = z.object({
  url: z.string().url(),
  sourceId: z.string().optional(),
});

export const parseRSSInputSchema = z.object({
  xmlContent: z.string(),
  validate: z.boolean().default(true),
});

// tRPC procedures interface
export interface RSSRouter {
  fetchRSS: Procedure<typeof fetchRSSInputSchema, ParsedFeedResult>;
  parseRSS: Procedure<typeof parseRSSInputSchema, ParsedFeedResult>;
  validateRSS: Procedure<typeof parseRSSInputSchema, FeedValidationResult>;
}
```

**Behavior Contract**:

- **Input validation**: URL validation for fetchRSS, string validation for parseRSS
- **Output guarantees**: Returns standardized ParsedFeedResult or FeedValidationResult
- **Error conditions**: HTTP errors for invalid URLs, parsing errors for malformed XML
- **Side effects**: HTTP requests for fetchRSS, database writes when saving articles

**Dependencies**: Interface 1 (RSS Parser Service)
**Estimated Hours**: 1
**Priority**: High

#### Interface 3: Database Operations Interface

**Description**: Database interface for saving parsed articles to Prisma database
**Files to create**:

- `ai-info/src/services/database/articles.ts` - Article database operations

**Interface Specification**:

```typescript
// Database service interface
export interface ArticleService {
  saveArticles(
    articles: ParsedArticle[],
    sourceId: string,
  ): Promise<SaveArticlesResult>;
  findExistingArticles(links: string[]): Promise<string[]>;
  updateSourceLastFetched(sourceId: string): Promise<void>;
}

// Database operation results
export interface SaveArticlesResult {
  created: number;
  updated: number;
  skipped: number;
  errors: ArticleSaveError[];
}

export interface ArticleSaveError {
  article: ParsedArticle;
  error: string;
}
```

**Behavior Contract**:

- **Input validation**: Valid ParsedArticle array and existing sourceId
- **Output guarantees**: Returns count of created/updated/skipped articles
- **Error conditions**: Database constraint violations, connection errors
- **Side effects**: Creates/updates Article records in database

**Dependencies**: Prisma client, Interface 1 (ParsedArticle type)
**Estimated Hours**: 1
**Priority**: Medium

### Data Schema Definitions

#### Schema 1: RSS Feed Validation Schema

**Description**: Zod schemas for RSS/Atom feed structure validation
**Files to create**:

- `ai-info/src/services/rss/validation.ts` - Feed validation schemas

**Schema Definition**:

```typescript
// RSS 2.0 validation schema
export const RSSFeedSchema = z.object({
  channel: z.object({
    title: z.string(),
    description: z.string(),
    link: z.string().url(),
    item: z
      .array(
        z.object({
          title: z.string(),
          description: z.string().optional(),
          link: z.string().url(),
          pubDate: z.string().optional(),
          author: z.string().optional(),
          guid: z.string().optional(),
        }),
      )
      .optional(),
  }),
});

// Atom 1.0 validation schema
export const AtomFeedSchema = z.object({
  feed: z.object({
    title: z.string(),
    subtitle: z.string().optional(),
    link: z.object({
      "@_href": z.string().url(),
    }),
    entry: z
      .array(
        z.object({
          title: z.string(),
          summary: z.string().optional(),
          link: z.object({
            "@_href": z.string().url(),
          }),
          published: z.string().optional(),
          author: z
            .object({
              name: z.string(),
            })
            .optional(),
          id: z.string().optional(),
        }),
      )
      .optional(),
  }),
});

export type RSSFeed = z.infer<typeof RSSFeedSchema>;
export type AtomFeed = z.infer<typeof AtomFeedSchema>;
```

**Validation Rules**:

- RSS feeds must have channel with title, description, link
- Atom feeds must have feed with title and link
- Article items are optional but validated when present
- URLs must be valid format
- Dates are optional but validated when provided

**Usage Context**: Feed format detection and structure validation
**Estimated Hours**: 0.5
**Priority**: High

## Integration Contracts

### External Service Interfaces

#### External Service 1: XML Parsing Library

**Description**: Integration with rss-parser or fast-xml-parser library
**Interface Requirements**:

- **Library**: rss-parser (recommended for RSS-specific features)
- **Alternative**: fast-xml-parser (for high-performance generic XML)
- **Input Format**: Raw XML string from HTTP response
- **Output Format**: Parsed JavaScript object with feed metadata
- **Error Handling**: Catch parsing errors and convert to standardized error format

**Mock Requirements**: None (real library integration only)
**Estimated Hours**: 0.5
**Priority**: Medium

#### External Service 2: HTTP Client

**Description**: HTTP client for fetching RSS feeds from URLs
**Interface Requirements**:

- **Client**: Built-in fetch API or axios
- **Endpoints**: RSS/Atom feed URLs provided by users
- **Authentication**: Basic auth support for protected feeds (future)
- **Data Format**: Raw XML response body
- **Error Handling**: Network errors, 404s, malformed responses

**Mock Requirements**: None (real HTTP requests only)
**Estimated Hours**: 0.5
**Priority**: Medium

## Interface Dependencies

### Interface Creation Flow

1. **Core Data Types**: Define ParsedArticle, FeedFormat, validation schemas first
2. **Parser Service Interface**: Define RSS parsing method signatures and contracts
3. **tRPC API Interface**: Define API endpoints using core types
4. **Database Service Interface**: Define persistence operations for parsed data

### Parallel Development Enablement

Once interfaces are defined, teams work simultaneously:

- **Development Team**: Implements RSS parsing logic against defined interfaces
- **Testing Team**: Writes tests using interface contracts (tests initially fail until implementation)
- **Integration Team**: Sets up tRPC endpoints and database operations using interface contracts
- **All Teams**: Coordinate through shared interface documentation, not through waiting

## Definition of Done

### Interface Design Complete When:

- [ ] All function signatures defined with TypeScript types
- [ ] Input/output contracts specified with Zod validation schemas
- [ ] Error conditions and types clearly defined for parsing failures
- [ ] Data schemas created for RSS/Atom validation
- [ ] Integration contracts specified for XML parsing library and HTTP client
- [ ] Interface documentation includes usage examples
- [ ] No mock interfaces - all definitions are for real implementations

## Development Handoff

**Development Guide**: `.claude/epics/ai-info-aggregation/5-dev.md`
**Testing Guide**: `.claude/epics/ai-info-aggregation/5-test.md`

### Handoff Requirements:

- Interface files created with type signatures only (no implementation)
- All types exported and available for import
- Zod schemas defined for validation
- tRPC input/output types specified
- Contract specifications detailed and unambiguous

## Interface Notes

**XML Parsing Library Choice**: Recommend rss-parser for RSS-specific features and built-in normalization. Fast-xml-parser is alternative for custom parsing logic.

**Error Handling Strategy**: Follow existing project pattern - fail fast for critical parsing errors, graceful degradation for optional metadata fields.

**Database Integration**: Leverage existing Article model in Prisma schema - no schema changes required.

**Type Safety**: Full end-to-end type safety through tRPC AppRouter export and Zod validation schemas.
