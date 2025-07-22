# API Design Patterns Template

**Version: 1.0.0** | **Last Updated: January 2025** | **AgentOS Standards**

---

This template provides standardized API design patterns for building consistent and maintainable APIs.

## API Structure Patterns

### RESTful Endpoints
Organize endpoints using consistent URL patterns:

```
/api/v1/
  /entities/              # Collection operations
    GET    /               # List entities
    POST   /               # Create entity
    GET    /:id            # Get specific entity
    PUT    /:id            # Update entity
    DELETE /:id            # Delete entity
    
  /entities/:id/
    /relationships/        # Related resource operations
    /actions/             # Entity-specific actions
```

### Versioning Strategy
- Use URL versioning: `/api/v1/`, `/api/v2/`
- Include version headers: `X-API-Version: v1`
- Maintain backward compatibility for at least one major version
- Document deprecation timelines

## Input/Output Validation

### Request Validation
- Use schema validation libraries (Zod, Joi, Yup)
- Validate at API boundaries before processing
- Provide clear validation error messages
- Sanitize inputs to prevent injection attacks

### Response Structure
```typescript
// Success Response
{
  data: any,              // Actual response data
  meta?: {                // Optional metadata
    pagination?: {...},
    timestamps?: {...}
  }
}

// Error Response
{
  error: {
    code: string,         // Machine-readable error code
    message: string,      // Human-readable message
    details?: any,        // Additional error context
    field?: string        // Field-specific errors
  }
}
```

## Error Handling Patterns

### HTTP Status Codes
- `200` - Success with data
- `201` - Created successfully
- `204` - Success with no content
- `400` - Bad request (validation errors)
- `401` - Unauthorized (authentication required)
- `403` - Forbidden (insufficient permissions)
- `404` - Not found
- `409` - Conflict (duplicate/constraint violation)
- `429` - Rate limit exceeded
- `500` - Internal server error

### Error Response Examples
```typescript
// Validation Error
{
  error: {
    code: "VALIDATION_ERROR",
    message: "Invalid input data",
    details: {
      fields: {
        email: "Invalid email format",
        password: "Password too short"
      }
    }
  }
}

// Authorization Error
{
  error: {
    code: "INSUFFICIENT_PERMISSIONS",
    message: "You don't have permission to access this resource",
    details: {
      required_role: "admin",
      current_role: "user"
    }
  }
}
```

## Authentication Patterns

### Token-Based Authentication
- Use JWT tokens for stateless authentication
- Implement refresh token rotation
- Include proper token expiration
- Validate tokens on every protected route

### Authorization Middleware
```typescript
// Role-based access control
const requireRole = (role: string) => {
  return (req, res, next) => {
    if (!req.user || req.user.role !== role) {
      return res.status(403).json({
        error: {
          code: "INSUFFICIENT_PERMISSIONS",
          message: "Access denied"
        }
      });
    }
    next();
  };
};
```

## Database Integration Patterns

### Service Role vs User Context
- **Service Role**: Use for API routes that bypass row-level security
- **User Context**: Use for operations that respect user permissions
- Always validate permissions at the application layer

### Transaction Handling
```typescript
// Wrap multi-step operations in transactions
try {
  await database.transaction(async (trx) => {
    const entity = await trx('entities').insert(data).returning('*');
    await trx('related_table').insert({ entity_id: entity.id, ...relatedData });
    return entity;
  });
} catch (error) {
  // Handle transaction rollback
  throw new APIError('CREATION_FAILED', 'Failed to create entity');
}
```

## Type Safety Patterns

### End-to-End Type Safety
- Define API schemas with validation libraries
- Generate TypeScript types from schemas
- Use type-safe API frameworks (tRPC, GraphQL)
- Share types between client and server

### Schema-First Development
```typescript
// Define schema
const CreateEntitySchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
  tags: z.array(z.string()).default([])
});

// Generate type
type CreateEntityInput = z.infer<typeof CreateEntitySchema>;

// Use in API handler
app.post('/api/v1/entities', async (req, res) => {
  const data = CreateEntitySchema.parse(req.body);
  // data is now properly typed and validated
});
```

## Rate Limiting & Security

### Rate Limiting Strategy
- Implement per-user rate limits
- Use different limits for different endpoints
- Provide clear rate limit headers
- Implement exponential backoff for retries

### Security Headers
```typescript
// Essential security headers
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000');
  next();
});
```

## Logging & Monitoring

### Request Logging
```typescript
// Log all API requests with correlation IDs
app.use((req, res, next) => {
  req.correlationId = generateUUID();
  logger.info('API Request', {
    correlationId: req.correlationId,
    method: req.method,
    url: req.url,
    userAgent: req.get('User-Agent'),
    ip: req.ip
  });
  next();
});
```

### Error Logging
```typescript
// Log errors with context
app.use((error, req, res, next) => {
  logger.error('API Error', {
    correlationId: req.correlationId,
    error: error.message,
    stack: error.stack,
    user: req.user?.id,
    endpoint: req.url
  });
  
  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
      correlationId: req.correlationId
    }
  });
});
```

## Performance Patterns

### Response Caching
- Implement appropriate cache headers
- Use ETags for conditional requests
- Cache expensive operations
- Invalidate cache when data changes

### Pagination
```typescript
// Consistent pagination pattern
interface PaginationParams {
  page?: number;
  limit?: number;
  cursor?: string;
}

interface PaginatedResponse<T> {
  data: T[];
  meta: {
    pagination: {
      total: number;
      page: number;
      limit: number;
      hasNext: boolean;
      hasPrev: boolean;
    };
  };
}
```

## Real-time Features

### WebSocket Integration
- Use WebSocket for real-time updates
- Implement proper connection management
- Add authentication for WebSocket connections
- Handle connection drops gracefully

### Event-Driven Updates
```typescript
// Emit events for real-time updates
eventEmitter.emit('entity:updated', {
  entityId: entity.id,
  changes: updatedFields,
  userId: req.user.id
});
```

## Testing Patterns

### API Testing Strategy
- Unit tests for business logic
- Integration tests for API endpoints
- Contract tests for API schemas
- Load tests for performance validation

### Test Data Management
```typescript
// Use factories for consistent test data
const createTestEntity = (overrides = {}) => ({
  name: 'Test Entity',
  description: 'Test description',
  status: 'active',
  ...overrides
});
```

## Documentation Standards

### API Documentation
- Use OpenAPI/Swagger for API documentation
- Include examples for all endpoints
- Document error responses
- Provide SDK/client library examples

### Code Documentation
- Document complex business logic
- Include JSDoc comments for public functions
- Maintain architectural decision records (ADRs)
- Keep README files up to date

## Implementation Guidelines

1. **Consistency**: Use consistent patterns across all endpoints
2. **Security**: Validate and sanitize all inputs
3. **Performance**: Optimize for common use cases
4. **Reliability**: Handle errors gracefully
5. **Maintainability**: Keep code clean and well-documented

## Adaptation Notes

When implementing these patterns:

1. **Framework Specific**: Adapt to your chosen framework (Express, Fastify, Next.js API routes)
2. **Database Choice**: Adjust patterns for your database (PostgreSQL, MongoDB, etc.)
3. **Authentication Provider**: Integrate with your auth system (Supabase, Auth0, custom)
4. **Deployment Environment**: Consider your hosting environment's constraints
5. **Domain Requirements**: Customize validation rules for your specific domain