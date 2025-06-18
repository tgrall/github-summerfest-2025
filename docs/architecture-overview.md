# Architecture Overview
# Surfing Trip Application

## System Architecture

### High-Level Architecture Diagram

```mermaid
graph TB
    subgraph "Client Layer"
        A[Web App<br/>Next.js + TypeScript]
        B[Mobile App<br/>PWA/React Native]
    end
    
    subgraph "API Gateway"
        C[API Gateway<br/>Next.js API Routes]
    end
    
    subgraph "Application Layer"
        D[Authentication Service]
        E[Trip Planning Service]
        F[Booking Service]
        G[Community Service]
        H[Weather Service]
        I[Notification Service]
    end
    
    subgraph "Data Layer"
        J[(Primary Database<br/>PostgreSQL)]
        K[(Cache Layer<br/>Redis)]
        L[(File Storage<br/>AWS S3)]
    end
    
    subgraph "External APIs"
        M[Weather APIs<br/>Surfline, OpenWeather]
        N[Booking APIs<br/>Airbnb, Booking.com]
        O[Payment APIs<br/>Stripe]
        P[Maps APIs<br/>Google Maps]
        Q[Email Service<br/>SendGrid]
    end
    
    A --> C
    B --> C
    C --> D
    C --> E
    C --> F
    C --> G
    C --> H
    C --> I
    D --> J
    E --> J
    F --> J
    G --> J
    H --> M
    F --> N
    F --> O
    E --> P
    I --> Q
    D --> K
    E --> K
    G --> L
```

### Technology Stack

#### Frontend
- **Framework**: Next.js 15 with React 19
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui (Radix UI primitives)
- **State Management**: React Context + React Query
- **Forms**: React Hook Form + Zod validation
- **Authentication**: NextAuth.js
- **Maps**: Mapbox GL JS
- **Mobile**: Progressive Web App (PWA) with offline capabilities

#### Backend
- **Runtime**: Node.js
- **Framework**: Next.js API Routes
- **Database**: PostgreSQL with Prisma ORM
- **Cache**: Redis for session management and API caching
- **File Storage**: AWS S3 for images and documents
- **Authentication**: NextAuth.js with JWT tokens
- **Real-time**: Server-Sent Events (SSE) for live updates

#### Infrastructure
- **Hosting**: Vercel (Frontend) + AWS (Backend services)
- **Database**: AWS RDS PostgreSQL
- **Cache**: AWS ElastiCache Redis
- **CDN**: Vercel Edge Network
- **Monitoring**: Vercel Analytics + Sentry
- **CI/CD**: GitHub Actions

### Database Schema

