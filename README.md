# Database Documentation: HireMind AI Recruitment Platform

## 1. Database Overview
The HireMind database is a robust relational storage system designed to facilitate the end-to-end recruitment lifecycle. It serves as the central integration point for:
- **React Frontend**: Consumes data via APIs to display dashboards, job listings, and candidate profiles.
- **Node.js Backend**: Manages business logic, user authentication, and coordinates data flow between the UI and AI services.
- **Python AI Components**: Enriches the database by parsing resumes into structured formats (JSONB) and calculating semantic matching scores between applicants and job vacancies.

## 2. Database Architecture
### Design Philosophy
The system follows the **Standard 3rd Normal Form (3NF)** to eliminate data redundancy and ensure transactional integrity. This design allows for horizontal scaling and maintains high performance even with large volumes of candidate and job data.

### Key Decisions
- **Relational Integrity**: Strict Foreign Key constraints ensure that job postings or applications are never orphaned.
- **Role-Based Separation**: Entities are separated by functional role (Candidate vs. Employer) while sharing a centralized `users` table for authentication.

## 3. Entity Descriptions

### 3.1 `users`
Central table for system-wide identity management.
| Column | Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `id` | SERIAL | Primary key for the user. | PRIMARY KEY |
| `email` | VARCHAR(255) | Unique login identifier. | UNIQUE, NOT NULL |
| `password_hash` | VARCHAR(255) | Encrypted password storage. | NOT NULL |
| `full_name` | VARCHAR(100) | Full legal name of the user. | NOT NULL |
| `role` | VARCHAR(20) | Permissions role: `candidate`, `employer`. | CHECK, NOT NULL |
| `created_at` | TIMESTAMP | Audit timestamp of registration. | DEFAULT NOW() |

### 3.2 `jobs`
Stores career vacancy details posted by companies.
| Column | Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `id` | SERIAL | Primary key for the job post. | PRIMARY KEY |
| `company_id` | INTEGER | The company hosting the vacancy. | FK, NOT NULL |
| `title` | VARCHAR(200) | Descriptive title of the role. | NOT NULL |
| `status` | VARCHAR(20) | Current state: `open`, `closed`, `draft`. | DEFAULT 'open' |

### 3.3 `resumes`
Stores candidate CV materials and AI-extracted data.
| Column | Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `id` | SERIAL | Primary key. | PRIMARY KEY |
| `candidate_id` | INTEGER | Reference to the profile owner. | FK, NOT NULL |
| `parsed_json` | JSONB | Structured AI extraction (skills, exp). | NULLABLE |

*(Full documentation covers all entities including `applications`, `skills`, `companies`, `candidates`, and `job_skills`/`candidate_skills`.)*

## 4. Relationships
- **One-to-Many (1:N)**: A `Company` can post many `Jobs`. A `Candidate` can upload multiple `Resumes`.
- **Many-to-Many (N:N)**: `Jobs` and `Skills` are linked via `job_skills`. `Candidates` and `Skills` are linked via `candidate_skills`.
- **One-to-One (1:1)**: Every `Candidate` or `Recruiter` record is linked to exactly one record in the `users` table.

## 5. ERD Reference
The Entity Relationship Diagram (ERD) visualizes the database where the `applications` table acts as the central transaction hub, linking the `jobs` (employers) side with the `resumes` (candidates) side.

## 6. Performance Considerations
### Indexing Strategy
- **Search Optimization**: B-tree indexes are applied to `email`, `job_title`, and `company_id` for fast lookups.
- **AI Score Retrieval**: An index on `applications.ai_score` allows Recruiters to instantly sort thousands of candidates by compatibility.

## 7. Data Integrity
- **Data Protection**: Sensitive PII (Personally Identifiable Information) like contact numbers and resumes are secured via application-level encryption and DB-level access controls.on**: Sensitive PII (Personally Identifiable Information) like contact numbers and resumes are secured via application-level encryption and DB-level access controls.
- **Audit Trails**: Every critical change is logged in the `audit_logs` table for compliance and security auditing.

## 9. Example Queries

### Insert a New Job Post
```sql
INSERT INTO jobs (company_id, title, description, status) 
VALUES (1, 'Senior AI Engineer', 'Python and NLP expert required.', 'open');
```

### Rank Candidates for a Job
```sql
SELECT c.full_name, a.ai_score 
FROM applications a
JOIN users c ON a.candidate_id = c.id
WHERE a.job_id = 101
ORDER BY a.ai_score DESC;
```

### Update Application Status
```sql
UPDATE applications 
SET status = 'shortlisted' 
WHERE id = 505;
```
