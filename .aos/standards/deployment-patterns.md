# Deployment Patterns Template

This template provides standardized deployment patterns for modern web applications.

## Deployment Strategy Overview

### Environment Structure
Maintain consistent environments across the deployment pipeline:

- **Development** - Local development environment
- **Preview/Staging** - Feature testing and integration
- **Production** - Live user-facing environment

### Deployment Philosophy
- **Continuous Integration/Continuous Deployment (CI/CD)**
- **Infrastructure as Code (IaC)**
- **Zero-downtime deployments**
- **Automated rollback capabilities**
- **Environment parity**

## Hosting Platforms

### Frontend Hosting (Recommended)

#### Vercel (Next.js Optimized)
```yaml
# vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["iad1", "sfo1"],
  "functions": {
    "app/api/**/*.ts": {
      "runtime": "nodejs18.x"
    }
  },
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "/api/$1"
    }
  ]
}
```

#### Netlify (Alternative)
```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "out"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200
```

### Backend Hosting

#### Serverless Functions
- **Vercel Functions** - Integrated with frontend
- **AWS Lambda** - Scalable serverless compute
- **Netlify Functions** - JAMstack integration

#### Container Hosting
- **Railway** - Simple container deployment
- **AWS ECS** - Managed container service
- **Google Cloud Run** - Serverless containers

## Database Deployment

### Managed Database Services

#### Supabase (Recommended)
```typescript
// Database configuration
export const supabaseConfig = {
  url: process.env.NEXT_PUBLIC_SUPABASE_URL,
  anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
  serviceKey: process.env.SUPABASE_SERVICE_ROLE_KEY
};

// Connection pooling for serverless
export const supabase = createClient(
  supabaseConfig.url,
  supabaseConfig.serviceKey,
  {
    db: {
      schema: 'public',
    },
    auth: {
      autoRefreshToken: false,
      persistSession: false
    }
  }
);
```

#### Alternative Database Options
- **PlanetScale** - Serverless MySQL
- **AWS RDS** - Managed relational database
- **MongoDB Atlas** - Managed NoSQL database

### Database Migration Strategy
```typescript
// Migration deployment process
export const deployDatabase = async () => {
  // 1. Backup current database
  await backupDatabase();
  
  // 2. Run migrations
  await runMigrations();
  
  // 3. Validate migration success
  await validateMigrations();
  
  // 4. Update application schema
  await updateApplicationSchema();
};
```

## CI/CD Pipeline Implementation

### GitHub Actions (Recommended)
```yaml
# .github/workflows/deploy.yml
name: Deploy Application

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build

  deploy-preview:
    if: github.event_name == 'pull_request'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: vercel/action@v0.1.0
        with:
          token: ${{ secrets.VERCEL_TOKEN }}
          org-id: ${{ secrets.VERCEL_ORG_ID }}
          project-id: ${{ secrets.VERCEL_PROJECT_ID }}

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: vercel/action@v0.1.0
        with:
          token: ${{ secrets.VERCEL_TOKEN }}
          org-id: ${{ secrets.VERCEL_ORG_ID }}
          project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          prod: true
```

### Alternative CI/CD Options
- **GitLab CI/CD** - Integrated with GitLab
- **AWS CodePipeline** - AWS native pipeline
- **Azure DevOps** - Microsoft ecosystem
- **Circle CI** - Third-party CI/CD service

## Environment Configuration

### Environment Variables Management
```bash
# .env.example
# Database
DATABASE_URL="postgresql://..."
NEXT_PUBLIC_SUPABASE_URL="https://..."
NEXT_PUBLIC_SUPABASE_ANON_KEY="..."
SUPABASE_SERVICE_ROLE_KEY="..."

# Authentication
NEXTAUTH_SECRET="..."
NEXTAUTH_URL="..."

# External Services
SENTRY_DSN="..."
POSTHOG_KEY="..."

# File Storage
NEXT_PUBLIC_STORAGE_URL="..."
AWS_ACCESS_KEY_ID="..."
AWS_SECRET_ACCESS_KEY="..."
S3_BUCKET_NAME="..."

# API Keys
STRIPE_SECRET_KEY="..."
SENDGRID_API_KEY="..."
```

