# Developer Portal - Internal API Management Platform

A comprehensive, serverless developer portal built on AWS for managing and documenting internal APIs. This platform provides developers with interactive documentation, testing capabilities, analytics, and collaboration tools.

## 🚀 Quick Start

### Prerequisites
- AWS Account with appropriate permissions
- Node.js 18+ and npm
- AWS CLI configured
- Git

### Installation
```bash
# Clone the repository
git clone <repository-url>
cd DevPortal

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your AWS configuration

# Deploy infrastructure
npm run deploy:infrastructure

# Deploy application
npm run deploy:app
```

## 📋 Project Overview

This developer portal is designed to:
- **Document APIs**: Interactive documentation with live testing
- **Manage Access**: Role-based access control and API key management
- **Track Usage**: Analytics and performance monitoring
- **Enable Testing**: Built-in testing tools and sandbox environment
- **Foster Collaboration**: Team features and feedback systems

## 🏗️ Architecture

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Developers    │───▶│  CloudFront CDN │───▶│  AWS Amplify    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                        │
                       ┌─────────────────┐              │
                       │  API Gateway    │◀─────────────┘
                       └─────────────────┘
                                │
                       ┌─────────────────┐
                       │  Lambda Functions│
                       └─────────────────┘
                                │
                       ┌─────────────────┐
                       │   DynamoDB      │
                       └─────────────────┘
```

### Key Components
- **Frontend**: Next.js 14 with TypeScript and Tailwind CSS
- **Backend**: AWS Lambda functions with Node.js
- **Database**: Amazon DynamoDB for NoSQL data storage
- **Storage**: Amazon S3 for static assets and file uploads
- **Authentication**: AWS Cognito for user management
- **CDN**: Amazon CloudFront for global content delivery

## 📁 Project Structure

```
DevPortal/
├── docs/                          # Documentation
│   ├── DESIGN_PLAN.md            # Comprehensive design plan
│   ├── ARCHITECTURE.md           # AWS architecture details
│   ├── IMPLEMENTATION_PHASES.md  # Phased implementation plan
│   └── TECHNOLOGY_STACK.md       # Technology choices and database design
├── frontend/                      # Next.js frontend application
│   ├── src/
│   │   ├── app/                  # App Router pages
│   │   ├── components/           # Reusable React components
│   │   ├── lib/                  # Utility functions and configurations
│   │   ├── hooks/                # Custom React hooks
│   │   └── types/                # TypeScript type definitions
│   ├── public/                   # Static assets
│   └── package.json
├── backend/                       # AWS Lambda functions
│   ├── src/
│   │   ├── auth/                 # Authentication service
│   │   ├── api/                  # API management service
│   │   ├── docs/                 # Documentation service
│   │   ├── analytics/            # Analytics service
│   │   └── testing/              # Testing service
│   └── package.json
├── infrastructure/                # Infrastructure as Code
│   ├── cdk/                      # AWS CDK stack definitions
│   │   ├── lib/                  # CDK constructs and stacks
│   │   ├── bin/                  # CDK app entry points
│   │   └── test/                 # CDK infrastructure tests
│   ├── templates/                # CloudFormation templates
│   └── scripts/                  # Deployment scripts
├── workflows/                     # Temporal workflow definitions
│   ├── auth/                     # Authentication workflows
│   ├── api/                      # API management workflows
│   ├── docs/                     # Documentation workflows
│   ├── analytics/                # Analytics workflows
│   └── testing/                  # Testing workflows
├── activities/                    # Temporal activity functions
│   ├── auth/                     # Authentication activities
│   ├── api/                      # API management activities
│   ├── docs/                     # Documentation activities
│   ├── analytics/                # Analytics activities
│   └── testing/                  # Testing activities
├── specs/                        # OpenAPI specifications as code
│   ├── auth/                     # Authentication API specs
│   ├── api/                      # API management specs
│   ├── docs/                     # Documentation API specs
│   ├── analytics/                # Analytics API specs
│   └── testing/                  # Testing API specs
├── tests/                        # Test files (TDD approach)
│   ├── unit/                     # Unit tests
│   │   ├── frontend/             # Frontend unit tests
│   │   ├── backend/              # Backend unit tests
│   │   ├── workflows/            # Workflow unit tests
│   │   └── activities/           # Activity unit tests
│   ├── integration/              # Integration tests
│   │   ├── api/                  # API integration tests
│   │   ├── workflows/            # Workflow integration tests
│   │   └── infrastructure/       # Infrastructure integration tests
│   ├── e2e/                      # End-to-end tests
│   │   ├── cypress/              # Cypress E2E tests
│   │   └── playwright/           # Playwright E2E tests
│   └── fixtures/                  # Test data and fixtures
├── scripts/                      # Build and deployment scripts
│   ├── build/                    # Build scripts
│   ├── deploy/                   # Deployment scripts
│   └── test/                     # Test execution scripts
├── .github/                      # GitHub Actions workflows
│   └── workflows/                # CI/CD pipelines
├── .cursorrules                  # Cursor AI development rules
├── package.json                  # Root package.json for workspace
├── tsconfig.json                 # TypeScript configuration
├── jest.config.js               # Jest testing configuration
├── cypress.config.js            # Cypress E2E configuration
└── README.md                     # This file
```

## 🛠️ Technology Stack

### Frontend
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS + shadcn/ui
- **State Management**: Temporal Workflows (via API calls)
- **Testing**: Jest + Cypress

### Backend
- **Runtime**: Node.js 18+ with TypeScript
- **Platform**: AWS Lambda
- **API**: AWS API Gateway
- **Database**: Amazon DynamoDB
- **Storage**: Amazon S3
- **Workflow Orchestration**: Temporal Cloud

### Infrastructure
- **Hosting**: AWS Amplify
- **CDN**: Amazon CloudFront
- **Authentication**: AWS Cognito
- **Monitoring**: AWS CloudWatch + X-Ray
- **CI/CD**: AWS CodePipeline
- **Workflow Engine**: Temporal Cloud

## 🧪 Test-Driven Development (TDD)

### TDD Approach
This project follows a strict Test-Driven Development methodology:

1. **Red**: Write a failing test first
2. **Green**: Write minimal code to make the test pass
3. **Refactor**: Improve the code while keeping tests green

### Test Structure
```
tests/
├── unit/                     # Fast, isolated tests
│   ├── frontend/            # React component tests
│   ├── backend/             # Lambda function tests
│   ├── workflows/           # Temporal workflow tests
│   └── activities/          # Temporal activity tests
├── integration/             # Service integration tests
│   ├── api/                 # API endpoint tests
│   ├── workflows/           # Workflow integration tests
│   └── infrastructure/      # Infrastructure tests
├── e2e/                     # End-to-end user journey tests
│   ├── cypress/            # Cypress E2E tests
│   └── playwright/         # Playwright E2E tests
└── fixtures/               # Test data and mock objects
```

### Running Tests
```bash
# Run all tests
npm test

