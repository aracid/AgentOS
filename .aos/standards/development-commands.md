# Development Commands Template

**Version: 1.0.0** | **Last Updated: January 2025** | **AgentOS Standards**

---

This template provides standardized development commands for consistent project workflows.

## Common Commands

### Development
- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run linting checks
- `npm run typecheck` - Run TypeScript type checking

### Testing
- `npm test` - Run unit tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report
- `npm run test:e2e` - Run end-to-end tests

### Database Operations
- `npm run db:migrate` - Run database migrations
- `npm run db:seed` - Seed database with initial data
- `npm run db:reset` - Reset database to initial state
- `npm run db:studio` - Open database management interface

### Code Quality
- `npm run format` - Format code with Prettier
- `npm run format:check` - Check code formatting
- `npm run lint:fix` - Fix linting issues automatically

### Deployment
- `npm run deploy` - Deploy to production
- `npm run deploy:staging` - Deploy to staging environment
- `npm run deploy:preview` - Create preview deployment

## Custom Scripts

### Project Specific Commands
> Add project-specific commands here that are unique to your domain

### Environment Management
- `npm run env:setup` - Set up environment variables
- `npm run env:validate` - Validate environment configuration

### Asset Management
- `npm run assets:optimize` - Optimize static assets
- `npm run assets:clean` - Clean generated assets

## Usage Guidelines

1. **Consistency**: Use these standard commands across all projects
2. **Documentation**: Always document custom commands in this section
3. **Naming**: Follow the `category:action` pattern for command names
4. **Environment**: Commands should work across different environments

## Setup Instructions

1. Copy these commands to your `package.json` scripts section
2. Customize the actual implementations for your tech stack
3. Update project-specific commands as needed
4. Ensure all team members are familiar with these commands