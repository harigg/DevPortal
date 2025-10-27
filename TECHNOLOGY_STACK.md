# Technology Stack & Database Design

## Frontend Technology Stack

### Core Framework
- **Next.js 14**: React framework with App Router for optimal performance
- **TypeScript**: Type-safe development with better IDE support
- **React 18**: Latest React with concurrent features and Suspense

### UI & Styling
- **Tailwind CSS**: Utility-first CSS framework for rapid development
- **shadcn/ui**: High-quality, accessible React components
- **Radix UI**: Unstyled, accessible UI primitives
- **Lucide React**: Beautiful, customizable icons

### State Management
- **Temporal Cloud**: Workflow orchestration and state management
- **Temporal Client**: Frontend integration for workflow state
- **React Hook Form**: Form handling with validation

### Development Tools
- **ESLint**: Code linting and quality
- **Prettier**: Code formatting
- **Husky**: Git hooks for code quality
- **Jest**: Unit testing framework
- **Cypress**: End-to-end testing
- **Playwright**: Cross-browser E2E testing
- **Testing Library**: React component testing utilities
- **MSW**: API mocking for tests

## Backend Technology Stack

### Runtime & Framework
- **Node.js 18+**: JavaScript runtime
- **TypeScript**: Type-safe backend development
- **Express.js**: Web framework (for local development)
- **AWS Lambda**: Serverless compute platform

### API & Integration
- **AWS API Gateway**: REST API management
- **OpenAPI 3.0**: API specification standard
- **Swagger UI**: Interactive API documentation
- **Axios**: HTTP client for API calls
- **Temporal Cloud**: Workflow orchestration and state management

### Authentication & Security
- **AWS Cognito**: User authentication and authorization
- **JWT**: JSON Web Tokens for session management
- **bcrypt**: Password hashing
- **helmet**: Security middleware

## Infrastructure as Code (IaC)

### AWS CDK
- **AWS CDK v2**: Infrastructure as Code framework
- **TypeScript**: CDK definitions in TypeScript
- **CDK Constructs**: Reusable infrastructure components
- **CDK Pipelines**: CI/CD for infrastructure deployments

### Infrastructure Components
```typescript
// infrastructure/cdk/lib/auth-stack.ts
export class AuthStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Cognito User Pool
    const userPool = new UserPool(this, 'DevPortalUserPool', {
      userPoolName: 'dev-portal-users',
      selfSignUpEnabled: true,
      signInAliases: {
        email: true,
      },
      passwordPolicy: {
        minLength: 8,
        requireLowercase: true,
        requireUppercase: true,
        requireDigits: true,
        requireSymbols: true,
      },
    });

    // API Gateway
    const api = new RestApi(this, 'DevPortalApi', {
      restApiName: 'dev-portal-api',
      description: 'Developer Portal API',
      defaultCorsPreflightOptions: {
        allowOrigins: Cors.ALL_ORIGINS,
        allowMethods: Cors.ALL_METHODS,
        allowHeaders: ['Content-Type', 'Authorization'],
      },
    });

    // Lambda Functions
    const authFunction = new Function(this, 'AuthFunction', {
      runtime: Runtime.NODEJS_18_X,
      handler: 'index.handler',
      code: Code.fromAsset('backend/src/auth'),
      environment: {
        USER_POOL_ID: userPool.userPoolId,
        USER_POOL_CLIENT_ID: userPoolClient.userPoolClientId,
      },
    });
  }
}
```

### OpenAPI Specifications as Code
```yaml
# specs/auth/auth-api.yaml
openapi: 3.0.3
info:
  title: Authentication API
  description: User authentication and authorization endpoints
  version: 1.0.0
servers:
  - url: https://api.dev-portal.company.com/auth
    description: Production server
paths:
  /login:
    post:
      summary: User login
      description: Authenticate user with email and password
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginRequest'
      responses:
        '200':
          description: Successful login
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LoginResponse'
        '401':
          description: Invalid credentials
components:
  schemas:
    LoginRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8
    LoginResponse:
      type: object
      properties:
        accessToken:
          type: string
        refreshToken:
          type: string
        user:
          $ref: '#/components/schemas/User'
```

## Testing Framework

### Unit Testing
```typescript
// tests/unit/frontend/components/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from '../../../frontend/src/components/LoginForm';

describe('LoginForm', () => {
  it('should render login form with email and password fields', () => {
    render(<LoginForm onLogin={jest.fn()} />);
    
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /login/i })).toBeInTheDocument();
  });

  it('should call onLogin with credentials when form is submitted', async () => {
    const mockOnLogin = jest.fn();
    render(<LoginForm onLogin={mockOnLogin} />);
    
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'test@example.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    
    await waitFor(() => {
      expect(mockOnLogin).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123'
      });
    });
  });
});
```