### Environment-Specific Configuration
```typescript
// config/environment.ts
const environments = {
  development: {
    apiUrl: 'http://localhost:3000/api',
    databaseUrl: process.env.DEV_DATABASE_URL,
    logLevel: 'debug'
  },
  staging: {
    apiUrl: 'https://staging.example.com/api',
    databaseUrl: process.env.STAGING_DATABASE_URL,
    logLevel: 'info'
  },
  production: {
    apiUrl: 'https://example.com/api',
    databaseUrl: process.env.PROD_DATABASE_URL,
    logLevel: 'warn'
  }
};

export const config = environments[process.env.NODE_ENV || 'development'];
```

## Security in Deployment

### SSL/TLS Configuration
- **Automatic HTTPS** with hosting providers
- **Custom SSL certificates** for custom domains
- **HSTS headers** for security
- **Content Security Policy (CSP)** implementation

### Secrets Management
```typescript
// Secure secrets handling
export const getSecrets = () => {
  const requiredSecrets = [
    'DATABASE_URL',
    'SUPABASE_SERVICE_ROLE_KEY',
    'NEXTAUTH_SECRET'
  ];
  
  const missing = requiredSecrets.filter(secret => !process.env[secret]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required secrets: ${missing.join(', ')}`);
  }
  
  return {
    databaseUrl: process.env.DATABASE_URL,
    supabaseKey: process.env.SUPABASE_SERVICE_ROLE_KEY,
    authSecret: process.env.NEXTAUTH_SECRET
  };
};
```

### Security Headers
```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-XSS-Protection',
    value: '1; mode=block'
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'origin-when-cross-origin'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: securityHeaders,
      },
    ];
  },
};
```

## Monitoring and Observability

### Application Monitoring
```typescript
// monitoring/setup.ts
import * as Sentry from '@sentry/nextjs';

