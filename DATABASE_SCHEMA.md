# Database Schema - Job-Task Analysis Application

## Entity-Relationship Diagram (ERD)

```
users
├── projects (1:M)
│   ├── team_members (1:M)
│   ├── job_roles (1:M)
│   ├── job_titles (1:M)
│   ├── job_postings (1:M)
│   │   ├── extracted_tasks (1:M)
│   │   ├── extracted_skills (1:M)
│   │   ├── extracted_knowledge (1:M)
│   │   ├── extracted_abilities (1:M)
│   │   ├── extracted_certifications (1:M)
│   │   ├── extracted_tools (1:M)
│   │   └── posting_tags (M:M)
│   │       ├── job_roles (M:M)
│   │       └── job_titles (M:M)
│   └── jta_reports (1:M)
│       └── report_items (1:M)
```

## Table Definitions

### 1. users
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    avatar_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);
```

### 2. projects
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    objectives TEXT,
    status VARCHAR(50) DEFAULT 'active', -- active, archived, completed
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);
```

### 3. team_members
```sql
CREATE TABLE team_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(50) NOT NULL, -- admin, editor, viewer
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, user_id)
);
```

### 4. job_roles
```sql
CREATE TABLE job_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(100), -- technical, non-technical, management, etc.
    is_primary BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_job_roles_project ON job_roles(project_id);
```

### 5. job_titles
```sql
CREATE TABLE job_titles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    role_id UUID REFERENCES job_roles(id) ON DELETE SET NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_job_titles_project ON job_titles(project_id);
CREATE INDEX idx_job_titles_role ON job_titles(role_id);
```

### 6. job_postings
```sql
CREATE TABLE job_postings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    source_url VARCHAR(2048) NOT NULL,
    source_platform VARCHAR(100), -- linkedin, indeed, glassdoor, etc.
    title VARCHAR(255),
    company VARCHAR(255),
    location VARCHAR(255),
    raw_content TEXT,
    html_content TEXT,
    status VARCHAR(50) DEFAULT 'imported', -- imported, processing, completed, failed
    processing_error TEXT,
    confidence_score FLOAT DEFAULT 0.0,
    extracted_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, source_url)
);

CREATE INDEX idx_postings_project ON job_postings(project_id);
CREATE INDEX idx_postings_status ON job_postings(status);
CREATE INDEX idx_postings_platform ON job_postings(source_platform);
```

### 7. posting_tags
```sql
CREATE TABLE posting_tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    role_id UUID REFERENCES job_roles(id) ON DELETE SET NULL,
    title_id UUID REFERENCES job_titles(id) ON DELETE SET NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(posting_id, role_id, title_id)
);

CREATE INDEX idx_posting_tags_posting ON posting_tags(posting_id);
CREATE INDEX idx_posting_tags_role ON posting_tags(role_id);
CREATE INDEX idx_posting_tags_title ON posting_tags(title_id);
```

### 8. extracted_tasks
```sql
CREATE TABLE extracted_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    task_text TEXT NOT NULL,
    category VARCHAR(100), -- primary, secondary, supporting
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    normalized_text TEXT, -- cleaned/normalized version
    embedding VECTOR(384), -- for semantic search (optional)
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tasks_posting ON extracted_tasks(posting_id);
CREATE INDEX idx_tasks_project ON extracted_tasks(project_id);
CREATE INDEX idx_tasks_normalized ON extracted_tasks USING gin(to_tsvector('english', normalized_text));
```

### 9. extracted_skills
```sql
CREATE TABLE extracted_skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    skill_name VARCHAR(255) NOT NULL,
    skill_type VARCHAR(50), -- technical, soft, domain
    category VARCHAR(100), -- required, preferred, nice-to-have
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    normalized_name VARCHAR(255),
    skill_level VARCHAR(50), -- beginner, intermediate, advanced, expert
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_skills_posting ON extracted_skills(posting_id);
CREATE INDEX idx_skills_project ON extracted_skills(project_id);
CREATE INDEX idx_skills_normalized ON extracted_skills(normalized_name);
```

### 10. extracted_knowledge
```sql
CREATE TABLE extracted_knowledge (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    knowledge_area VARCHAR(255) NOT NULL,
    category VARCHAR(100), -- domain, technical, business, regulatory
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_knowledge_posting ON extracted_knowledge(posting_id);
CREATE INDEX idx_knowledge_project ON extracted_knowledge(project_id);
```