### Integration Testing
```typescript
// tests/integration/api/auth.test.ts
import { APIGatewayProxyEvent } from 'aws-lambda';
import { handler } from '../../../backend/src/auth/index';

describe('Auth API Integration', () => {
  it('should authenticate valid user credentials', async () => {
    const event: APIGatewayProxyEvent = {
      httpMethod: 'POST',
      path: '/auth/login',
      body: JSON.stringify({
        email: 'test@example.com',
        password: 'password123'
      }),
      headers: {
        'Content-Type': 'application/json'
      }
    } as APIGatewayProxyEvent;

    const result = await handler(event);
    
    expect(result.statusCode).toBe(200);
    expect(JSON.parse(result.body)).toHaveProperty('accessToken');
  });

  it('should reject invalid credentials', async () => {
    const event: APIGatewayProxyEvent = {
      httpMethod: 'POST',
      path: '/auth/login',
      body: JSON.stringify({
        email: 'test@example.com',
        password: 'wrongpassword'
      }),
      headers: {
        'Content-Type': 'application/json'
      }
    } as APIGatewayProxyEvent;

    const result = await handler(event);
    
    expect(result.statusCode).toBe(401);
    expect(JSON.parse(result.body)).toHaveProperty('error');
  });
});
```

### End-to-End Testing
```typescript
// tests/e2e/cypress/integration/user-journey.spec.ts
describe('User Authentication Journey', () => {
  it('should allow user to login and access dashboard', () => {
    cy.visit('/login');
    
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="login-button"]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-menu"]').should('be.visible');
    cy.get('[data-testid="welcome-message"]').should('contain', 'Welcome');
  });

  it('should show error for invalid credentials', () => {
    cy.visit('/login');
    
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="error-message"]').should('be.visible');
    cy.get('[data-testid="error-message"]').should('contain', 'Invalid credentials');
  });
});
```

### Workflow Testing
```typescript
// tests/unit/workflows/auth-workflow.test.ts
import { TestWorkflowEnvironment } from '@temporalio/testing';
import { authWorkflow } from '../../workflows/auth/auth-workflow';

describe('Auth Workflow', () => {
  let testEnv: TestWorkflowEnvironment;

  beforeAll(async () => {
    testEnv = await TestWorkflowEnvironment.createLocal();
  });

  afterAll(async () => {
    await testEnv.teardown();
  });

  it('should complete authentication workflow successfully', async () => {
    const { workflowEnv, mockActivities } = testEnv;
    
    // Mock activities
    mockActivities.validateToken = jest.fn().mockResolvedValue({
      isValid: true,
      userId: 'user123',
      user: { id: 'user123', email: 'test@example.com' }
    });
    
    mockActivities.createSession = jest.fn().mockResolvedValue({
      sessionId: 'session123',
      userId: 'user123',
      expiresAt: '2024-12-31T23:59:59Z'
    });
    
    mockActivities.updateUserActivity = jest.fn().mockResolvedValue(undefined);

    const result = await workflowEnv.run(authWorkflow, {
      args: [{
        headers: { Authorization: 'Bearer valid-token' }
      }],
      taskQueue: 'test-queue',
    });

    expect(result.success).toBe(true);
    expect(result.session).toBeDefined();
    expect(result.user).toBeDefined();
  });
});
```

## Temporal Workflow Architecture

### Workflow Definitions
```typescript
// workflows/auth-workflow.ts
import { proxyActivities, log } from '@temporalio/workflow';
import type * as activities from '../activities/auth-activities';

const { validateToken, createSession, updateUserActivity } = proxyActivities<typeof activities>({
  startToCloseTimeout: '1 minute',
});

export async function authWorkflow(event: APIGatewayProxyEvent): Promise<AuthResult> {
  try {
    const tokenValidation = await validateToken(event.headers.Authorization);
    
    if (!tokenValidation.isValid) {
      return { success: false, error: 'Invalid token' };
    }
    
    const session = await createSession(tokenValidation.userId);
    await updateUserActivity(tokenValidation.userId, 'login');
    
    return { success: true, session, user: tokenValidation.user };
  } catch (error) {
    log.error('Authentication workflow failed', { error });
    throw error;
  }
}
```