export const initializeMonitoring = () => {
  if (process.env.NODE_ENV === 'production') {
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
      tracesSampleRate: 0.1,
      beforeSend(event) {
        // Filter out sensitive data
        if (event.request?.headers) {
          delete event.request.headers.authorization;
        }
        return event;
      }
    });
  }
};
```

### Health Checks
```typescript
// app/api/health/route.ts
export async function GET() {
  try {
    // Check database connection
    await supabase.from('health_check').select('1').limit(1);
    
    // Check external services
    const checks = await Promise.all([
      checkDatabaseConnection(),
      checkExternalAPIs(),
      checkFileStorage()
    ]);
    
    return Response.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      checks: checks
    });
  } catch (error) {
    return Response.json(
      {
        status: 'unhealthy',
        error: error.message,
        timestamp: new Date().toISOString()
      },
      { status: 503 }
    );
  }
}
```

### Performance Monitoring
```typescript
// monitoring/performance.ts
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export const PerformanceMonitoring = () => (
  <>
    <Analytics />
    <SpeedInsights />
  </>
);
```

## Backup and Recovery

### Database Backup Strategy
```typescript
// scripts/backup-database.ts
export const backupDatabase = async () => {
  const timestamp = new Date().toISOString().split('T')[0];
  const backupName = `backup-${timestamp}`;
  
  try {
    // Create database backup
    await createDatabaseSnapshot(backupName);
    
    // Upload to secure storage
    await uploadBackupToStorage(backupName);
    
    // Clean up old backups (keep last 30 days)
    await cleanupOldBackups(30);
    
    console.log(`Backup created successfully: ${backupName}`);
  } catch (error) {
    console.error('Backup failed:', error);
    throw error;
  }
};
```

### Disaster Recovery Plan
1. **Data Recovery**: Restore from latest backup
2. **Service Recovery**: Redeploy from known good state
3. **DNS Failover**: Route traffic to backup infrastructure
4. **Communication**: Notify users and stakeholders

## Rollback Strategies

### Automated Rollback
```typescript
// scripts/rollback.ts
export const rollbackDeployment = async (version: string) => {
  try {
    // 1. Revert application code
    await revertToVersion(version);
    
    // 2. Rollback database migrations if needed
    await rollbackMigrations(version);
    
    // 3. Clear caches
    await clearApplicationCaches();
    
    // 4. Validate rollback
    await validateRollback();
    
    console.log(`Rollback to ${version} completed successfully`);
  } catch (error) {
    console.error('Rollback failed:', error);
    throw error;
  }
};
```

### Blue-Green Deployment
```typescript
// deployment/blue-green.ts
export const blueGreenDeploy = async () => {
  // 1. Deploy to inactive environment (green)
  await deployToEnvironment('green');
  
  // 2. Run health checks
  await validateEnvironment('green');
  
  // 3. Switch traffic gradually
  await switchTraffic('green', { percentage: 10 });
  await waitAndMonitor(300); // 5 minutes
  
  // 4. Complete switch if healthy
  await switchTraffic('green', { percentage: 100 });
  
  // 5. Mark old environment as inactive
  await markEnvironmentInactive('blue');
};
```

## Performance Optimization

### Build Optimization
```typescript
// next.config.js
module.exports = {
  // Enable compression
  compress: true,
  
  // Optimize images
  images: {
    domains: ['example.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Bundle analyzer
  ...(process.env.ANALYZE === 'true' && {
    webpack: (config) => {
      config.plugins.push(new BundleAnalyzerPlugin());
      return config;
    },
  }),
  
  // Experimental features
  experimental: {
    optimizeCss: true,
    scrollRestoration: true,
  }
};
```

### CDN Configuration
```typescript
// cdn/setup.ts
export const configureCDN = () => ({
  // Cache static assets for 1 year
  '/_next/static/**/*': {
    'Cache-Control': 'public, max-age=31536000, immutable'
  },
  
  // Cache API responses for 5 minutes
  '/api/**/*': {
    'Cache-Control': 'public, max-age=300, s-maxage=300'
  },
  
  // Cache pages for 1 hour
  '/**/*': {
    'Cache-Control': 'public, max-age=3600, s-maxage=3600'
  }
});
```

## Scaling Strategies

### Horizontal Scaling
- **Load balancing** across multiple instances
- **Database read replicas** for read-heavy workloads
- **CDN** for global content delivery
- **Caching layers** (Redis, Memcached)

### Vertical Scaling
- **Increase server resources** when needed
- **Database connection pooling**
- **Optimize database queries**
- **Implement caching strategies**

### Auto-scaling Configuration
```yaml
# Auto-scaling configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Cost Optimization

### Resource Optimization
- **Right-size instances** based on actual usage
- **Use spot instances** for non-critical workloads
- **Implement auto-shutdown** for development environments
- **Optimize database queries** to reduce compute costs

### Monitoring Costs
```typescript
// cost-monitoring/alerts.ts
export const setupCostAlerts = () => {
  // Set up billing alerts
  createBillingAlert({
    threshold: 100, // $100 USD
    emailNotifications: ['admin@example.com']
  });
  
  // Monitor resource usage
  trackResourceUsage([
    'database-connections',
    'api-requests',
    'storage-usage',
    'bandwidth-usage'
  ]);
};
```

## Deployment Checklist

### Pre-Deployment
- [ ] All tests passing in CI/CD
- [ ] Security scan completed
- [ ] Performance tests passed
- [ ] Database migrations tested
- [ ] Environment variables configured
- [ ] Backup created
- [ ] Rollback plan prepared

### Post-Deployment
- [ ] Health checks passing
- [ ] Performance metrics normal
- [ ] Error rates within acceptable limits
- [ ] User-facing features working
- [ ] Database performance stable
- [ ] Monitoring alerts configured
- [ ] Documentation updated

### Emergency Procedures
- [ ] Incident response plan documented
- [ ] Contact information up to date
- [ ] Rollback procedures tested
- [ ] Communication channels established
- [ ] Escalation procedures defined

## Adaptation Guidelines

When implementing these deployment patterns:

1. **Start Simple**: Begin with basic deployment and add complexity as needed
2. **Environment Parity**: Ensure consistency across all environments
3. **Automate Everything**: Reduce manual steps and human error
4. **Monitor Continuously**: Set up comprehensive monitoring from day one
5. **Plan for Failure**: Always have backup and rollback strategies
6. **Document Procedures**: Maintain up-to-date deployment documentation
7. **Test Regularly**: Regularly test deployment and recovery procedures