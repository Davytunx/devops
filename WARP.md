# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

Node.js Express API with authentication features using PostgreSQL (Neon), Drizzle ORM, JWT authentication, and structured logging with Winston.

## Development Commands

### Running the Application
- `npm run dev` - Start development server with hot reload (uses Node's --watch flag)

### Code Quality
- `npm run lint` - Check code for linting errors
- `npm run lint:fix` - Fix auto-fixable linting errors
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check if code is properly formatted

### Database Operations
- `npm run db:generate` - Generate Drizzle migration files from schema changes
- `npm run db:migrate` - Run pending database migrations
- `npm run db:studio` - Open Drizzle Studio (visual database manager)

## Architecture

### Module Path Aliases

The project uses Node.js import maps (package.json `imports` field) for clean imports:
- `#config/*` → `./src/config/*` - Configuration modules
- `#controllers/*` → `./src/controllers/*` - Request handlers
- `#middlewares/*` → `./src/middlewares/*` - Express middlewares
- `#models/*` → `./src/models/*` - Drizzle ORM schemas
- `#routes/*` → `./src/routes/*` - Route definitions
- `#services/*` → `./src/services/*` - Business logic
- `#utils/*` → `./src/utils/*` - Utility functions
- `#validations/*` → `./src/validations/*` - Zod validation schemas

Always use these aliases when importing, not relative paths.

### Application Structure

**Entry Points:**
- `src/index.js` - Loads environment variables and starts server
- `src/server.js` - Creates HTTP server and listens on PORT
- `src/app.js` - Express app configuration with middleware stack

**Middleware Stack (in order):**
1. Helmet (security headers)
2. CORS
3. JSON body parser
4. URL-encoded body parser
5. Cookie parser
6. Morgan (HTTP request logging to Winston)

**Database:**
- Uses Neon serverless PostgreSQL via `@neondatabase/serverless`
- ORM: Drizzle with HTTP adapter
- Connection configured in `src/config/database.js`
- Schema files in `src/models/` (e.g., `user.model.js`)
- After schema changes: run `npm run db:generate` then `npm run db:migrate`

**Logging:**
- Winston logger configured in `src/config/logger.js`
- Writes to `logs/error.log` (errors only) and `logs/combined.log` (all logs)
- Console output in non-production environments
- HTTP requests logged via Morgan integration

**Authentication Flow:**
- Zod schemas validate request bodies (`src/validations/`)
- Controllers handle HTTP logic (`src/controllers/`)
- Services contain business logic (`src/services/`)
- JWT tokens stored in HTTP-only cookies via `src/utils/cookies.js`
- JWT expiration: 1 day (configured in `src/utils/jwt.js`)
- Password hashing: bcrypt with 10 salt rounds

### Code Style

ESLint enforces:
- 2-space indentation (switch cases included)
- Single quotes
- Semicolons required
- Unix line endings
- No `var` (use `const`/`let`)
- Prefer arrow functions and object shorthand
- Unused vars allowed if prefixed with underscore

### Environment Variables

Required variables (see `.env.example`):
- `PORT` - Server port (defaults to 3000)
- `NODE_ENV` - Environment (development/production)
- `LOG_LEVEL` - Winston log level
- `DATABASE_URL` - PostgreSQL connection string (Neon format)
- `JWT_SECRET` - JWT signing secret (defaults to placeholder, must change for production)

## Health Check

`GET /health` endpoint returns server status, timestamp, and uptime.