# Run unit tests only
npm run test:unit

# Run integration tests
npm run test:integration

# Run E2E tests
npm run test:e2e

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

## 🚀 Deployment

### Development Environment
```bash
# Start local development
npm run dev

# Run tests
npm test

# Build for production
npm run build
```

### Production Deployment
```bash
# Deploy infrastructure
npm run deploy:infrastructure

# Deploy application
npm run deploy:app

# Deploy with environment
npm run deploy:prod
```

## 🏗️ Infrastructure as Code (IaC)

### AWS CDK Stacks
All infrastructure is defined as code using AWS CDK:

```bash
# Deploy infrastructure
npm run deploy:infrastructure

# Deploy specific stack
npm run deploy:stack -- --stack auth-stack

# Destroy infrastructure
npm run destroy:infrastructure

# Synthesize CloudFormation templates
npm run synth
```

### Infrastructure Components
- **Authentication Stack**: Cognito, IAM roles, API Gateway
- **Database Stack**: DynamoDB tables, indexes, and policies
- **Compute Stack**: Lambda functions, API Gateway, CloudFront
- **Monitoring Stack**: CloudWatch, X-Ray, alarms
- **Security Stack**: WAF, Secrets Manager, KMS

### OpenAPI Specifications as Code
All API specifications are maintained as code in the `specs/` directory:

```bash
# Validate OpenAPI specs
npm run validate:specs

# Generate API documentation
npm run generate:docs

# Test API contracts
npm run test:contracts
```

