# Testing Strategy Template

This template provides a comprehensive testing strategy for ensuring application quality and reliability.

## Testing Philosophy

### Test Pyramid Approach
Structure your testing strategy following the test pyramid:

1. **Unit Tests (70%)** - Fast, isolated tests for individual functions
2. **Integration Tests (20%)** - Tests for component interactions
3. **End-to-End Tests (10%)** - Full workflow testing

### Testing Principles
- **Fast Feedback** - Tests should run quickly and provide immediate feedback
- **Reliable** - Tests should be deterministic and not flaky
- **Maintainable** - Tests should be easy to update when code changes
- **Comprehensive** - Critical paths should have good test coverage

## Unit Testing Strategy

### Frontend Unit Tests
Focus on testing individual components and utilities:

```typescript
// Component testing with React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import { CreateButton } from './CreateButton';

describe('CreateButton', () => {
  it('should call onClick when clicked', () => {
    const mockOnClick = jest.fn();
    render(<CreateButton onClick={mockOnClick} />);
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });
});
```

### Backend Unit Tests
Test business logic and utilities independently:

```typescript
// API handler testing
import { createMockRequest, createMockResponse } from './test-utils';
import { entityHandler } from './entity-handler';

describe('entityHandler', () => {
  it('should create entity with valid data', async () => {
    const req = createMockRequest({
      method: 'POST',
      body: { name: 'Test Entity' }
    });
    const res = createMockResponse();
    
    await entityHandler(req, res);
    
    expect(res.status).toHaveBeenCalledWith(201);
    expect(res.json).toHaveBeenCalledWith(
      expect.objectContaining({
        data: expect.objectContaining({ name: 'Test Entity' })
      })
    );
  });
});
```

### Test Coverage Goals
- **Critical Business Logic**: 95%+ coverage
- **API Endpoints**: 90%+ coverage
- **UI Components**: 80%+ coverage
- **Utilities**: 90%+ coverage

## Integration Testing Strategy

### API Integration Tests
Test API endpoints with real database interactions:

```typescript
// API integration testing
import { testDb } from './test-database';
import { createTestUser } from './test-fixtures';

describe('Entities API Integration', () => {
  beforeEach(async () => {
    await testDb.clear();
  });

  it('should create and retrieve entity', async () => {
    const user = await createTestUser();
    
    // Create entity
    const createResponse = await request(app)
      .post('/api/v1/entities')
      .set('Authorization', `Bearer ${user.token}`)
      .send({ name: 'Test Entity' })
      .expect(201);
    
    // Retrieve entity
    const getResponse = await request(app)
      .get(`/api/v1/entities/${createResponse.body.data.id}`)
      .set('Authorization', `Bearer ${user.token}`)
      .expect(200);
    
    expect(getResponse.body.data.name).toBe('Test Entity');
  });
});
```

### Database Integration Tests
Test database operations and migrations:

```typescript
// Database integration testing
describe('Database Operations', () => {
  it('should handle entity relationships correctly', async () => {
    const entity = await db.entities.create({
      data: { name: 'Parent Entity' }
    });
    
    const childEntity = await db.entities.create({
      data: { 
        name: 'Child Entity',
        parentId: entity.id 
      }
    });
    
    const entityWithChildren = await db.entities.findUnique({
      where: { id: entity.id },
      include: { children: true }
    });
    
    expect(entityWithChildren.children).toHaveLength(1);
    expect(entityWithChildren.children[0].id).toBe(childEntity.id);
  });
});
```

## End-to-End Testing Strategy

### User Workflow Testing
Test complete user journeys from start to finish:

```typescript
// E2E testing with Playwright
import { test, expect } from '@playwright/test';

test.describe('Entity Management Workflow', () => {
  test('should allow user to create, edit, and delete entity', async ({ page }) => {
    // Login
    await page.goto('/login');
    await page.fill('[data-testid=email]', 'test@example.com');
    await page.fill('[data-testid=password]', 'password');
    await page.click('[data-testid=login-button]');
    
    // Navigate to entities
    await page.click('[data-testid=entities-nav]');
    await expect(page).toHaveURL('/entities');
    
    // Create entity
    await page.click('[data-testid=create-entity-button]');
    await page.fill('[data-testid=entity-name]', 'Test Entity');
    await page.click('[data-testid=save-button]');
    
    // Verify creation
    await expect(page.locator('[data-testid=entity-list]')).toContainText('Test Entity');
    
    // Edit entity
    await page.click('[data-testid=edit-entity-button]');
    await page.fill('[data-testid=entity-name]', 'Updated Entity');
    await page.click('[data-testid=save-button]');
    
    // Verify update
    await expect(page.locator('[data-testid=entity-list]')).toContainText('Updated Entity');
    
    // Delete entity
    await page.click('[data-testid=delete-entity-button]');
    await page.click('[data-testid=confirm-delete]');
    
    // Verify deletion
    await expect(page.locator('[data-testid=entity-list]')).not.toContainText('Updated Entity');
  });
});
```

### Cross-Browser Testing
Test critical workflows across different browsers:

```typescript
// Cross-browser configuration
const browsers = ['chromium', 'firefox', 'webkit'];

browsers.forEach(browserName => {
  test.describe(`Critical workflows on ${browserName}`, () => {
    test.use({ browserName });
    
    test('authentication flow works correctly', async ({ page }) => {
      // Test authentication across browsers
    });
  });
});
```

## Testing Environment Setup