### Activity Functions
```typescript
// activities/auth-activities.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand, GetCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export async function validateToken(token: string): Promise<TokenValidation> {
  // JWT token validation logic
  // Return validation result
}

export async function createSession(userId: string): Promise<Session> {
  const session = {
    sessionId: generateSessionId(),
    userId,
    createdAt: new Date().toISOString(),
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(),
  };
  
  await docClient.send(new PutCommand({
    TableName: 'UserSessions',
    Item: session,
  }));
  
  return session;
}

export async function updateUserActivity(userId: string, activity: string): Promise<void> {
  await docClient.send(new PutCommand({
    TableName: 'UserActivities',
    Item: {
      userId,
      activity,
      timestamp: new Date().toISOString(),
    },
  }));
}
```

### Frontend Integration
```typescript
// lib/temporal-client.ts
import { Connection, Client } from '@temporalio/client';

const connection = await Connection.connect({
  address: process.env.TEMPORAL_CLOUD_ADDRESS,
  tls: {
    clientCertPair: {
      crt: process.env.TEMPORAL_CLIENT_CERT,
      key: process.env.TEMPORAL_CLIENT_KEY,
    },
  },
});

export const temporalClient = new Client({
  connection,
  namespace: process.env.TEMPORAL_NAMESPACE,
});

// Usage in React components
export async function startAuthWorkflow(event: APIGatewayProxyEvent) {
  const workflow = await temporalClient.workflow.start(authWorkflow, {
    args: [event],
    taskQueue: 'auth-service',
    workflowId: `auth-${Date.now()}`,
  });
  
  return await workflow.result();
}
```

### Workflow Benefits
- **Reliability**: Automatic retries and error handling
- **Observability**: Built-in monitoring and debugging
- **Scalability**: Handle complex, long-running processes
- **State Persistence**: Maintain state across Lambda invocations
- **Compensation**: Automatic rollback for failed operations

## Database Design

### Primary Database: Amazon DynamoDB

#### Users Table
```typescript
interface User {
  PK: string;           // USER#user123
  SK: string;           // PROFILE
  GSI1PK?: string;      // EMAIL#user@company.com
  GSI1SK?: string;      // USER#user123
  email: string;
  name: string;
  role: 'admin' | 'developer' | 'viewer';
  permissions: string[];
  apiKeys: ApiKey[];
  createdAt: string;
  lastLogin: string;
  isActive: boolean;
  preferences: UserPreferences;
}

interface ApiKey {
  keyId: string;
  name: string;
  permissions: string[];
  createdAt: string;
  lastUsed: string;
  expiresAt?: string;
  isActive: boolean;
}

interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notifications: NotificationSettings;
  dashboard: DashboardSettings;
}
```

#### APIs Table
```typescript
interface API {
  PK: string;           // API#api123
  SK: string;           // VERSION#v1.0.0
  GSI1PK?: string;      // CATEGORY#user-management
  GSI1SK?: string;      // API#api123
  GSI2PK?: string;      // STATUS#active
  GSI2SK?: string;      // API#api123
  name: string;
  version: string;
  description: string;
  baseUrl: string;
  category: string;
  status: 'active' | 'deprecated' | 'beta' | 'draft';
  openApiSpec: OpenAPISpec;
  tags: string[];
  createdBy: string;
  createdAt: string;
  updatedAt: string;
  usage: UsageStats;
  rating: number;
  reviewCount: number;
}

interface UsageStats {
  totalRequests: number;
  uniqueUsers: number;
  averageResponseTime: number;
  errorRate: number;
  lastUsed: string;
}
```

#### Documentation Table
```typescript
interface Documentation {
  PK: string;           // DOC#api123
  SK: string;           // VERSION#v1.0.0
  GSI1PK?: string;      // TYPE#getting-started
  GSI1SK?: string;      // DOC#api123
  apiId: string;
  version: string;
  type: 'getting-started' | 'reference' | 'tutorial' | 'example';
  title: string;
  content: string;
  examples: CodeExample[];
  lastUpdated: string;
  updatedBy: string;
  isPublished: boolean;
  order: number;
}

interface CodeExample {
  language: string;
  title: string;
  code: string;
  description: string;
}
```

#### Analytics Table
```typescript
interface Analytics {
  PK: string;           // ANALYTICS#api123
  SK: string;           // DATE#2024-01-15
  GSI1PK?: string;      // USER#user123
  GSI1SK?: string;      // DATE#2024-01-15
  apiId: string;
  userId: string;
  date: string;
  requests: number;
  errors: number;
  averageResponseTime: number;
  uniqueEndpoints: string[];
  userAgent: string;
  ipAddress: string;
  timestamp: string;
}
```

