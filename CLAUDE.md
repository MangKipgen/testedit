# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation
```bash
npm run setup              # Full setup: install deps, generate Prisma client, run migrations
npm install                # Install dependencies only
npx prisma generate        # Generate Prisma client
npx prisma migrate dev     # Run database migrations
```

### Development Server
```bash
npm run dev                # Start development server with Turbopack
npm run dev:daemon         # Start dev server as daemon, logs to logs.txt
```

### Build and Production
```bash
npm run build              # Build for production
npm start                  # Start production server
```

### Testing and Quality
```bash
npm run test               # Run tests with Vitest
npm run lint               # Run ESLint
```

### Database Operations
```bash
npm run db:reset           # Reset database (force migrate reset)
npx prisma studio          # Open Prisma Studio for database inspection
```

## Architecture Overview

UIGen is an AI-powered React component generator built on Next.js 15 with a virtual file system. The core architecture centers around:

### Virtual File System (VFS)
- **Core Implementation**: `src/lib/file-system.ts` - Complete VFS with CRUD operations
- **React Integration**: `src/lib/contexts/file-system-context.tsx` - Context provider for VFS state
- **Key Feature**: All generated components exist in memory only, no files written to disk
- **File Structure**: Uses path-based organization with recursive directory support

### AI Integration Architecture
- **Chat API**: `src/app/api/chat/route.ts` - Handles streaming AI responses
- **AI Tools**: 
  - `src/lib/tools/str-replace.ts` - File editing tool for AI
  - `src/lib/tools/file-manager.ts` - File operations tool for AI
- **Provider**: `src/lib/provider.ts` - Abstracts AI model selection (Anthropic Claude)
- **Generation Prompt**: `src/lib/prompts/generation.tsx` - System prompt for component generation

### Database and Persistence
- **ORM**: Prisma with SQLite for development
- **Schema**: `prisma/schema.prisma` - User authentication and project persistence
- **Auth**: JWT-based authentication in `src/lib/auth.ts`
- **Project Storage**: Chat messages and VFS state serialized to database

### Component Architecture
```
src/components/
├── auth/           # Authentication components
├── chat/           # AI chat interface
├── editor/         # Code editor and file tree
├── preview/        # Live component preview
└── ui/             # Shared UI components (shadcn/ui)
```

### Key Technical Patterns

#### AI Tool Integration
The AI uses two primary tools to manipulate the VFS:
- `str_replace_editor`: Create, edit files with string replacement
- `file_manager`: Rename, delete files and directories

#### Context Management
- `FileSystemContext`: Global state for VFS operations
- `ChatContext`: AI conversation state and message handling

#### Component Generation Flow
1. User describes component in chat
2. AI receives system prompt from `generation.tsx`
3. AI uses tools to create/modify files in VFS
4. Preview frame renders components in real-time
5. For authenticated users, state persists to database

## Development Guidelines

### File Import Conventions
- Use `@/` alias for all internal imports (configured in tsconfig.json)
- Example: `import { VirtualFileSystem } from '@/lib/file-system'`

### Component Generation Requirements
- Every project must have a root `/App.jsx` file as the main export
- Use Tailwind CSS for styling, avoid inline styles
- Components render in a sandboxed preview frame

### Testing Strategy
- Tests use Vitest with jsdom environment
- Component tests in `__tests__/` directories alongside source files
- Focus on VFS operations and React component behavior

### Authentication
- Optional - app works without API key (returns static code)
- Authenticated users get component persistence
- Anonymous users tracked via `anon-work-tracker.ts`