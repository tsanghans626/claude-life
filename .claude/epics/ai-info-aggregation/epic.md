---
name: ai-info-aggregation
status: backlog
created: 2025-09-10T03:03:51Z
progress: 0%
prd: .claude/prds/ai-info-aggregation.md
github: https://github.com/tsanghans626/claude-life/issues/1
---

# Epic: AI Info Aggregation

## Overview

Build an RSS aggregation tool within the existing ai-info Next.js project template. The system will manage up to 9999 RSS sources, automatically fetch articles, and provide a card-based reading interface with read/unread tracking and article retention features.

## Architecture Decisions

- **Extend existing T3 stack**: Build on Next.js 15 + tRPC + Prisma + SQLite foundation
- **Database-first approach**: Use Prisma schema extensions for RSS data modeling
- **Server-side RSS parsing**: Implement RSS fetching as tRPC procedures for type safety
- **Background jobs**: Use Next.js API routes with cron-like scheduling for automated fetching
- **Component reuse**: Leverage shadcn/ui for consistent UI patterns

## Technical Approach

### Frontend Components

- RSS源管理页面：表格形式展示，支持批量操作
- 文章卡片列表：响应式网格布局，支持无限滚动
- 状态指示器：实时显示获取进度和错误状态
- 操作面板：分类管理、定时设置、手动触发获取

### Backend Services

- **RSS管理 API**: CRUD operations for RSS sources with validation
- **内容获取服务**: RSS parser using xml2js or similar, with retry logic
- **定时任务系统**: Background job runner for scheduled fetching
- **数据清理服务**: Automated cleanup of old articles (7-day retention)

### Infrastructure

- **数据库扩展**: Add RSS sources, articles, categories tables to existing Prisma schema
- **错误处理**: Graceful degradation with detailed error logging
- **性能优化**: Database indexes for article queries, pagination support

## Implementation Strategy

- **Phase 1**: Database schema and RSS source management
- **Phase 2**: RSS fetching and parsing logic
- **Phase 3**: Article display and state management
- **Phase 4**: Background jobs and cleanup automation
- **测试策略**: Unit tests for RSS parsing, integration tests for full workflows

## Task Breakdown Preview

High-level task categories:

- [ ] **数据库设计**: Extend Prisma schema with RSS tables and relationships
- [ ] **RSS源管理**: CRUD operations with category and tag support
- [ ] **RSS解析服务**: Fetch and parse RSS/Atom feeds with error handling
- [ ] **文章展示界面**: Card-based layout with read/unread state management
- [ ] **后台任务系统**: Scheduled fetching and automated cleanup
- [ ] **用户界面完善**: Responsive design and loading states

## Dependencies

- **External RSS feeds**: Relies on standard RSS/Atom format compliance
- **Network connectivity**: Required for RSS fetching operations
- **Background job execution**: Node.js runtime for scheduled tasks
- **XML parsing library**: xml2js or fast-xml-parser for RSS processing

## Success Criteria (Technical)

- **数据库性能**: Article queries < 200ms with proper indexing
- **RSS获取成功率**: > 95% for valid feeds with retry mechanism
- **UI响应性**: Page loads < 2s, smooth scroll performance
- **数据一致性**: No article duplicates, accurate read/unread state
- **错误恢复**: Graceful handling of network failures and invalid feeds

## Tasks Created
- [ ] #10 - Article List and Pagination (parallel: false)
- [ ] #11 - Background Job System (parallel: false)
- [ ] #12 - Article Cleanup Service (parallel: true)
- [ ] #13 - Integration Testing Suite (parallel: false)
- [ ] #2 - Database Schema Design (parallel: true)
- [ ] #3 - RSS Source Management API (parallel: false)
- [ ] #4 - Article Storage Schema (parallel: false)
- [ ] #5 - RSS Parser Service (parallel: true)
- [ ] #6 - RSS Fetching Service (parallel: false)
- [ ] #7 - Content Deduplication Logic (parallel: false)
- [ ] #8 - RSS Source Management UI (parallel: true)
- [ ] #9 - Article Card Component (parallel: true)

Total tasks:       12
Parallel tasks:        5
Sequential tasks: 7
## Estimated Effort

- **总体时间**: 4-5 weeks for full implementation
- **核心功能**: 2-3 weeks (database + RSS + basic UI)
- **优化完善**: 1-2 weeks (performance + UX + testing)
- **关键路径**: RSS parsing reliability and background job scheduling
