# Job-Task Analysis (JTA) Application - Architecture

## System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend Layer                            │
│  React.js + Redux | Responsive Web UI | Real-time Updates       │
└──────────────────────────────────┬──────────────────────────────┘
                                   │
                    API Gateway / Load Balancer
                                   │
┌──────────────────────────────────┴──────────────────────────────┐
│                      Backend Layer (FastAPI)                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Auth Service | Project Service | Analysis Service      │   │
│  │ Posting Service | Report Service | Export Service      │   │
│  └─────────────────────────────────────────────────────────┘   │
└──────────────────────────────────┬──────────────────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
    ┌────▼────┐           ┌────────▼────────┐        ┌──────▼──────┐
    │PostgreSQL│           │ Redis + Celery │        │File Storage │
    │Database  │           │Task Queue      │        │(S3/GCS)     │
    └──────────┘           └────────┬────────┘        └─────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    │                               │
            ┌──────▼─────────┐          ┌──────────▼──────┐
            │NLP/ML Pipeline │          │Web Scraper      │
            │(Extraction)    │          │(Job Posting)    │
            └────────────────┘          └─────────────────┘
```

## Service Architecture

### 1. Authentication Service
- JWT-based authentication
- OAuth 2.0 integration
- Role-based access control (RBAC)
- Session management

### 2. Project Service
- Project CRUD operations
- Team member management
- Project configuration
- Permission management

### 3. Job Posting Service
- URL validation and ingestion
- Batch import processing
- Posting storage and versioning
- Duplicate detection

### 4. Extraction Service
- AI/ML-based data extraction
- NLP processing pipeline
- Confidence scoring
- Manual review workflow

### 5. Analysis Service
- Aggregation and deduplication
- Frequency analysis
- Correlation analysis
- Superset generation

### 6. Report Service
- Report generation and formatting
- Template management
- Export handling
- Archive management

### 7. Export Service
- PDF generation
- Excel export
- JSON serialization
- Custom format support

## Technology Details

### Backend Framework: FastAPI
- High performance async framework
- Automatic API documentation
- Built-in validation
- WebSocket support for real-time updates

### Database: PostgreSQL
- ACID compliance
- Complex queries support
- Full-text search capabilities
- JSON data type support

### Task Queue: Celery + Redis
- Asynchronous job processing
- Distributed task execution
- Progress tracking
- Error handling and retries

### NLP Pipeline
- spaCy for entity extraction
- Hugging Face models for classification
- LangChain for LLM integration
- Custom training for domain-specific terms

### Frontend: React.js
- Component-based architecture
- State management with Redux
- Real-time updates with WebSockets
- Responsive design

## Deployment Architecture

```
GitHub Repository
        ↓
GitHub Actions (CI/CD)
        ↓
┌───────────────────────────────┐
│   Docker Image Build          │
│   - Run Tests                 │
│   - Security Scan             │
│   - Push to Registry          │
└───────────────────────────────┘
        ↓
┌───────────────────────────────┐
│  Kubernetes Cluster           │
│  ├─ API Pods (3 replicas)    │
│  ├─ Worker Pods (5 replicas) │
│  ├─ Database Pod             │
│  ├─ Redis Pod                │
│  └─ Ingress Controller        │
└───────────────────────────────┘
        ↓
┌───────────────────────────────┐
│  Monitoring & Logging         │
│  - Prometheus                 │
│  - ELK Stack                  │
│  - Sentry (Error Tracking)    │
└───────────────────────────────┘
```

## API Endpoints Structure

### Authentication
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/refresh` - Refresh token
- `POST /api/auth/logout` - User logout

### Projects
- `GET /api/projects` - List all projects
- `POST /api/projects` - Create new project
- `GET /api/projects/{id}` - Get project details
- `PUT /api/projects/{id}` - Update project
- `DELETE /api/projects/{id}` - Delete project

### Team Members
- `GET /api/projects/{id}/members` - List team members
- `POST /api/projects/{id}/members` - Add team member
- `PUT /api/projects/{id}/members/{member_id}` - Update member role
- `DELETE /api/projects/{id}/members/{member_id}` - Remove member

### Job Roles & Titles
- `GET /api/job-roles` - List job roles
- `POST /api/job-roles` - Create job role
- `GET /api/job-titles` - List job titles
- `POST /api/job-titles` - Create job title

### Job Postings
- `POST /api/projects/{id}/postings/upload-url` - Upload single URL
- `POST /api/projects/{id}/postings/batch-upload` - Batch upload URLs
- `GET /api/projects/{id}/postings` - List postings
- `GET /api/projects/{id}/postings/{posting_id}` - Get posting details
- `PUT /api/projects/{id}/postings/{posting_id}/tag` - Tag posting
- `DELETE /api/projects/{id}/postings/{posting_id}` - Delete posting

### Extracted Data
- `GET /api/projects/{id}/skills` - Get all extracted skills
- `GET /api/projects/{id}/tasks` - Get all extracted tasks
- `GET /api/projects/{id}/certifications` - Get certifications
- `GET /api/projects/{id}/tools` - Get tools/technologies
- `GET /api/projects/{id}/knowledge` - Get knowledge areas
- `GET /api/projects/{id}/abilities` - Get abilities

### Reports
- `POST /api/projects/{id}/reports/generate` - Generate JTA report
- `GET /api/projects/{id}/reports` - List generated reports
- `GET /api/projects/{id}/reports/{report_id}` - Get report details
- `POST /api/projects/{id}/reports/{report_id}/export` - Export report

### Health & Monitoring
- `GET /api/health` - System health check
- `GET /api/metrics` - Prometheus metrics

## Data Flow

### Job Posting Processing Flow
```
1. User submits URL(s)
   ↓
2. URL Validation
   ↓
3. Web Scraping (Selenium/BeautifulSoup)
   ↓
4. Content Extraction
   ↓
5. Store Raw Posting
   ↓
6. Queue for NLP Processing (Celery Task)
   ↓
7. Extract Entities (Skills, Tasks, etc.)
   ↓
8. Confidence Scoring
   ↓
9. Store Extracted Data
   ↓
10. Mark as Ready for Review
   ↓
11. User Review/Refinement (Optional)
   ↓
12. Finalize Extraction
```

### Report Generation Flow
```
1. User clicks "Generate JTA Report"
   ↓
2. Select target roles/titles
   ↓
3. Query all relevant postings
   ↓
4. Aggregate extracted data
   ↓
5. Deduplicate similar entries
   ↓
6. Calculate frequency/metrics
   ↓
7. Format for export
   ↓
8. Generate PDF/Excel/JSON
   ↓
9. Store report
   ↓
10. Provide download link
```

## Scalability Considerations

### Horizontal Scaling
- Stateless API services (auto-scaling)
- Distributed Celery workers
- PostgreSQL read replicas for queries
- Redis cluster for caching

### Vertical Scaling
- Increased worker pod resources
- Database optimization (indexing, partitioning)
- Caching strategy (Redis)
- Load balancing

### Performance Optimization
- Pagination for large datasets
- Database query optimization
- Response caching
- Async operations for long-running tasks
- Batch processing for bulk operations

## Security Measures

- HTTPS/TLS encryption
- SQL injection prevention (ORM + parameterized queries)
- XSS protection
- CSRF tokens
- Rate limiting
- Input validation and sanitization
- Audit logging
- Secret management (environment variables)
- Regular security updates
