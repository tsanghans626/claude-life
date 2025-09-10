---
issue: 5
title: RSS Parser Service
type: testing
decomposed: 2025-09-10T15:00:28Z
estimated_hours: 6
---

# Testing Guide: Issue #5

## Overview

Comprehensive testing for RSS/Atom feed parsing service including unit tests, integration tests, and end-to-end validation. Tests are designed to validate interface contracts and real-world RSS feed scenarios without mocking.

## Prerequisites

- [ ] Interface contracts defined: `.claude/epics/ai-info-aggregation/5-interface.md` (function signatures & behavior)
- [ ] Development specifications understood: `.claude/epics/ai-info-aggregation/5-dev.md` (implementation approach)
- [ ] Testing strategy aligned with interface contracts
- [ ] Test environment set up (independent of development completion)
- [ ] Testing frameworks/tools available (Jest/Vitest expected from T3 stack)

## Testing Strategy

### Testing Levels

- **Unit Tests**: Individual RSS parser functions and database operations
- **Integration Tests**: tRPC endpoints with real HTTP requests and database
- **End-to-End Tests**: Complete RSS feed processing workflows
- **Performance Tests**: Large feed parsing and memory usage
- **Error Handling Tests**: Malformed feeds and network failures

### Real Data Testing Philosophy

- **No Mock Services**: All tests use real RSS feeds and actual database
- **Real Feed Sources**: Test with authentic RSS/Atom feeds from various sources
- **Live HTTP Requests**: Network tests use actual feed URLs
- **Database Integration**: Tests run against real database with cleanup

## Testing Task List

### Unit Testing Tasks

#### Test 1: RSS Parser Core Functions Unit Tests

**Description**: Test individual parsing functions with various RSS/Atom formats
**Test files to create/modify**:

- `ai-info/src/services/rss/__tests__/parser.test.ts` - Parser function tests

**Context needed**:

- Sample RSS 2.0 feeds (valid and malformed)
- Sample Atom 1.0 feeds (valid and malformed)
- Edge cases: missing fields, invalid dates, encoding issues
- Expected ParsedArticle output format

**Test scenarios**:

- **Happy Path**: Valid RSS 2.0 feed parsing returns correct ParsedFeedResult
- **Happy Path**: Valid Atom 1.0 feed parsing returns correct ParsedFeedResult
- **Format Detection**: detectFeedFormat correctly identifies RSS vs Atom vs unknown
- **Date Parsing**: Various date formats normalized to Date objects
- **CDATA Handling**: CDATA sections in titles/descriptions parsed correctly
- **Encoding**: UTF-8, ISO-8859-1 feeds handled properly
- **Missing Fields**: Optional fields (author, description) handled gracefully
- **Malformed XML**: Invalid XML throws RSSParsingError with useful message
- **Invalid Structure**: Missing required fields caught by validation

**Test Data Requirements**:

```typescript
// Sample feeds to create as test fixtures:
const VALID_RSS_FEED = `<?xml version="1.0"?>
<rss version="2.0">
  <channel>
    <title>Test Feed</title>
    <description>Test Description</description>
    <link>https://example.com</link>
    <item>
      <title>Test Article</title>
      <description><![CDATA[Test content with <em>HTML</em>]]></description>
      <link>https://example.com/article1</link>
      <pubDate>Wed, 10 Sep 2025 12:00:00 GMT</pubDate>
      <author>test@example.com</author>
      <guid>unique-id-1</guid>
    </item>
  </channel>
</rss>`;

const VALID_ATOM_FEED = `<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title>Test Atom Feed</title>
  <subtitle>Test Atom Description</subtitle>
  <link href="https://example.com" />
  <entry>
    <title>Test Atom Article</title>
    <summary>Test atom content</summary>
    <link href="https://example.com/atom-article1" />
    <published>2025-09-10T12:00:00Z</published>
    <author><name>Test Author</name></author>
    <id>atom-unique-id-1</id>
  </entry>
</feed>`;
```

**Estimated Hours**: 2.5
**Priority**: High
**Dependencies**: Interface contracts and expected behavior defined

#### Test 2: Feed Validation Schema Tests

**Description**: Test Zod schema validation for RSS/Atom feed structures
**Test files to create/modify**:

- `ai-info/src/services/rss/__tests__/validation.test.ts` - Schema validation tests

**Context needed**:

- RSSFeedSchema and AtomFeedSchema definitions
- Valid and invalid feed structure examples
- Expected validation error formats

**Test scenarios**:

- **Valid RSS Schema**: Well-formed RSS passes RSSFeedSchema validation
- **Valid Atom Schema**: Well-formed Atom passes AtomFeedSchema validation
- **Missing Required Fields**: RSS without title/description fails validation
- **Invalid URLs**: Malformed link URLs caught by validation
- **Optional Fields**: Feeds with missing optional fields pass validation
- **Validation Error Details**: Clear error messages for specific field violations

