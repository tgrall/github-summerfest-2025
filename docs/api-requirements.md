# API Requirements and Data Models

This document outlines the API requirements and data models needed to support the Surfing Trip application features.

## 1. Core Data Models

### User Model
```typescript
interface User {
  id: string;
  email: string;
  username: string;
  firstName: string;
  lastName: string;
  profileImage?: string;
  surfingExperience: 'beginner' | 'intermediate' | 'advanced' | 'expert';
  preferredSurfStyle: string[];
  location: {
    latitude: number;
    longitude: number;
    city: string;
    country: string;
  };
  emergencyContacts: EmergencyContact[];
  privacySettings: PrivacySettings;
  createdAt: Date;
  updatedAt: Date;
}
```

### Surf Spot Model
```typescript
interface SurfSpot {
  id: string;
  name: string;
  description: string;
  location: {
    latitude: number;
    longitude: number;
    address: string;
    city: string;
    country: string;
  };
  difficulty: 'beginner' | 'intermediate' | 'advanced' | 'expert';
  waveType: 'beach_break' | 'reef_break' | 'point_break' | 'river_mouth';
  bestConditions: {
    swellDirection: string[];
    windDirection: string[];
    tideStage: string[];
    season: string[];
  };
  amenities: string[];
  hazards: string[];
  images: Image[];
  averageRating: number;
  totalReviews: number;
  accessInfo: string;
  localRules: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Booking Model
```typescript
interface Booking {
  id: string;
  userId: string;
  type: 'accommodation' | 'lesson' | 'equipment' | 'experience';
  providerId: string;
  itemId: string;
  startDate: Date;
  endDate: Date;
  status: 'pending' | 'confirmed' | 'cancelled' | 'completed';
  totalAmount: number;
  currency: string;
  paymentStatus: 'pending' | 'paid' | 'refunded';
  participants: number;
  specialRequests?: string;
  cancellationPolicy: string;
  confirmationCode: string;
  createdAt: Date;
  updatedAt: Date;
}
```

## 2. API Endpoints

### Authentication APIs
```typescript
POST /api/auth/register
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh
POST /api/auth/forgot-password
POST /api/auth/reset-password
```

### User Management APIs
```typescript
GET /api/users/profile
PUT /api/users/profile
DELETE /api/users/account
GET /api/users/:id/public-profile
POST /api/users/upload-avatar
```

### Surf Spots APIs
```typescript
GET /api/surf-spots
GET /api/surf-spots/:id
POST /api/surf-spots/:id/reviews
GET /api/surf-spots/:id/reviews
GET /api/surf-spots/:id/conditions
GET /api/surf-spots/search
GET /api/surf-spots/nearby
```

### Booking APIs
```typescript
POST /api/bookings
GET /api/bookings/user/:userId
GET /api/bookings/:id
PUT /api/bookings/:id/cancel
GET /api/bookings/:id/confirmation
```

### Weather and Conditions APIs
```typescript
GET /api/conditions/current/:spotId
GET /api/conditions/forecast/:spotId
GET /api/conditions/tides/:spotId
GET /api/conditions/wind/:spotId
```

## 3. Third-Party Integrations

### Weather Data Providers
- **Primary**: OpenWeatherMap API
- **Backup**: WeatherAPI
- **Surf-specific**: Surfline API, Magic Seaweed API

### Payment Processing
- **Stripe**: Primary payment processor
- **PayPal**: Alternative payment method
- **Regional**: Local payment gateways

### Mapping and Location
- **Google Maps**: Primary mapping service
- **Mapbox**: Alternative for custom styling
- **Places API**: Location search and details

### Communication
- **SendGrid**: Email notifications
- **Twilio**: SMS notifications
- **Firebase**: Push notifications

## 4. Real-time Features

### WebSocket Events
```typescript
// Weather updates
'weather-update': {
  spotId: string;
  conditions: WeatherConditions;
  timestamp: Date;
}

// Booking confirmations
'booking-confirmed': {
  bookingId: string;
  userId: string;
  details: BookingDetails;
}

// Safety alerts
'safety-alert': {
  userId: string;
  location: Location;
  alertType: 'check-in-missed' | 'emergency';
}
```

## 5. Data Validation Schemas

### User Registration
```typescript
const UserRegistrationSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  firstName: z.string().min(1).max(50),
  lastName: z.string().min(1).max(50),
  surfingExperience: z.enum(['beginner', 'intermediate', 'advanced', 'expert']),
  agreeToTerms: z.boolean().refine(val => val === true)
});
```

### Surf Spot Search
```typescript
const SurfSpotSearchSchema = z.object({
  location: z.string().optional(),
  radius: z.number().min(1).max(100).default(25),
  difficulty: z.array(z.enum(['beginner', 'intermediate', 'advanced', 'expert'])).optional(),
  waveType: z.array(z.string()).optional(),
  amenities: z.array(z.string()).optional(),
  sortBy: z.enum(['distance', 'rating', 'popularity']).default('distance')
});
```

## 6. Error Handling

### Standard Error Response
```typescript
interface ApiError {
  error: {
    code: string;
    message: string;
    details?: any;
    timestamp: Date;
    requestId: string;
  }
}
```

### Common Error Codes
- `VALIDATION_ERROR`: Input validation failed
- `UNAUTHORIZED`: Authentication required
- `FORBIDDEN`: Insufficient permissions
- `NOT_FOUND`: Resource not found
- `RATE_LIMITED`: Too many requests
- `EXTERNAL_SERVICE_ERROR`: Third-party service failure

## 7. Performance Considerations

### Caching Strategy
- **Redis**: Session data, frequently accessed spot data
- **CDN**: Images, static assets
- **Database**: Query result caching for expensive operations

### Rate Limiting
- **Authentication**: 5 attempts per 15 minutes per IP
- **API calls**: 1000 requests per hour per user
- **Search**: 60 requests per minute per user

### Pagination
- **Default page size**: 20 items
- **Maximum page size**: 100 items
- **Cursor-based pagination** for real-time feeds

## 8. Security Requirements

### Data Encryption
- **At rest**: AES-256 encryption for sensitive data
- **In transit**: TLS 1.3 for all API communications
- **PII**: Additional encryption layer for personal information

### Access Control
- **JWT tokens**: 15-minute access tokens, 7-day refresh tokens
- **Role-based access**: User, Local Expert, Admin roles
- **API key management**: For partner integrations

### Input Sanitization
- **SQL injection**: Parameterized queries only
- **XSS prevention**: Input sanitization and output encoding
- **File uploads**: Type validation and virus scanning

## 9. Monitoring and Analytics

### Key Metrics
- **API response times**: 95th percentile < 200ms
- **Error rates**: < 0.1% for critical endpoints
- **User engagement**: Session duration, feature usage
- **Business metrics**: Booking conversion rates, revenue

### Logging
- **Structured logging**: JSON format with correlation IDs
- **Log levels**: ERROR, WARN, INFO, DEBUG
- **Retention**: 30 days for application logs, 1 year for audit logs

### Health Checks
- **Database connectivity**: Connection pool status
- **External services**: Weather API, payment processor status
- **Application health**: Memory usage, response times