## 📊 Features

### Phase 1: Core Features ✅
- [x] User authentication and authorization
- [x] API documentation with interactive testing
- [x] Basic analytics and usage tracking
- [x] API key management
- [x] Responsive design

### Phase 2: Advanced Features 🚧
- [ ] Sandbox environment for testing
- [ ] Advanced analytics and reporting
- [ ] Team collaboration tools
- [ ] API versioning and change management
- [ ] Search and discovery features

### Phase 3: Enterprise Features 📋
- [ ] OAuth 2.0 / OpenID Connect integration
- [ ] Advanced security policies
- [ ] Compliance and audit reporting
- [ ] Custom SDK generation
- [ ] Webhook management

## 🔄 Temporal Workflow Integration

### Why Temporal?
Temporal provides robust workflow orchestration for complex business processes:

- **Reliability**: Automatic retries, timeouts, and error handling
- **Observability**: Built-in workflow monitoring and debugging UI
- **Scalability**: Handle long-running processes across multiple Lambda invocations
- **State Persistence**: Maintain workflow state across service restarts
- **Compensation**: Automatic rollback for failed operations

### Workflow Use Cases
1. **User Onboarding**: Multi-step registration and setup processes
2. **API Registration**: Validation, documentation generation, and team notification
3. **Analytics Processing**: Complex data aggregation and report generation
4. **Testing Orchestration**: Multi-step API testing with retry logic
5. **Documentation Generation**: Automated documentation updates and publishing

## 💰 Cost Optimization

### Serverless Benefits
- **Pay-per-use**: Only pay for actual usage
- **Auto-scaling**: No idle server costs
- **Managed Services**: Reduced operational overhead

### Estimated Monthly Costs (100 developers)
- **Total**: $190-380/month
- **Lambda**: $50-100
- **DynamoDB**: $25-50
- **S3**: $10-20
- **CloudFront**: $15-30
- **API Gateway**: $35-70
- **Cognito**: $5-10
- **Temporal Cloud**: $50-100

## 🔒 Security

### Authentication & Authorization
- Multi-factor authentication (MFA)
- JWT token-based authentication
- Role-based access control (RBAC)
- API key rotation policies

### Data Protection
- Encryption at rest and in transit
- VPC endpoints for secure communication
- WAF protection against common attacks
- Regular security audits

## 📈 Monitoring & Analytics

### Application Metrics
- API response times and error rates
- User activity and engagement
- Feature utilization
- Performance monitoring

### Business Metrics
- Developer adoption rates
- API usage patterns
- Support ticket reduction
- User satisfaction scores

## 🤝 Contributing

### Development Workflow
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Submit a pull request

### Code Standards
- TypeScript for type safety
- ESLint for code quality
- Prettier for code formatting
- Jest for unit testing
- Cypress for E2E testing

## 📚 Documentation

- [Design Plan](docs/DESIGN_PLAN.md) - Comprehensive design overview
- [Architecture](docs/ARCHITECTURE.md) - AWS architecture details
- [Implementation Phases](docs/IMPLEMENTATION_PHASES.md) - Phased rollout plan
- [Technology Stack](docs/TECHNOLOGY_STACK.md) - Technology choices and database design

## 🆘 Support

### Getting Help
- Check the [documentation](docs/) for detailed information
- Review [existing issues](https://github.com/your-org/dev-portal/issues)
- Create a [new issue](https://github.com/your-org/dev-portal/issues/new) for bugs or feature requests

### Contact
- **Team**: Developer Portal Team
- **Email**: dev-portal@company.com
- **Slack**: #dev-portal-support

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🎯 Roadmap

### Q1 2024
- [ ] Complete Phase 1 implementation
- [ ] Launch beta version
- [ ] Gather user feedback

### Q2 2024
- [ ] Implement Phase 2 features
- [ ] Add advanced analytics
- [ ] Launch production version

### Q3 2024
- [ ] Implement Phase 3 features
- [ ] Add enterprise integrations
- [ ] Scale to 500+ developers

### Q4 2024
- [ ] Advanced AI features
- [ ] Mobile app
- [ ] International expansion

---

**Built with ❤️ by the Developer Portal Team**