```mermaid
erDiagram
    User {
        id UUID PK
        email string
        password_hash string
        first_name string
        last_name string
        profile_photo_url string
        surf_level enum
        created_at timestamp
        updated_at timestamp
    }
    
    UserProfile {
        id UUID PK
        user_id UUID FK
        preferred_wave_height string
        preferred_wind_conditions string
        budget_range string
        travel_preferences json
        emergency_contacts json
        created_at timestamp
        updated_at timestamp
    }
    
    Destination {
        id UUID PK
        name string
        country string
        region string
        latitude decimal
        longitude decimal
        difficulty_level enum
        best_season string
        description text
        photos json
        created_at timestamp
        updated_at timestamp
    }
    
    SurfSpot {
        id UUID PK
        destination_id UUID FK
        name string
        latitude decimal
        longitude decimal
        break_type enum
        difficulty_level enum
        best_conditions json
        hazards json
        created_at timestamp
        updated_at timestamp
    }
    
    Trip {
        id UUID PK
        user_id UUID FK
        destination_id UUID FK
        title string
        start_date date
        end_date date
        group_size integer
        budget_estimate decimal
        status enum
        itinerary json
        created_at timestamp
        updated_at timestamp
    }
    
    TripParticipant {
        id UUID PK
        trip_id UUID FK
        user_id UUID FK
        role enum
        status enum
        created_at timestamp
    }
    
    Booking {
        id UUID PK
        trip_id UUID FK
        user_id UUID FK
        booking_type enum
        external_booking_id string
        provider string
        amount decimal
        currency string
        status enum
        booking_data json
        created_at timestamp
        updated_at timestamp
    }
    
    Review {
        id UUID PK
        user_id UUID FK
        destination_id UUID FK
        surf_spot_id UUID FK
        rating integer
        title string
        content text
        photos json
        created_at timestamp
        updated_at timestamp
    }
    
    SurfGuide {
        id UUID PK
        user_id UUID FK
        business_name string
        description text
        certifications json
        service_areas json
        pricing json
        availability json
        rating decimal
        created_at timestamp
        updated_at timestamp
    }
    
    WeatherData {
        id UUID PK
        surf_spot_id UUID FK
        date date
        wave_height decimal
        wave_period decimal
        wind_speed decimal
        wind_direction integer
        temperature decimal
        conditions json
        created_at timestamp
    }
    
    User ||--|| UserProfile : has
    User ||--o{ Trip : creates
    User ||--o{ TripParticipant : participates
    User ||--o{ Booking : makes
    User ||--o{ Review : writes
    User ||--|| SurfGuide : can_be
    
    Destination ||--o{ SurfSpot : contains
    Destination ||--o{ Trip : destination_for
    Destination ||--o{ Review : reviewed_for
    
    Trip ||--o{ TripParticipant : has
    Trip ||--o{ Booking : includes
    
    SurfSpot ||--o{ Review : reviewed_for
    SurfSpot ||--o{ WeatherData : has
```

### API Design

#### Authentication Endpoints
```
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
GET  /api/auth/me
PUT  /api/auth/profile
```

#### Destination & Planning Endpoints
```
GET    /api/destinations
GET    /api/destinations/:id
GET    /api/destinations/:id/surf-spots
GET    /api/surf-spots/:id/weather
POST   /api/trips
GET    /api/trips
GET    /api/trips/:id
PUT    /api/trips/:id
DELETE /api/trips/:id
```

#### Booking Endpoints
```
GET  /api/accommodations/search
POST /api/bookings
GET  /api/bookings
GET  /api/bookings/:id
PUT  /api/bookings/:id/cancel
```

#### Community Endpoints
```
GET    /api/reviews
POST   /api/reviews
GET    /api/reviews/:id
PUT    /api/reviews/:id
DELETE /api/reviews/:id
GET    /api/users/:id/profile
POST   /api/messages
GET    /api/messages
```

### Security Architecture

#### Authentication & Authorization
- **JWT Tokens**: Secure token-based authentication
- **Role-Based Access**: User, Guide, Admin roles
- **Session Management**: Redis-based session storage
- **Password Security**: bcrypt hashing with salt
- **Two-Factor Authentication**: Optional TOTP support

#### Data Protection
- **Encryption**: AES-256 encryption for sensitive data at rest
- **TLS/SSL**: All API communications encrypted in transit
- **API Rate Limiting**: Prevent abuse and DDoS attacks
- **Input Validation**: Comprehensive input sanitization
- **CORS**: Configured for security without blocking legitimate requests

#### Privacy & Compliance
- **GDPR Compliance**: Data portability and deletion rights
- **Data Minimization**: Collect only necessary user data
- **Consent Management**: Clear consent for data collection
- **Audit Logging**: Track data access and modifications
- **PCI DSS**: Secure payment processing standards

### Performance & Scalability

#### Caching Strategy
```mermaid
graph LR
    A[Client Request] --> B{Cache Check}
    B -->|Hit| C[Return Cached Data]
    B -->|Miss| D[Fetch from Database]
    D --> E[Update Cache]
    E --> F[Return Data]
    
    subgraph "Cache Layers"
        G[Browser Cache<br/>Static Assets]
        H[CDN Cache<br/>Global Distribution]
        I[Application Cache<br/>Redis]
        J[Database Cache<br/>Query Results]
    end
```

