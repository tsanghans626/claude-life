# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start the Next.js development server
- `npm run build` - Build the application for production
- `npm run start` - Start the production server
- `npm run lint` - Run ESLint on the src directory

## Architecture Overview

This is a Next.js full-stack application using the modern T3 stack:

### Core Technologies

- **Next.js 15** with Pages Router (not App Router)
- **tRPC** for end-to-end type-safe APIs
- **Prisma** for database management with SQLite as default, using Prisma for database access, using explicit select statements
- **TailwindCSS v4** for styling
- **shadcn/ui** for UI components (New York style variant)
- **React Query** (@tanstack/react-query) for client-side state management
- **Zod** for runtime type validation

### Project Structure

- `/src/pages/` - Next.js pages and API routes
- `/src/pages/api/trpc/[trpc].ts` - Main tRPC API handler with example procedures
- `/src/server/trpc.ts` - tRPC server configuration and base procedures
- `/src/utils/trpc.ts` - tRPC client setup with SSR support
- `/src/utils/transformer.ts` - Data transformer for tRPC (superjson)
- `/src/lib/utils.ts` - Utility functions for UI components
- `/src/styles/global.css` - Global styles and Tailwind imports
- `/prisma/schema.prisma` - Database schema (generates client to `src/generated/prisma/`)
- `/components.json` - shadcn/ui configuration

### Key Configuration Details

- TypeScript path alias: `~/*` maps to `./src/*`
- Prisma client outputs to `src/generated/prisma/`
- shadcn/ui components use `~/components`, `~/lib/utils` aliases
- tRPC includes subscription support with WebSocket-like functionality
- Environment-aware base URLs (Vercel, localhost fallback)

### tRPC API Pattern

- All procedures defined in `/src/pages/api/trpc/[trpc].ts`
- Uses `publicProcedure` from server config (no auth layer currently)
- Example procedures: `greeting` (query) and `loopData` (subscription)
- Type definitions exported as `AppRouter` for client consumption

### Database Setup

- SQLite database with Prisma ORM
- Schema uses custom output directory
- Environment variable: `DATABASE_URL` for database connection

## Code Style

- Formatting: Use Prettier with 100-character lines
- Imports: Sorted using simple-import-sort
- Components: Use Pascal naming conventions, co-located with tests
- Hooks: Always prefixed with 'use'

### Patterns to Follow

- Data fetching happens in server components
- Client components receive data as props
- Use the Zod pattern for all external data
- Use error boundaries around every data display component

### Cautions

- IMPORTANT: Don't use useEffect for data fetching
- IMPORTANT: Don't create global state without explicit permission
- IMPORTANT: Don't use the 'any' type to bypass TypeScript
- IMPORTANT: Write end-to-end test cases to cover all major business processes
- IMPORTANT: Do not create mock services or use mock data to help you pass test cases