#### Test Results Table
```typescript
interface TestResult {
  PK: string;           // TEST#test123
  SK: string;           // RUN#run456
  GSI1PK?: string;      // API#api123
  GSI1SK?: string;      // TEST#test123
  testId: string;
  runId: string;
  apiId: string;
  userId: string;
  testName: string;
  status: 'passed' | 'failed' | 'error';
  request: TestRequest;
  response: TestResponse;
  duration: number;
  timestamp: string;
  environment: string;
}

interface TestRequest {
  method: string;
  url: string;
  headers: Record<string, string>;
  body?: any;
  queryParams?: Record<string, string>;
}

interface TestResponse {
  statusCode: number;
  headers: Record<string, string>;
  body: any;
  duration: number;
}
```

#### Comments Table
```typescript
interface Comment {
  PK: string;           // COMMENT#comment123
  SK: string;           // COMMENT#comment123
  GSI1PK?: string;      // API#api123
  GSI1SK?: string;      // COMMENT#comment123
  commentId: string;
  apiId: string;
  userId: string;
  content: string;
  type: 'feedback' | 'question' | 'suggestion' | 'bug-report';
  status: 'open' | 'resolved' | 'closed';
  createdAt: string;
  updatedAt: string;
  replies: CommentReply[];
  likes: number;
  isResolved: boolean;
}

interface CommentReply {
  replyId: string;
  userId: string;
  content: string;
  createdAt: string;
  updatedAt: string;
}
```

### Secondary Storage: Amazon S3

#### Bucket Structure
```
dev-portal-assets/
├── api-docs/
│   ├── {apiId}/
│   │   ├── {version}/
│   │   │   ├── openapi.json
│   │   │   ├── examples/
│   │   │   └── assets/
├── user-uploads/
│   ├── {userId}/
│   │   ├── avatars/
│   │   ├── documents/
│   │   └── exports/
├── static-assets/
│   ├── images/
│   ├── icons/
│   └── templates/
└── backups/
    ├── daily/
    ├── weekly/
    └── monthly/
```

## AWS Services Architecture

### Compute Services
- **AWS Lambda**: Serverless functions for all backend logic
- **AWS Fargate**: Containerized services (if needed)
- **AWS Batch**: Batch processing for analytics

### Storage Services
- **Amazon DynamoDB**: Primary NoSQL database
- **Amazon S3**: Object storage for files and assets
- **Amazon ElastiCache**: Redis for caching and sessions
- **Amazon RDS**: Aurora Serverless (if SQL needed)

### Networking & Content Delivery
- **Amazon CloudFront**: Global CDN
- **AWS API Gateway**: API management
- **Amazon Route 53**: DNS management
- **AWS VPC**: Virtual private cloud

### Security Services
- **AWS Cognito**: Authentication and authorization
- **AWS Secrets Manager**: Secrets and API keys
- **AWS KMS**: Key management
- **AWS WAF**: Web application firewall
- **AWS Shield**: DDoS protection

### Monitoring & Logging
- **Amazon CloudWatch**: Metrics and logging
- **AWS X-Ray**: Distributed tracing
- **AWS CloudTrail**: API call auditing
- **Amazon OpenSearch**: Log analysis

### Additional Services
- **Amazon SNS**: Notifications
- **Amazon SQS**: Message queuing
- **Amazon SES**: Email service
- **AWS Step Functions**: Workflow orchestration
- **AWS Systems Manager**: Parameter store

## Development Environment Setup

### Local Development
```bash
# Frontend setup
npm create next-app@latest dev-portal --typescript --tailwind --eslint
cd dev-portal
npm install @radix-ui/react-* lucide-react @testing-library/react @testing-library/jest-dom

# Backend setup
npm init -y
npm install aws-sdk @types/aws-sdk express cors helmet @temporalio/client @temporalio/worker

# Testing setup
npm install -D @types/node @types/express typescript ts-node nodemon jest @types/jest ts-jest cypress @playwright/test

# Infrastructure setup
npm install -D aws-cdk-lib constructs

# AWS CLI setup
aws configure
aws sts get-caller-identity

# Temporal CLI setup
temporal env create dev-portal
temporal env set dev-portal
```