**Estimated Hours**: 1
**Priority**: High
**Dependencies**: Schema definitions and validation contracts

#### Test 3: Database Article Service Unit Tests

**Description**: Test database operations for saving and retrieving articles
**Test files to create/modify**:

- `ai-info/src/services/database/__tests__/articles.test.ts` - Database service tests

**Context needed**:

- Existing Article and Source models from Prisma schema
- Database test setup with cleanup after each test
- Sample ParsedArticle objects for testing

**Test scenarios**:

- **Save New Articles**: saveArticles creates new Article records correctly
- **Duplicate Detection**: Existing articles (by link) not created again
- **Batch Operations**: Multiple articles saved in single transaction
- **Update Existing**: Modified articles update existing records
- **Source Update**: updateSourceLastFetched updates Source.updatedAt
- **Error Handling**: Database constraint violations handled gracefully
- **Find Existing**: findExistingArticles returns correct existing links

**Database Test Setup**:

```typescript
// Before each test: create test Source
const testSource = await prisma.source.create({
  data: {
    url: "https://example.com/feed",
    title: "Test Source",
    categoryId: testCategoryId,
  },
});

// After each test: cleanup
await prisma.article.deleteMany();
await prisma.source.deleteMany();
```

**Estimated Hours**: 2
**Priority**: Medium
**Dependencies**: Prisma database access and ParsedArticle interface

### Integration Testing Tasks

#### Test 4: tRPC RSS Router Integration Tests

**Description**: Test tRPC endpoints with real HTTP requests and database integration
**Test files to create/modify**:

- `ai-info/src/server/api/routers/__tests__/rss.test.ts` - tRPC endpoint tests

**Context needed**:

- tRPC test setup with database context
- Real RSS feed URLs for testing (with stable content)
- HTTP client configuration for feed fetching

**Test scenarios**:

- **fetchRSS Success**: Valid RSS URL fetched and parsed correctly
- **fetchRSS Network Error**: Invalid URL returns appropriate error
- **parseRSS Success**: Direct XML parsing returns ParsedFeedResult
- **validateRSS Success**: Feed validation returns correct FeedValidationResult
- **Input Validation**: Invalid inputs rejected by Zod schemas
- **Database Integration**: Parsed articles saved to database when sourceId provided
- **Error Propagation**: Parser errors properly converted to tRPC errors

**Real Feed Testing**:

```typescript
// Use stable, reliable RSS feeds for testing:
const TEST_FEEDS = [
  "https://rss.cnn.com/rss/edition.rss", // CNN RSS
  "https://feeds.feedburner.com/oreilly/radar", // O'Reilly Atom
  "https://blog.golang.org/feed.atom", // Go Blog Atom
];
```

**Estimated Hours**: 2
**Priority**: High
**Dependencies**: tRPC router implementation and database service

### End-to-End Testing Tasks

#### Test 5: Complete RSS Processing Workflow Tests

**Description**: Test complete RSS feed processing from URL to database storage
**Test files to create/modify**:

- `ai-info/__tests__/e2e/rss-workflow.test.ts` - End-to-end workflow tests

**Context needed**:

- Full application stack running (database, tRPC server)
- Real RSS feed sources with predictable content
- Database cleanup between test runs

**Test scenarios**:

- **Full RSS Workflow**: URL → HTTP fetch → Parse → Validate → Save → Verify in DB
- **Atom Workflow**: Same workflow for Atom feeds
- **Error Recovery**: Malformed feeds don't crash the system
- **Duplicate Prevention**: Re-processing same feed doesn't create duplicates
- **Source Management**: Processing updates Source.updatedAt timestamp
- **Performance**: Large feeds (100+ articles) process within reasonable time

**Workflow Test Structure**:

```typescript
describe("RSS Processing Workflow", () => {
  test("processes RSS feed end-to-end", async () => {
    // 1. Create test source in database
    // 2. Call fetchRSS tRPC endpoint
    // 3. Verify articles saved to database
    // 4. Verify source updated timestamp
    // 5. Verify article count matches feed
  });
});
```

**Estimated Hours**: 1.5
**Priority**: Medium
**Dependencies**: Full implementation stack completed

### Performance Testing Tasks

#### Test 6: RSS Parser Performance Tests

**Description**: Test parsing performance and memory usage with large feeds
**Test files to create/modify**:

- `ai-info/__tests__/performance/rss-performance.test.ts` - Performance benchmarks

**Context needed**:

- Large RSS feeds (500+ articles) for testing
- Memory usage monitoring utilities
- Performance timing measurements

**Test scenarios**:

- **Large Feed Parsing**: 500+ article RSS feed parses within 10 seconds
- **Memory Usage**: Parser doesn't leak memory with multiple feeds
- **Database Batch Performance**: 100+ articles save in single batch operation
- **Concurrent Processing**: Multiple feeds processed simultaneously
- **Error Performance**: Malformed feed errors don't cause memory leaks

**Performance Benchmarks**:

```typescript
test("parses large RSS feed within performance limits", async () => {
  const startTime = performance.now();
  const startMemory = process.memoryUsage();

  const result = await rssParser.parseFeed(LARGE_RSS_FEED);

  const endTime = performance.now();
  const endMemory = process.memoryUsage();

  expect(endTime - startTime).toBeLessThan(10000); // 10 seconds
  expect(result.articles.length).toBeGreaterThan(500);
});
```

**Estimated Hours**: 1
**Priority**: Low
**Dependencies**: Core parser implementation completed

## Testing Dependencies

### Testing Task Flow

1. **Test-First Approach**: Unit tests written from interface specifications (will fail until implementation)
2. **Red-Green-Refactor Cycle**: Tests fail (red) → implementation makes them pass (green) → refactor code
3. **Integration Testing**: Requires actual parser and database implementations
4. **E2E Validation**: Requires fully implemented tRPC endpoints and database operations
5. **Performance Testing**: Requires stable implementations for meaningful benchmarks

### Parallel Testing Opportunities

- **Interface-Based Testing**: Unit tests written from specifications before implementation exists
- **Test Data Preparation**: Create RSS/Atom feed fixtures immediately
- **Test Infrastructure**: Setup Jest/Vitest configuration and database test utilities
- **Real Feed Collection**: Identify and validate reliable RSS feeds for testing
- **Continuous Feedback**: Tests provide real-time validation during development

## Test Data Requirements

### Test Feed Fixtures

Create comprehensive test fixtures in `ai-info/src/services/rss/__tests__/fixtures/`:

```typescript
// feeds.ts - Test feed collection
export const RSS_FEEDS = {
  valid_rss: VALID_RSS_FEED,
  malformed_xml: '<?xml version="1.0"?><rss><channel><title>Test</title>', // missing closing tags
  missing_required: '<?xml version="1.0"?><rss><channel></channel></rss>', // missing title
  encoding_utf8: RSS_FEED_WITH_UTF8_CHARS,
  encoding_latin1: RSS_FEED_WITH_LATIN1_ENCODING,
  large_feed: RSS_FEED_WITH_500_ARTICLES,
};

export const ATOM_FEEDS = {
  valid_atom: VALID_ATOM_FEED,
  minimal_atom: MINIMAL_ATOM_FEED,
  malformed_atom: MALFORMED_ATOM_FEED,
};
```

### Real Service Dependencies

- **Live RSS Feeds**: Identify stable feeds for integration testing
- **Database Setup**: Test database with proper cleanup between tests
- **HTTP Mock Server**: Optional - for testing network error scenarios
- **Performance Monitoring**: Memory and timing measurement utilities

## Definition of Done

### Testing Tasks Complete When:

- [ ] All unit tests pass and cover core parsing functionality
- [ ] Integration tests verify tRPC endpoints with real HTTP and database
- [ ] End-to-end tests validate complete workflow scenarios
- [ ] Performance tests establish baseline benchmarks
- [ ] Error handling tests cover all failure scenarios
- [ ] Test coverage meets project requirements (80%+ recommended)
- [ ] Tests are verbose and provide useful debugging information
- [ ] Test data fixtures cover all major RSS/Atom variations
- [ ] Real feed integration tests use stable, reliable sources
- [ ] Test documentation updated with examples and setup instructions

## Testing Notes

### RSS/Atom Testing Considerations

- **Date Format Variations**: RSS uses RFC 2822, Atom uses ISO 8601
- **Namespace Handling**: Atom feeds require proper namespace parsing
- **Content Types**: Test both plain text and HTML content in descriptions
- **Author Formats**: RSS uses email, Atom uses name structure
- **GUID vs ID**: Different unique identifier formats between RSS/Atom

### Database Testing Strategy

- **Transaction Isolation**: Each test runs in isolated transaction
- **Cleanup Strategy**: Truncate test tables after each test suite
- **Foreign Key Handling**: Ensure Source exists before creating Articles
- **Constraint Testing**: Test unique constraints on article links

### Error Testing Approach

- **Network Errors**: Use invalid URLs, timeout scenarios
- **XML Parsing Errors**: Malformed XML, invalid characters
- **Validation Errors**: Missing required fields, invalid data types
- **Database Errors**: Constraint violations, connection failures

### Performance Testing Guidelines

- **Baseline Metrics**: Establish performance baselines for regression testing
- **Memory Monitoring**: Track memory usage during large feed processing
- **Concurrency Testing**: Test multiple simultaneous feed processing
- **Resource Cleanup**: Ensure no memory leaks in long-running tests

## Development Coordination

- **Development Guide**: `.claude/epics/ai-info-aggregation/5-dev.md`
- **Communication**: Regular sync with development team on progress
- **Blocking Issues**: Process for handling test-blocking development issues
- **Test-First Development**: Write tests before implementation when interface contracts are clear
- **Continuous Integration**: Tests run automatically on code changes
