# Node.js Coding Standards

These standards apply to all Node.js API and service projects within EWB.

---

## Language and Tooling

- Use **TypeScript** — JavaScript is not permitted in new code
- Target **Node.js 20 LTS**
- Use **ESLint** with `@typescript-eslint` for linting
- Use **Prettier** for formatting
- Use **Jest** or **Vitest** for testing

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| File | kebab-case | `order-service.ts` |
| Class | PascalCase | `OrderService` |
| Interface | PascalCase | `OrderRepository` |
| Function | camelCase | `getOrderById` |
| Variable | camelCase | `orderCount` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_RETRY_ATTEMPTS` |
| Environment variable | SCREAMING_SNAKE_CASE | `DATABASE_URL` |

---

## Project Structure

```
src/
├── controllers/     # HTTP request handlers — thin, delegate to services
├── services/        # Business logic
├── repositories/    # Data access — database queries
├── middleware/      # Express middleware (auth, validation, error handling)
├── routes/          # Route definitions
├── types/           # TypeScript types and interfaces
├── config/          # Configuration loading from environment variables
└── utils/           # Pure utility functions
tests/
├── unit/
└── integration/
```

---

## TypeScript

- Enable strict mode (`"strict": true` in `tsconfig.json`)
- Avoid `any` — use `unknown` and type guards
- Define explicit return types on service and repository functions
- Keep types close to where they are used; share types via `types/`

---

## Async Patterns

- Use `async`/`await` exclusively — do not mix callbacks and promises
- Handle promise rejections — use `try/catch` in async functions
- Do not swallow errors in catch blocks

```typescript
export async function getOrderById(id: string): Promise<Order> {
  const order = await orderRepository.findById(id);
  if (!order) {
    throw new NotFoundError(`Order ${id} not found`);
  }
  return order;
}
```

---

## Error Handling

- Define custom error classes that extend `Error` for domain errors
- Centralise HTTP error handling in a single Express error-handling middleware
- Do not return stack traces in API responses — log them internally
- Use structured logging with a correlation/request ID

```typescript
// middleware/error-handler.ts
export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  if (err instanceof NotFoundError) {
    return res.status(404).json({ message: err.message });
  }
  logger.error({ err, requestId: req.id }, 'Unhandled error');
  res.status(500).json({ message: 'An unexpected error occurred' });
}
```

---

## Configuration

- Load all configuration from environment variables — never hardcode values
- Validate required environment variables at startup and fail fast if missing
- Use a configuration module that centralises env var access:

```typescript
// config/index.ts
export const config = {
  port: parseInt(process.env.PORT ?? '3000', 10),
  databaseUrl: requireEnv('DATABASE_URL'),
  jwtSecret: requireEnv('JWT_SECRET'),
};

function requireEnv(name: string): string {
  const value = process.env[name];
  if (!value) throw new Error(`Missing required environment variable: ${name}`);
  return value;
}
```

---

## Controllers

Controllers handle HTTP request/response only — they do not contain business logic:

```typescript
export async function getOrder(req: Request, res: Response, next: NextFunction) {
  try {
    const order = await orderService.getOrderById(req.params.id);
    res.json(order);
  } catch (err) {
    next(err);
  }
}
```

---

## Testing

- Unit test service and utility functions in isolation using mocked repositories
- Integration test routes using `supertest` against a real Express app with an in-memory or test database
- Test file naming: `order-service.test.ts` co-located with the source file, or in `tests/unit/`

```typescript
describe('OrderService.getOrderById', () => {
  it('returns the order when found', async () => {
    mockRepo.findById.mockResolvedValue({ id: '1', status: 'pending' });
    const result = await orderService.getOrderById('1');
    expect(result.status).toBe('pending');
  });

  it('throws NotFoundError when order does not exist', async () => {
    mockRepo.findById.mockResolvedValue(null);
    await expect(orderService.getOrderById('999')).rejects.toThrow(NotFoundError);
  });
});
```