### Environment Variables
```env
# Frontend (.env.local)
NEXT_PUBLIC_API_URL=https://api.dev-portal.company.com
NEXT_PUBLIC_COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
NEXT_PUBLIC_COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
NEXT_PUBLIC_COGNITO_REGION=us-east-1
NEXT_PUBLIC_TEMPORAL_CLOUD_ADDRESS=your-namespace.tmprl.cloud:7233

# Backend (.env)
AWS_REGION=us-east-1
DYNAMODB_TABLE_PREFIX=dev-portal
S3_BUCKET_NAME=dev-portal-assets
COGNITO_USER_POOL_ID=us-east-1_xxxxxxxxx
JWT_SECRET=your-jwt-secret
TEMPORAL_CLOUD_ADDRESS=your-namespace.tmprl.cloud:7233
TEMPORAL_CLIENT_CERT=your-client-cert
TEMPORAL_CLIENT_KEY=your-client-key
TEMPORAL_NAMESPACE=your-namespace
```

## Performance Optimization

### Frontend Optimization
- **Code Splitting**: Dynamic imports for route-based splitting
- **Image Optimization**: Next.js Image component with WebP support
- **Caching**: React Query for API response caching
- **Bundle Analysis**: Webpack Bundle Analyzer
- **Lazy Loading**: Component and route lazy loading

### Backend Optimization
- **Lambda Optimization**: Provisioned concurrency for critical functions
- **Database Optimization**: Proper DynamoDB indexing and query patterns
- **Caching**: ElastiCache for frequently accessed data
- **Connection Pooling**: RDS Proxy for database connections
- **Cold Start Mitigation**: Lambda warming strategies

### CDN & Caching Strategy
- **CloudFront**: Global edge caching
- **Cache Headers**: Proper cache control headers
- **Static Assets**: S3 + CloudFront for static content
- **API Caching**: API Gateway caching for read-heavy operations

## Security Implementation

### Authentication Flow
```typescript
// JWT Token Structure
interface JWTPayload {
  sub: string;          // User ID
  email: string;
  role: string;
  permissions: string[];
  iat: number;          // Issued at
  exp: number;          // Expires at
  iss: string;          // Issuer
  aud: string;          // Audience
}
```

### API Security
- **Rate Limiting**: API Gateway throttling
- **CORS**: Proper cross-origin resource sharing
- **Input Validation**: Request validation and sanitization
- **SQL Injection Prevention**: Parameterized queries
- **XSS Protection**: Content Security Policy headers

### Data Security
- **Encryption at Rest**: DynamoDB and S3 encryption
- **Encryption in Transit**: TLS 1.2+ for all communications
- **Secrets Management**: AWS Secrets Manager
- **Access Control**: IAM roles and policies
- **Audit Logging**: CloudTrail for all API calls

## Deployment Strategy

### CI/CD Pipeline
```yaml
# .github/workflows/deploy.yml
name: Deploy to AWS
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
      - name: Build application
        run: npm run build
      - name: Deploy to AWS
        run: npm run deploy
```

### Environment Management
- **Development**: Feature branch deployments
- **Staging**: Pre-production testing environment
- **Production**: Stable releases with blue-green deployment
- **Feature Flags**: LaunchDarkly or AWS AppConfig

## Monitoring & Observability

### Application Metrics
- **Performance**: Response times, throughput, error rates
- **Business**: User activity, API usage, feature adoption
- **Infrastructure**: CPU, memory, disk usage
- **Security**: Failed login attempts, suspicious activity

### Alerting Rules
- **Critical**: System down, security breach
- **Warning**: High error rate, slow response times
- **Info**: Deployment success, feature usage milestones

### Logging Strategy
- **Structured Logging**: JSON format for all logs
- **Log Levels**: ERROR, WARN, INFO, DEBUG
- **Correlation IDs**: Request tracing across services
- **Log Retention**: 30 days for debug, 1 year for audit

## Cost Optimization

### Serverless Benefits
- **Pay-per-use**: Only pay for actual usage
- **Auto-scaling**: No idle server costs
- **Managed Services**: Reduced operational overhead
- **Reserved Capacity**: Available for predictable workloads

### Cost Monitoring
- **AWS Cost Explorer**: Budget tracking and analysis
- **CloudWatch Billing**: Real-time cost monitoring
- **Resource Tagging**: Cost allocation and tracking
- **Automated Optimization**: AWS Cost Optimization recommendations

### Estimated Costs (100 developers)
- **Lambda**: $50-100/month
- **DynamoDB**: $25-50/month
- **S3**: $10-20/month
- **CloudFront**: $15-30/month
- **API Gateway**: $35-70/month
- **Cognito**: $5-10/month
- **Temporal Cloud**: $50-100/month
- **Total**: $190-380/month
