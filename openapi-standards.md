# OpenAPI Standards Guide for AI Code Assistants

This document provides comprehensive OpenAPI 3.0+ standards and best practices for AI code assistants to generate high-quality API documentation and implementations.

## Table of Contents
- [Core Principles](#core-principles)
- [Document Structure](#document-structure)
- [Schema Design](#schema-design)
- [Path Operations](#path-operations)
- [Request/Response Patterns](#requestresponse-patterns)
- [Authentication & Security](#authentication--security)
- [Error Handling](#error-handling)
- [Data Types & Validation](#data-types--validation)
- [Documentation Standards](#documentation-standards)
- [Versioning Strategy](#versioning-strategy)
- [Common Patterns](#common-patterns)

## Core Principles

### 1. API-First Design
- Write OpenAPI specifications before implementing code
- Use specifications as the single source of truth
- Generate client SDKs and server stubs from specifications
- Validate implementations against specifications

### 2. Consistency Standards
- Use consistent naming conventions across all endpoints
- Maintain uniform response structures
- Apply standard HTTP status codes consistently
- Follow RESTful principles unless explicitly deviating

### 3. Developer Experience Focus
- Write clear, actionable descriptions
- Provide comprehensive examples
- Include error scenarios and edge cases
- Make specifications human-readable and tool-friendly

## Document Structure

### Basic OpenAPI Document Template
```yaml
openapi: 3.0.3
info:
  title: Your API Name
  version: 1.0.0
  description: |
    Brief description of what this API does.
    
    ## Authentication
    This API uses Bearer token authentication.
    
    ## Rate Limiting
    Rate limits are applied per API key.
  contact:
    name: API Support
    email: api-support@company.com
    url: https://company.com/support
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT
  termsOfService: https://company.com/terms

servers:
  - url: https://api.company.com/v1
    description: Production server
  - url: https://staging-api.company.com/v1
    description: Staging server
  - url: http://localhost:3000/v1
    description: Local development server

paths:
  # Path definitions here

components:
  schemas:
    # Schema definitions here
  responses:
    # Reusable responses here
  parameters:
    # Reusable parameters here
  examples:
    # Reusable examples here
  requestBodies:
    # Reusable request bodies here
  headers:
    # Reusable headers here
  securitySchemes:
    # Security schemes here

security:
  - BearerAuth: []

tags:
  - name: Users
    description: User management operations
  - name: Orders
    description: Order processing operations
```

### Required Info Section Fields
Always include these fields in the `info` section:
- `title`: Clear, descriptive API name
- `version`: Semantic versioning (e.g., "1.2.3")
- `description`: Multi-line description with key information
- `contact`: Support contact information
- `license`: License information (if applicable)

## Schema Design

### Schema Naming Conventions
- **PascalCase** for schema names: `UserProfile`, `OrderItem`
- **camelCase** for property names: `firstName`, `createdAt`
- **SCREAMING_SNAKE_CASE** for enum values: `PENDING`, `COMPLETED`

### Base Schema Patterns

#### Resource Schema Template
```yaml
components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
        - createdAt
      properties:
        id:
          type: string
          format: uuid
          description: Unique identifier for the user
          example: "123e4567-e89b-12d3-a456-426614174000"
        email:
          type: string
          format: email
          description: User's email address
          example: "john.doe@example.com"
        firstName:
          type: string
          minLength: 1
          maxLength: 50
          description: User's first name
          example: "John"
        lastName:
          type: string
          minLength: 1
          maxLength: 50
          description: User's last name
          example: "Doe"
        createdAt:
          type: string
          format: date-time
          description: When the user was created
          example: "2023-07-30T12:34:56Z"
          readOnly: true
        updatedAt:
          type: string
          format: date-time
          description: When the user was last updated
          example: "2023-07-30T12:34:56Z"
          readOnly: true
```

#### Pagination Schema
```yaml
components:
  schemas:
    PaginatedUsers:
      type: object
      required:
        - data
        - pagination
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/PaginationMeta'
    
    PaginationMeta:
      type: object
      required:
        - page
        - perPage
        - total
        - totalPages
      properties:
        page:
          type: integer
          minimum: 1
          description: Current page number
          example: 2
        perPage:
          type: integer
          minimum: 1
          maximum: 100
          description: Number of items per page
          example: 20
        total:
          type: integer
          minimum: 0
          description: Total number of items
          example: 150
        totalPages:
          type: integer
          minimum: 0
          description: Total number of pages
          example: 8
        hasNext:
          type: boolean
          description: Whether there are more pages
          example: true
        hasPrevious:
          type: boolean
          description: Whether there are previous pages
          example: true
```

#### Error Schema
```yaml
components:
  schemas:
    Error:
      type: object
      required:
        - error
        - message
        - statusCode
      properties:
        error:
          type: string
          description: Error type identifier
          example: "VALIDATION_ERROR"
        message:
          type: string
          description: Human-readable error message
          example: "The email field is required"
        statusCode:
          type: integer
          description: HTTP status code
          example: 400
        details:
          type: object
          description: Additional error details
          additionalProperties: true
          example:
            field: "email"
            code: "REQUIRED"
        timestamp:
          type: string
          format: date-time
          description: When the error occurred
          example: "2023-07-30T12:34:56Z"
        traceId:
          type: string
          description: Request trace identifier for debugging
          example: "abc123def456"
```

## Path Operations

### RESTful Resource Patterns
Follow these standard patterns for CRUD operations:

```yaml
paths:
  /users:
    get:
      summary: List users
      description: Retrieve a paginated list of users
      tags: [Users]
      parameters:
        - $ref: '#/components/parameters/PageQuery'
        - $ref: '#/components/parameters/PerPageQuery'
        - name: search
          in: query
          description: Search users by name or email
          schema:
            type: string
            maxLength: 100
          example: "john"
      responses:
        '200':
          description: Users retrieved successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PaginatedUsers'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
    
    post:
      summary: Create user
      description: Create a new user account
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
            examples:
              basic:
                summary: Basic user creation
                value:
                  email: "jane.doe@example.com"
                  firstName: "Jane"
                  lastName: "Doe"
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
          headers:
            Location:
              description: URL of the created user
              schema:
                type: string
                format: uri
                example: "/users/123e4567-e89b-12d3-a456-426614174000"
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /users/{userId}:
    parameters:
      - $ref: '#/components/parameters/UserIdPath'
    
    get:
      summary: Get user by ID
      description: Retrieve a specific user by their unique identifier
      tags: [Users]
      responses:
        '200':
          description: User retrieved successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
    
    put:
      summary: Update user
      description: Update an existing user (full replacement)
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUserRequest'
      responses:
        '200':
          description: User updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
    
    patch:
      summary: Partially update user
      description: Update specific fields of an existing user
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/PatchUserRequest'
      responses:
        '200':
          description: User updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
    
    delete:
      summary: Delete user
      description: Permanently delete a user account
      tags: [Users]
      responses:
        '204':
          description: User deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'
```

### Non-CRUD Operations
For operations that don't fit CRUD patterns:

```yaml
paths:
  /users/{userId}/activate:
    post:
      summary: Activate user account
      description: Activate a user's account and send confirmation email
      tags: [Users]
      parameters:
        - $ref: '#/components/parameters/UserIdPath'
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                sendEmail:
                  type: boolean
                  description: Whether to send activation email
                  default: true
      responses:
        '200':
          description: User activated successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  message:
                    type: string
                    example: "User account activated successfully"
                  emailSent:
                    type: boolean
                    example: true

  /users/bulk:
    post:
      summary: Bulk create users
      description: Create multiple users in a single operation
      tags: [Users]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - users
              properties:
                users:
                  type: array
                  minItems: 1
                  maxItems: 100
                  items:
                    $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '207':
          description: Multi-status response for bulk operation
          content:
            application/json:
              schema:
                type: object
                properties:
                  results:
                    type: array
                    items:
                      type: object
                      properties:
                        index:
                          type: integer
                        status:
                          type: integer
                        user:
                          $ref: '#/components/schemas/User'
                        error:
                          $ref: '#/components/schemas/Error'
```

## Request/Response Patterns

### Standard Request Bodies
```yaml
components:
  schemas:
    CreateUserRequest:
      type: object
      required:
        - email
        - firstName
        - lastName
      properties:
        email:
          type: string
          format: email
          description: User's email address
          example: "jane.doe@example.com"
        firstName:
          type: string
          minLength: 1
          maxLength: 50
          description: User's first name
          example: "Jane"
        lastName:
          type: string
          minLength: 1
          maxLength: 50
          description: User's last name
          example: "Doe"
        phoneNumber:
          type: string
          pattern: '^\+[1-9]\d{1,14}$'
          description: User's phone number in E.164 format
          example: "+1234567890"
      additionalProperties: false

    UpdateUserRequest:
      type: object
      required:
        - email
        - firstName
        - lastName
      properties:
        email:
          type: string
          format: email
        firstName:
          type: string
          minLength: 1
          maxLength: 50
        lastName:
          type: string
          minLength: 1
          maxLength: 50
        phoneNumber:
          type: string
          pattern: '^\+[1-9]\d{1,14}$'
          nullable: true
      additionalProperties: false

    PatchUserRequest:
      type: object
      properties:
        email:
          type: string
          format: email
        firstName:
          type: string
          minLength: 1
          maxLength: 50
        lastName:
          type: string
          minLength: 1
          maxLength: 50
        phoneNumber:
          type: string
          pattern: '^\+[1-9]\d{1,14}$'
          nullable: true
      additionalProperties: false
      minProperties: 1
```

### Common Parameters
```yaml
components:
  parameters:
    UserIdPath:
      name: userId
      in: path
      required: true
      description: Unique identifier for the user
      schema:
        type: string
        format: uuid
      example: "123e4567-e89b-12d3-a456-426614174000"

    PageQuery:
      name: page
      in: query
      description: Page number for pagination
      schema:
        type: integer
        minimum: 1
        default: 1
      example: 2

    PerPageQuery:
      name: perPage
      in: query
      description: Number of items per page
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
      example: 50

    SortQuery:
      name: sort
      in: query
      description: Sort field and direction
      schema:
        type: string
        enum: [createdAt, -createdAt, name, -name, email, -email]
        default: -createdAt
      example: "name"

    FieldsQuery:
      name: fields
      in: query
      description: Comma-separated list of fields to include in response
      schema:
        type: string
      example: "id,email,firstName"
```

### Standard Response Templates
```yaml
components:
  responses:
    BadRequest:
      description: Invalid request parameters or body
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          examples:
            validation:
              summary: Validation error
              value:
                error: "VALIDATION_ERROR"
                message: "Request validation failed"
                statusCode: 400
                details:
                  email: ["The email field is required"]
                timestamp: "2023-07-30T12:34:56Z"

    Unauthorized:
      description: Authentication required or invalid
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          examples:
            missing_token:
              summary: Missing authentication token
              value:
                error: "UNAUTHORIZED"
                message: "Authentication token is required"
                statusCode: 401
                timestamp: "2023-07-30T12:34:56Z"

    Forbidden:
      description: Insufficient permissions
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          examples:
            user_not_found:
              summary: User not found
              value:
                error: "NOT_FOUND"
                message: "User with ID '123' not found"
                statusCode: 404
                timestamp: "2023-07-30T12:34:56Z"

    Conflict:
      description: Resource conflict (e.g., duplicate email)
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    TooManyRequests:
      description: Rate limit exceeded
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
      headers:
        Retry-After:
          description: Number of seconds to wait before retrying
          schema:
            type: integer
          example: 60

    InternalServerError:
      description: Internal server error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

## Authentication & Security

### Security Schemes
```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        JWT token authentication. Include the token in the Authorization header:
        `Authorization: Bearer <token>`

    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: |
        API key authentication. Include your API key in the X-API-Key header.

    OAuth2:
      type: oauth2
      description: OAuth2 authentication
      flows:
        authorizationCode:
          authorizationUrl: https://auth.company.com/oauth/authorize
          tokenUrl: https://auth.company.com/oauth/token
          scopes:
            read: Read access to resources
            write: Write access to resources
            admin: Administrative access

# Apply security globally or per-operation
security:
  - BearerAuth: []
  # - ApiKeyAuth: []
  # - OAuth2: [read, write]
```

### Security Best Practices
- Always specify security requirements
- Document authentication flows clearly
- Include security examples in operation descriptions
- Use HTTPS for all production endpoints
- Implement proper scope-based authorization for OAuth2

## Error Handling

### HTTP Status Code Guidelines

#### 2xx Success
- **200 OK**: Successful GET, PUT, PATCH requests
- **201 Created**: Successful POST requests that create resources
- **202 Accepted**: Async operations accepted for processing
- **204 No Content**: Successful DELETE requests

#### 4xx Client Errors
- **400 Bad Request**: Invalid request syntax or parameters
- **401 Unauthorized**: Authentication required or failed
- **403 Forbidden**: Valid request but insufficient permissions
- **404 Not Found**: Resource doesn't exist
- **405 Method Not Allowed**: HTTP method not supported for endpoint
- **409 Conflict**: Request conflicts with current resource state
- **422 Unprocessable Entity**: Valid syntax but semantic errors
- **429 Too Many Requests**: Rate limit exceeded

#### 5xx Server Errors
- **500 Internal Server Error**: Unexpected server error
- **502 Bad Gateway**: Invalid response from upstream server
- **503 Service Unavailable**: Server temporarily unavailable

### Error Response Format
Always use consistent error format across all endpoints:

```yaml
components:
  schemas:
    ValidationError:
      allOf:
        - $ref: '#/components/schemas/Error'
        - type: object
          properties:
            details:
              type: object
              additionalProperties:
                type: array
                items:
                  type: string
              example:
                email: ["The email field is required", "The email format is invalid"]
                age: ["The age must be at least 18"]
```

## Data Types & Validation

### String Formats
Use appropriate string formats for validation:

```yaml
properties:
  email:
    type: string
    format: email
  url:
    type: string
    format: uri
  date:
    type: string
    format: date          # YYYY-MM-DD
  datetime:
    type: string
    format: date-time     # RFC 3339
  uuid:
    type: string
    format: uuid
  password:
    type: string
    format: password
    minLength: 8
    pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]'
```

### Number Validation
```yaml
properties:
  age:
    type: integer
    minimum: 0
    maximum: 150
  price:
    type: integer
    description: "Price in the smallest currency unit (e.g., cents)."
    minimum: 0
    multipleOf: 1       # For currency precision
  percentage:
    type: number
    minimum: 0
    maximum: 100
```

### Array Validation
```yaml
properties:
  tags:
    type: array
    items:
      type: string
      minLength: 1
      maxLength: 50
    minItems: 1
    maxItems: 10
    uniqueItems: true
  coordinates:
    type: array
    items:
      type: number
    minItems: 2
    maxItems: 2
    description: "[longitude, latitude]"
```

### Enum Definitions
```yaml
components:
  schemas:
    OrderStatus:
      type: string
      enum:
        - PENDING
        - CONFIRMED
        - SHIPPED
        - DELIVERED
        - CANCELLED
      description: |
        Order status values:
        * `PENDING` - Order placed but not confirmed
        * `CONFIRMED` - Order confirmed and being prepared
        * `SHIPPED` - Order shipped to customer
        * `DELIVERED` - Order delivered successfully
        * `CANCELLED` - Order cancelled
```

## Documentation Standards

### Description Guidelines
Write clear, actionable descriptions:

```yaml
paths:
  /users:
    get:
      summary: List users
      description: |
        Retrieve a paginated list of users with optional filtering and sorting.
        
        ## Filtering
        Use query parameters to filter results:
        - `search`: Search by name or email (case-insensitive)
        - `status`: Filter by user status
        
        ## Sorting
        Use the `sort` parameter with field names. Prefix with `-` for descending order:
        - `createdAt`: Sort by creation date (oldest first)
        - `-createdAt`: Sort by creation date (newest first)
        
        ## Pagination
        Results are paginated with a default page size of 20 items.
        Use `page` and `perPage` parameters to navigate through results.
```

### Example Guidelines
Always provide realistic, helpful examples:

```yaml
components:
  examples:
    UserExample:
      summary: Complete user object
      description: Example of a fully populated user with all optional fields
      value:
        id: "123e4567-e89b-12d3-a456-426614174000"
        email: "john.doe@example.com"
        firstName: "John"
        lastName: "Doe"
        phoneNumber: "+1234567890"
        status: "ACTIVE"
        createdAt: "2023-07-30T12:34:56Z"
        updatedAt: "2023-07-30T12:34:56Z"

    MinimalUserExample:
      summary: Minimal user object
      description: Example with only required fields
      value:
        id: "987fcdeb-51a2-43d1-9f12-345678901234"
        email: "jane.smith@example.com"
        firstName: "Jane"
        lastName: "Smith"
        status: "PENDING"
        createdAt: "2023-07-30T12:34:56Z"
        updatedAt: "2023-07-30T12:34:56Z"
```

## Versioning Strategy

### URL-based Versioning
```yaml
servers:
  - url: https://api.company.com/v1
    description: Version 1 (current)
  - url: https://api.company.com/v2
    description: Version 2 (beta)

info:
  version: 1.2.3  # API version
```

### Header-based Versioning (Alternative)
```yaml
parameters:
  - name: API-Version
    in: header
    schema:
      type: string
      enum: ['v1', 'v2']
      default: 'v1'
```

### Deprecation Handling
```yaml
paths:
  /old-endpoint:
    get:
      deprecated: true
      summary: Get data (deprecated)
      description: |
        **⚠️ DEPRECATED**: This endpoint is deprecated and will be removed in v2.
        Use `/new-endpoint` instead.
        
        **Removal date**: 2024-01-01
```

## Common Patterns

### Health Check Endpoint
```yaml
paths:
  /health:
    get:
      summary: Health check
      description: Check API service health and dependencies
      tags: [System]
      security: []  # No authentication required
      responses:
        '200':
          description: Service is healthy
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    enum: [healthy, degraded, unhealthy]
                  timestamp:
                    type: string
                    format: date-time
                  version:
                    type: string
                  dependencies:
                    type: object
                    additionalProperties:
                      type: object
                      properties:
                        status:
                          type: string
                          enum: [up, down]
                        responseTime:
                          type: number
                          description: Response time in milliseconds
```

### File Upload Endpoint
```yaml
paths:
  /users/{userId}/avatar:
    post:
      summary: Upload user avatar
      description: Upload a new avatar image for the user
      tags: [Users]
      parameters:
        - $ref: '#/components/parameters/UserIdPath'
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              required:
                - file
              properties:
                file:
                  type: string
                  format: binary
                  description: Avatar image file
                description:
                  type: string
                  maxLength: 200
                  description: Optional description of the avatar
            encoding:
              file:
                contentType: image/png, image/jpeg, image/gif
      responses:
        '200':
          description: Avatar uploaded successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  url:
                    type: string
                    format: uri
                    description: URL of the uploaded avatar
                  size:
                    type: integer
                    description: File size in bytes
```

### Webhook Definitions
```yaml
webhooks:
  userCreated:
    post:
      summary: User created webhook
      description: Fired when a new user is created
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                event:
                  type: string
                  enum: [user.created]
                timestamp:
                  type: string
                  format: date-time
                data:
                  $ref: '#/components/schemas/User'
      responses:
        '200':
          description: Webhook received successfully
```

## Quality Checklist

Before finalizing your OpenAPI specification:

### Structure & Organization
- [ ] Document follows OpenAPI 3.0+ specification
- [ ] All required fields in `info` section are present
- [ ] Components are properly organized and reused
- [ ] Tags are defined and used consistently
- [ ] Servers are properly configured for different environments

### Schema Design
- [ ] All schemas have clear, descriptive names
- [ ] Required fields are properly marked
- [ ] Appropriate validation rules are applied
- [ ] Examples are provided for all schemas
- [ ] Enum values are documented with descriptions

### Path Operations
- [ ] RESTful conventions are followed
- [ ] HTTP methods are used correctly
- [ ] All parameters are properly typed and documented
- [ ] Response codes are appropriate and consistent
- [ ] Request/response examples are realistic

### Documentation Quality
- [ ] All descriptions are clear and actionable
- [ ] Examples cover common use cases
- [ ] Error scenarios are documented
- [ ] Authentication requirements are clear
- [ ] Deprecation warnings are included where needed

### Security & Validation
- [ ] Security schemes are properly defined
- [ ] Authentication requirements are specified
- [ ] Input validation rules are comprehensive
- [ ] Error responses follow consistent format
- [ ] Rate limiting is documented

### Developer Experience
- [ ] Specification is human-readable
- [ ] Code generation works without issues
- [ ] Mock servers can be generated
- [ ] Documentation is comprehensive but not overwhelming

---

This guide ensures that AI code assistants generate OpenAPI specifications that are comprehensive, consistent, and developer-friendly while following industry best practices.