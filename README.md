# HireMind: System Architecture & UML Modeling

This document presents the structural and behavioral design of the **HireMind Smart AI Recruitment Platform**.

## 1. Entity Relationship Diagram (ERD)

The database follows a 3NF structure to ensure scalability and data integrity.

```mermaid
erDiagram
    USERS {
        int id PK
        string email UK
        string password_hash
        string full_name
        string role
        int company_id FK
        timestamp created_at
    }
    COMPANIES {
        int id PK
        string name
        string industry
        string description
        string website
        string logo_url
    }
    CANDIDATES {
        int id PK
        int user_id FK
        text bio
        string contact_number
        string current_title
    }
    JOBS {
        int id PK
        int company_id FK
        int posted_by_id FK
        string title
        text description
        string experience_level
        string status
        timestamp posted_at
    }
    RESUMES {
        int id PK
        int candidate_id FK
        string file_path
        text raw_text
        jsonb parsed_json
    }
    APPLICATIONS {
        int id PK
        int job_id FK
        int candidate_id FK
        int resume_id FK
        decimal ai_score
        text ai_feedback
        string status
    }
    SKILLS {
        int id PK
        string name UK
        string category
    }
    JOB_SKILLS {
        int job_id PK, FK
        int skill_id PK, FK
        decimal weight
    }
    CANDIDATE_SKILLS {
        int candidate_id PK, FK
        int skill_id PK, FK
        int years_exp
        string proficiency
    }

    USERS ||--o| CANDIDATES : "is a"
    USERS ||--o| COMPANIES : "belongs to (employer)"
    COMPANIES ||--o{ JOBS : "posts"
    JOBS ||--o{ APPLICATIONS : "receives"
    CANDIDATES ||--o{ APPLICATIONS : "submits"
    CANDIDATES ||--o{ RESUMES : "uploads"
    RESUMES ||--|| APPLICATIONS : "linked to"
    JOBS ||--o{ JOB_SKILLS : "requires"
    CANDIDATES ||--o{ CANDIDATE_SKILLS : "possesses"
    SKILLS ||--o{ JOB_SKILLS : "categorizes"
    SKILLS ||--o{ CANDIDATE_SKILLS : "categorizes"
```

---

## 2. Use Case Diagram

Models actor interactions with the platform's core functional modules.

```mermaid
useCaseDiagram
    actor Candidate
    actor Employer
    
    package "User Management" {
        usecase "Login/Registration" as UC1
        usecase "Manage Profile" as UC2
    }
    
    package "Recruitment Process" {
        usecase "Upload Resume" as UC3
        usecase "Search & Apply for Jobs" as UC4
        usecase "View Match Score/Feedback" as UC5
        usecase "Post Job Vacancies" as UC6
        usecase "Manage Applicants" as UC7
        usecase "View AI-Ranked Candidates" as UC8
    }
    
    Candidate --> UC1
    Candidate --> UC2
    Candidate --> UC3
    Candidate --> UC4
    Candidate --> UC5
    
    Employer --> UC1
    Employer --> UC2
    Employer --> UC6
    Employer --> UC7
    Employer --> UC8
```

---

## 3. Class Diagram (Backend Services)

Shows the object-oriented structure of the Node.js backend.

```mermaid
classDiagram
    class User {
        +int id
        +string email
        +role enum
        +register()
        +login()
    }
    class AuthService {
        +hashPassword(password)
        +generateToken(user)
        +verifyToken(token)
    }
    class JobService {
        +createJob(data)
        +getJobs(filter)
        +updateStatus(jobId, status)
    }
    class ApplicationService {
        +submitApplication(candidateId, jobId, resumeId)
        +rankCandidates(jobId)
        +updateStatus(applicationId, status)
    }
    class AIService {
        +parseResume(filePath)
        +matchResumeToJob(resumeId, jobId)
        +getFeedback(score)
    }
    class Database {
        +query(sql, params)
        +transaction(callback)
    }

    User -- AuthService : uses
    ApplicationService ..> AIService : calls
    ApplicationService ..> Database : persists
    JobService ..> Database : persists
    JobService o-- User : posted_by
```

---

## 4. Sequence Diagrams

### 4.1 User Registration & Profile Initialization
```mermaid
sequenceDiagram
    participant C as Candidate (React)
    participant B as Backend (Node.js)
    participant DB as SQL Database

    C->>B: POST /api/register (email, password, role)
    B->>B: Hash Password
    B->>DB: INSERT INTO users ...
    DB-->>B: User Created (ID)
    alt role == 'candidate'
        B->>DB: INSERT INTO candidates (user_id) ...
    else role == 'employer'
        B->>DB: UPDATE users SET company_id = ...
    end
    B-->>C: 201 Created & JWT Token
```

### 4.2 AI-Powered Job Application Flow
```mermaid
sequenceDiagram
    participant C as Candidate (React)
    participant B as Backend (Node.js)
    participant AI as AI Service (Python)
    participant DB as SQL Database

    C->>B: POST /api/apply (job_id, resume_file)
    B->>B: Save file to storage
    B->>DB: INSERT INTO resumes (candidate_id, file_path)
    DB-->>B: resume_id
    B->>AI: POST /analyze-resume (raw_text)
    AI->>AI: NLP Extraction (BERT)
    AI-->>B: Extracted Skills & Experience (JSON)
    B->>DB: UPDATE resumes SET parsed_json = ...
    B->>AI: POST /match (resume_data, job_requirements)
    AI-->>B: Score (e.g., 85%) & AI Feedback
    B->>DB: INSERT INTO applications (candidate_id, job_id, score, feedback)
    DB-->>B: Success
    B-->>C: Application Submitted & Initial Score Displayed
```

### 4.3 Employer: Posting a Job & Skill Weighting
```mermaid
sequenceDiagram
    participant E as Employer (React)
    participant B as Backend (Node.js)
    participant DB as SQL Database

    E->>B: POST /api/jobs (title, description, skills[])
    loop For each skill
        B->>DB: Check if skill exists
        alt Skill missing
            B->>DB: INSERT INTO skills (name)
        end
    end
    B->>DB: INSERT INTO jobs ...
    B->>DB: INSERT INTO job_skills (job_id, skill_id, weight)
    B-->>E: Job Posted Successfully
```