### Test Database Configuration
```typescript
// Test database setup
export class TestDatabase {
  private static instance: TestDatabase;
  
  static async getInstance() {
    if (!this.instance) {
      this.instance = new TestDatabase();
      await this.instance.setup();
    }
    return this.instance;
  }
  
  async setup() {
    // Set up test database with migrations
    await this.runMigrations();
    await this.seedTestData();
  }
  
  async clear() {
    // Clear all test data between tests
    await this.clearAllTables();
  }
  
  async teardown() {
    // Clean up test database
    await this.dropDatabase();
  }
}
```

### Test Fixtures and Factories
```typescript
// Test data factories
export const createTestUser = (overrides = {}) => ({
  id: generateUUID(),
  email: 'test@example.com',
  name: 'Test User',
  role: 'user',
  createdAt: new Date(),
  ...overrides
});

export const createTestEntity = (overrides = {}) => ({
  id: generateUUID(),
  name: 'Test Entity',
  description: 'Test description',
  status: 'active',
  ...overrides
});
```

## Testing Tools Configuration

### Jest/Vitest Configuration
```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.d.ts',
        '**/*.config.*'
      ]
    }
  }
});
```

### Playwright Configuration
```typescript
// playwright.config.ts
export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure'
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } }
  ]
});
```

## Continuous Integration Testing

### CI/CD Pipeline
```yaml
# GitHub Actions workflow
name: Test Suite
on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:coverage
  
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run test:integration
  
  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - run: npx playwright install
      - run: npm run test:e2e
```

## Performance Testing

### Load Testing
```typescript
// Performance testing with Artillery
export const config = {
  target: 'http://localhost:3000',
  phases: [
    { duration: 60, arrivalRate: 5 },  // Warm up
    { duration: 120, arrivalRate: 20 }, // Load test
    { duration: 60, arrivalRate: 50 }   // Stress test
  ]
};

export const scenarios = [
  {
    name: 'Entity CRUD operations',
    weight: 70,
    flow: [
      { post: { url: '/api/v1/entities', json: { name: 'Load Test Entity' } } },
      { get: { url: '/api/v1/entities/{{ id }}' } },
      { put: { url: '/api/v1/entities/{{ id }}', json: { name: 'Updated Entity' } } },
      { delete: { url: '/api/v1/entities/{{ id }}' } }
    ]
  }
];
```

### Frontend Performance Testing
```typescript
// Performance testing with Lighthouse CI
module.exports = {
  ci: {
    collect: {
      numberOfRuns: 3,
      startServerCommand: 'npm start',
      url: ['http://localhost:3000', 'http://localhost:3000/entities']
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }]
      }
    }
  }
};
```

## Test Data Management

### Test Data Isolation
- Use separate test databases
- Clear data between test runs
- Use transactions that rollback
- Implement proper cleanup procedures

### Mock Data Strategy
```typescript
// API mocking with MSW
export const handlers = [
  rest.get('/api/v1/entities', (req, res, ctx) => {
    return res(
      ctx.json({
        data: [
          { id: '1', name: 'Mock Entity 1' },
          { id: '2', name: 'Mock Entity 2' }
        ]
      })
    );
  }),
  
  rest.post('/api/v1/entities', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({
        data: { id: '3', ...req.body }
      })
    );
  })
];
```

## Testing Best Practices

### Test Organization
- Group related tests in describe blocks
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Keep tests independent and isolated

### Test Maintenance
- Update tests when requirements change
- Remove obsolete tests
- Refactor test code like production code
- Document complex test scenarios

### Error Handling Testing
```typescript
// Test error scenarios
describe('Error Handling', () => {
  it('should handle validation errors gracefully', async () => {
    const response = await request(app)
      .post('/api/v1/entities')
      .send({ name: '' }) // Invalid data
      .expect(400);
    
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
    expect(response.body.error.details.fields.name).toBeDefined();
  });
  
  it('should handle database connection errors', async () => {
    // Mock database failure
    jest.spyOn(db, 'entities').mockRejectedValue(new Error('Connection failed'));
    
    const response = await request(app)
      .get('/api/v1/entities')
      .expect(500);
    
    expect(response.body.error.code).toBe('INTERNAL_ERROR');
  });
});
```

## Monitoring Test Health

### Test Metrics
- Test execution time
- Test failure rates
- Code coverage trends
- Flaky test identification

### Test Reporting
```typescript
// Custom test reporter
class TestReporter {
  onRunComplete(contexts, results) {
    const stats = {
      totalTests: results.numTotalTests,
      passedTests: results.numPassedTests,
      failedTests: results.numFailedTests,
      coverage: results.coverageMap?.getCoverageSummary()
    };
    
    // Send metrics to monitoring service
    this.sendMetrics(stats);
  }
}
```

## Adaptation Guidelines

When implementing this testing strategy:

1. **Project Size**: Scale testing complexity with project size
2. **Team Experience**: Provide training for testing tools and practices
3. **CI/CD Integration**: Ensure tests run automatically on code changes
4. **Performance Requirements**: Add performance tests for critical paths
5. **Domain Specific**: Add specialized tests for your business logic

## Testing Checklist

### Before Deployment
- [ ] All unit tests passing
- [ ] Integration tests passing
- [ ] E2E tests for critical paths passing
- [ ] Code coverage meets minimum thresholds
- [ ] Performance tests within acceptable limits
- [ ] Security tests passing
- [ ] Accessibility tests passing

### Regular Maintenance
- [ ] Remove or update obsolete tests
- [ ] Add tests for new features
- [ ] Review and improve flaky tests
- [ ] Update test documentation
- [ ] Monitor test execution times
- [ ] Review test coverage reports