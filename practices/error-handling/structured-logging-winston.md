---
concern: error-handling
tech: [winston, node, typescript]
priority: recommended
source-repo: supportforge-platform
applies-to: [node, typescript]
---
# Structured Logging with Winston

## PATTERN
Use Winston for structured JSON logging with: timestamp formatting, error stack capture, default metadata (service name, PID, hostname), environment-aware formatting (pretty for dev, JSON for prod), child loggers per component, and optional transport to external stores (Redis, cloud logging).

## WHY
`console.log` produces unstructured text that's impossible to search, filter, or alert on in production. Winston provides structured JSON output with consistent metadata, log levels, and error stack capture. Child loggers add component context without manual string concatenation. External transports enable centralized log aggregation.

## EXAMPLE
From supportforge-platform (`src/utils/logger.ts`):
```typescript
import winston from 'winston';

const logFormat = winston.format.combine(
  winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss.SSS' }),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

const consoleFormat = winston.format.combine(
  winston.format.colorize(),
  winston.format.timestamp({ format: 'HH:mm:ss.SSS' }),
  winston.format.printf(({ level, message, timestamp, ...meta }) => {
    const metaStr = Object.keys(meta).length > 0
      ? ' ' + JSON.stringify(meta, null, 2) : '';
    return `${timestamp} [${level}]: ${message}${metaStr}`;
  })
);

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: logFormat,
  defaultMeta: {
    service: 'supportforge-platform',
    pid: process.pid,
    hostname: require('os').hostname()
  },
  transports: [
    new winston.transports.Console({
      format: NODE_ENV === 'development' ? consoleFormat : logFormat
    })
  ],
  exceptionHandlers: [/* console transport */],
  rejectionHandlers: [/* console transport */]
});

// Child loggers for component context
export function createComponentLogger(component: string) {
  return logger.child({ component });
}
```

Usage:
```typescript
const log = createComponentLogger('auth');
log.info('User signed in', { userId: user.id, method: 'oauth' });
log.error('Auth failed', { error: err.message, stack: err.stack });
```

## CHECK
How to verify if a repo already follows this:
- [ ] Winston (or equivalent structured logger) is installed
- [ ] Logger produces JSON output in production
- [ ] Default metadata includes service name and hostname
- [ ] Error logging captures stack traces
- [ ] Child loggers or equivalent context mechanism exists

## IMPLEMENT
1. Install winston: `npm install winston`
2. Create `src/utils/logger.ts` with the pattern above
3. Configure environment-aware formatting (pretty dev, JSON prod)
4. Add default metadata (service name, PID, hostname)
5. Create `createComponentLogger()` helper for per-module context
6. Replace all `console.log/error/warn` with logger calls
7. Add `exceptionHandlers` and `rejectionHandlers` for uncaught errors

## NOTES
- Guard against debug logging in production (e.g., `DEBUG_AUTH` flag with production throw guard)
- Console interception (forwarding console.* to Winston) is useful for capturing third-party library output but must guard against recursion
- Log levels: error, warn, info, http, verbose, debug, silly
- For BullMQ workers, include `workerId`, `jobId`, and `duration` in structured logs