### 11. extracted_abilities
```sql
CREATE TABLE extracted_abilities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    ability_name VARCHAR(255) NOT NULL,
    category VARCHAR(100), -- cognitive, psychomotor, affective
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_abilities_posting ON extracted_abilities(posting_id);
CREATE INDEX idx_abilities_project ON extracted_abilities(project_id);
```

### 12. extracted_certifications
```sql
CREATE TABLE extracted_certifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    certification_name VARCHAR(255) NOT NULL,
    issuing_body VARCHAR(255),
    category VARCHAR(100), -- required, preferred, nice-to-have
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_certs_posting ON extracted_certifications(posting_id);
CREATE INDEX idx_certs_project ON extracted_certifications(project_id);
```

### 13. extracted_tools
```sql
CREATE TABLE extracted_tools (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    posting_id UUID NOT NULL REFERENCES job_postings(id) ON DELETE CASCADE,
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    tool_name VARCHAR(255) NOT NULL,
    tool_type VARCHAR(50), -- programming_language, framework, platform, tool, ide, etc.
    category VARCHAR(100), -- required, preferred, nice-to-have
    frequency INT DEFAULT 1,
    confidence_score FLOAT DEFAULT 0.0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_tools_posting ON extracted_tools(posting_id);
CREATE INDEX idx_tools_project ON extracted_tools(project_id);
CREATE INDEX idx_tools_type ON extracted_tools(tool_type);
```

### 14. jta_reports
```sql
CREATE TABLE jta_reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
    created_by UUID NOT NULL REFERENCES users(id),
    title VARCHAR(255),
    description TEXT,
    filters JSONB, -- role filters, title filters, etc.
    superset_data JSONB, -- aggregated data snapshot
    export_format VARCHAR(50), -- pdf, excel, json
    file_path VARCHAR(500),
    status VARCHAR(50) DEFAULT 'generating', -- generating, ready, archived
    total_postings_analyzed INT,
    unique_skills_count INT,
    unique_tasks_count INT,
    unique_certifications_count INT,
    unique_tools_count INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    generated_at TIMESTAMP
);

CREATE INDEX idx_reports_project ON jta_reports(project_id);
CREATE INDEX idx_reports_created_by ON jta_reports(created_by);
CREATE INDEX idx_reports_status ON jta_reports(status);
```

### 15. report_items
```sql
CREATE TABLE report_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    report_id UUID NOT NULL REFERENCES jta_reports(id) ON DELETE CASCADE,
    item_type VARCHAR(50) NOT NULL, -- skill, task, certification, tool, knowledge, ability
    item_id VARCHAR(500), -- reference to original extracted item
    item_name VARCHAR(500),
    frequency INT,
    confidence_score FLOAT,
    prevalence_percentage FLOAT, -- % of postings that include this item
    related_items JSONB, -- related skills, tools, etc.
    importance_score INT, -- 1-10 scale
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_report_items_report ON report_items(report_id);
CREATE INDEX idx_report_items_type ON report_items(item_type);
```

### 16. audit_logs
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    project_id UUID REFERENCES projects(id),
    action VARCHAR(100), -- create, update, delete, export, etc.
    entity_type VARCHAR(100), -- posting, skill, report, etc.
    entity_id UUID,
    changes JSONB, -- before/after values
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_audit_project ON audit_logs(project_id);
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_action ON audit_logs(action);
```

## Indexes Strategy

### Performance Indexes
- Foreign key indexes for JOINs
- Project-scoped queries (filtered by project_id)
- Status-based queries (posting status, report status)
- Full-text search on normalized fields
- Composite indexes for common query patterns

## Partitioning Strategy

For large-scale deployments (100s of thousands of postings):
```sql
-- Partition extracted_skills by project_id
ALTER TABLE extracted_skills 
PARTITION BY HASH (project_id) PARTITIONS 16;
```

## Normalization Rules

- Skills/Tools/Certifications deduplicated via normalized_name field
- Frequency counters for metrics
- Confidence scores for extraction quality
- JSONB for flexible metadata storage

## Data Retention

- Active projects: indefinite retention
- Archived projects: 2-year retention before deletion
- Audit logs: 1-year retention
- Processing logs: 30-day retention
- Cache invalidation: 24-hour TTL

## Query Performance Patterns

### High-frequency queries
1. Get all skills for a project → indexed by project_id
2. Get posting by status → indexed by status
3. Get aggregated metrics → pre-aggregated in reports
4. Full-text search on task descriptions → GIN index on tsvector

### Optimization techniques
- Materialized views for frequent aggregations
- Query result caching (Redis)
- Batch processing for bulk updates
- Prepared statements to prevent SQL injection