#### Optimization Strategies
- **Code Splitting**: Load only necessary JavaScript
- **Image Optimization**: Next.js Image component with WebP
- **Database Indexing**: Optimized queries with proper indexes
- **CDN Integration**: Global content delivery
- **Lazy Loading**: Components and images loaded on demand
- **API Response Caching**: Redis caching for expensive operations

#### Monitoring & Observability
- **Application Monitoring**: Real-time performance tracking
- **Error Tracking**: Sentry for error monitoring and alerting
- **Database Monitoring**: Query performance and optimization
- **User Analytics**: Usage patterns and feature adoption
- **Uptime Monitoring**: Service availability tracking

### Mobile Strategy

#### Progressive Web App (PWA)
- **Service Workers**: Offline functionality and caching
- **App Manifest**: Native app-like experience
- **Push Notifications**: Real-time alerts and updates
- **Offline Storage**: IndexedDB for offline data
- **Background Sync**: Sync data when connectivity returns

#### Mobile-First Design
- **Responsive Design**: Optimized for all screen sizes
- **Touch Interface**: Gesture-friendly interactions
- **Performance**: Optimized for mobile networks
- **Battery Efficiency**: Minimized background processing
- **App Store Deployment**: PWA installable from browsers

### Integration Architecture

#### Third-Party APIs
```mermaid
graph TB
    subgraph "External Services"
        A[Surfline API<br/>Surf Conditions]
        B[OpenWeatherMap<br/>Weather Data]
        C[Stripe<br/>Payments]
        D[Airbnb API<br/>Accommodations]
        E[Booking.com<br/>Hotels]
        F[Google Maps<br/>Mapping]
        G[SendGrid<br/>Email]
    end
    
    subgraph "Integration Layer"
        H[API Aggregator]
        I[Data Transformer]
        J[Rate Limiter]
        K[Error Handler]
    end
    
    subgraph "Application Services"
        L[Weather Service]
        M[Booking Service]
        N[Notification Service]
        O[Mapping Service]
    end
    
    A --> H
    B --> H
    C --> H
    D --> H
    E --> H
    F --> H
    G --> H
    
    H --> I
    I --> J
    J --> K
    K --> L
    K --> M
    K --> N
    K --> O
```

#### API Gateway Features
- **Request Routing**: Route requests to appropriate services
- **Authentication**: Validate user tokens
- **Rate Limiting**: Prevent API abuse
- **Request/Response Transformation**: Format data consistently
- **Error Handling**: Standardized error responses
- **Logging**: Comprehensive request logging

### Deployment Architecture

#### Development Environment
- **Local Development**: Docker containers for consistency
- **Database**: PostgreSQL container with sample data
- **Cache**: Redis container
- **External APIs**: Mock services for testing

#### Staging Environment
- **Vercel Preview**: Branch-based preview deployments
- **Database**: Separate staging database
- **API Testing**: Automated API testing pipeline
- **Performance Testing**: Load testing with staging data

#### Production Environment
- **Vercel Production**: Global edge deployment
- **AWS RDS**: Production PostgreSQL database
- **AWS ElastiCache**: Production Redis cache
- **AWS S3**: Production file storage
- **Monitoring**: Full observability stack

### Disaster Recovery & Backup

#### Backup Strategy
- **Database Backups**: Daily automated backups with 30-day retention
- **File Storage**: S3 cross-region replication
- **Configuration Backup**: Infrastructure as Code (Terraform)
- **Code Repository**: GitHub with protected main branch

#### Recovery Procedures
- **RTO (Recovery Time Objective)**: 4 hours
- **RPO (Recovery Point Objective)**: 1 hour
- **Failover Process**: Automated failover for critical services
- **Data Recovery**: Point-in-time recovery capabilities

This architecture provides a solid foundation for the Surfing Trip application, ensuring scalability, security, and maintainability while delivering an excellent user experience across all devices and platforms.