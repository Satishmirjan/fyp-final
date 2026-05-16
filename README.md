# GW-ImpactAI Learning Platform - Design Documentation

## 📚 Complete Architecture & Design Documentation

Welcome to the comprehensive design documentation for the **GW-ImpactAI Learning Platform**. This directory contains all architectural diagrams, design specifications, and technical documentation in both **PlantUML** and **Mermaid** formats.

---

## 📖 Quick Navigation

### 🎯 Start Here
- **New to the project?** → Read `SYSTEM_DESIGN_INDEX.md` first
- **Need a specific diagram?** → Check `COMPLETE_DIAGRAMS.md`
- **Building a feature?** → Reference `LLD_Architecture.md`
- **Understanding the system?** → Study `HLD_Architecture.md`
- **Working with data?** → Consult `ERD_Database_Schema.md`

---

## 📁 Document Overview

### 1. **HLD_Architecture.md** - High-Level Design
```
What: System-wide architecture and component overview
Who: Architects, Project Managers, Stakeholders
When: Project kickoff, design reviews, stakeholder discussions
Where: https://github.com/[repo]/DESIGN_DOCUMENTS/HLD_Architecture.md
```

**Contents:**
- System Overview & Key Components
- PlantUML Architecture Diagram
- Mermaid System Diagram
- Technology Stack
- Data Flow Diagrams
- Security Considerations
- Future Roadmap

**Key Diagrams:**
- System Architecture (Components & Dependencies)
- Technology Stack Reference
- Security Architecture

---

### 2. **LLD_Architecture.md** - Low-Level Design
```
What: Detailed component specifications and APIs
Who: Developers, Backend Engineers, Frontend Engineers
When: During development, code reviews, integration
Where: https://github.com/[repo]/DESIGN_DOCUMENTS/LLD_Architecture.md
```

**Contents:**
- Frontend Component Architecture (PlantUML)
- Backend Service Architecture (PlantUML)
- Data Models (JSON Schema)
- Complete API Endpoints with Examples
- Component Interactions & Flows
- Error Handling Strategies
- Performance Optimizations

**Key Diagrams:**
- React Component Hierarchy
- Flask Service Architecture
- API Request/Response Examples
- User Journey Flows

---

### 3. **ERD_Database_Schema.md** - Entity Relationship Diagram
```
What: Database design and data relationships
Who: Database Designers, Backend Developers, DBAs
When: Database design, optimization, migrations
Where: https://github.com/[repo]/DESIGN_DOCUMENTS/ERD_Database_Schema.md
```

**Contents:**
- Entity Relationship Diagrams (PlantUML & Mermaid)
- MongoDB Collection Schemas (with JSON examples)
- Relationships & Cardinality
- Data Validation Rules
- Indexing Strategy
- Backup & Recovery Strategy

**Key Diagrams:**
- Complete ER Diagram
- Collection Relationships
- Data Aggregation Examples

---

### 4. **COMPLETE_DIAGRAMS.md** - All Diagrams Repository
```
What: Comprehensive collection of all technical diagrams
Who: Everyone (reference document)
When: Anytime you need a specific diagram
Where: https://github.com/[repo]/DESIGN_DOCUMENTS/COMPLETE_DIAGRAMS.md
```

**Contents:**
- 11 Complete Diagrams (PlantUML + Mermaid)
- High-Level Architecture
- Frontend Component Tree
- Backend Services
- Database ER Diagram
- User Journey Flow
- Data Processing Pipeline
- Authentication & Security Flow
- System Sequence Diagram
- Deployment Architecture
- Error Handling Flow

**Included Diagrams:**
1. HLD Complete Architecture
2. Frontend Component Tree
3. Backend Services Architecture
4. Database ER Diagram
5. User Journey Flow
6. Data Processing Pipeline
7. Authentication & Security Flow
8. System Sequence Diagram
9. Deployment Architecture
10. Error Handling Flow
11. Mermaid Complete Architecture

---

### 5. **SYSTEM_DESIGN_INDEX.md** - Master Index
```
What: Master reference guide for all documentation
Who: Everyone (comprehensive overview)
When: Understanding the complete system
Where: https://github.com/[repo]/DESIGN_DOCUMENTS/SYSTEM_DESIGN_INDEX.md
```

**Contents:**
- Architecture Layers Breakdown
- Data Flow Diagrams & Sequences
- Database Collections Reference
- Security Architecture Details
- Performance Optimizations
- Testing Strategy
- Deployment Architecture
- Technology Stack Summary
- Service Dependencies
- User Roles & Permissions
- Future Enhancements
- Document Maintenance Info

---

## 🏗️ Architecture at a Glance

```
┌─────────────────┐
│  React Browser  │
│   (Vite)        │
└────────┬────────┘
         │
    HTTP/REST
         │
┌────────▼────────┐
│  Flask API      │
│  (Port 5000)    │
└────────┬────────┘
         │
    ┌────┴────┬────────┬──────────┬─────────┐
    │          │        │          │         │
    ▼          ▼        ▼          ▼         ▼
MongoDB    PyMuPDF  HuggingFace  Gemini  ElevenLabs
(Auth)     (PDF)    (Summary)    (Gen)   (TTS)
```

---

## 🎯 Core Features Mapped to Architecture

### User Authentication
- **Frontend**: Login/Signup pages, Auth Context
- **Backend**: `/api/login`, `/api/signup` endpoints
- **Database**: Users collection
- **Security**: JWT tokens, bcrypt passwords

### PDF Processing & Summarization
- **Frontend**: Upload page, Summary display
- **Backend**: `/summary` endpoint, PDF extraction
- **ML/AI**: BART model via Hugging Face
- **Storage**: Temporary file handling

### Flashcard Generation
- **Frontend**: StudyRoom component
- **Backend**: Flashcard generation service
- **ML/AI**: Google Gemini API
- **Database**: Flashcard collection

### Quiz System
- **Frontend**: QuizSession component
- **Backend**: Quiz generation & scoring
- **ML/AI**: Google Gemini API
- **Database**: QuizQuestion & QuizAttempt collections

### Audio Generation
- **Frontend**: Summary page with audio player
- **Backend**: TTS service
- **External**: ElevenLabs API
- **Storage**: Audio file system

### Analytics
- **Frontend**: Analytics dashboard with ReCharts
- **Backend**: Progress aggregation endpoints
- **Database**: UserProgress collection

---

## 📊 Data Models Summary

```javascript
// Users - Authentication & Identity
{
  _id, name, email, password (hashed), role, preferences, createdAt
}

// LearningSession - Study Sessions
{
  _id, userId, title, contentType, status, fileMetadata, createdAt
}

// ProcessedContent - Extracted & Summarized Text
{
  _id, sessionId, userId, originalText, summary, processingTime, createdAt
}

// Flashcard - Study Materials
{
  _id, sessionId, userId, question, answer, difficulty, tags, reviewCount, createdAt
}

// QuizQuestion - Assessment Items
{
  _id, sessionId, userId, question, options[], correctAnswer, difficulty, createdAt
}

// QuizAttempt - Assessment Results
{
  _id, userId, quizId, answers[], score, percentage, status, completedAt, createdAt
}

// AudioFile - Generated Speech Files
{
  _id, contentId, userId, filename, filePath, duration, format, createdAt
}

// UserProgress - Learning Statistics
{
  _id, userId, totalSessionsCompleted, averageScore, streakDays, updatedAt
}
```

---

## 🔄 Key Data Flows

### 1. **PDF Upload & Processing Flow**
```
User Upload PDF
    ↓
Upload Component sends File + Options
    ↓
POST /summary endpoint
    ↓
Backend extracts text (PyMuPDF)
    ↓
Parallel Processing:
  ├─ Summarize (BART)
  ├─ Generate Flashcards (Gemini)
  ├─ Generate Quiz (Gemini)
  └─ Generate Audio (ElevenLabs)
    ↓
Store in MongoDB
    ↓
Return results to Frontend
    ↓
Display in Summary page
```

### 2. **Authentication Flow**
```
Credentials Input
    ↓
POST /api/login or /api/signup
    ↓
Validate in MongoDB
    ↓
Hash/Verify password
    ↓
Generate JWT token
    ↓
Return token to frontend
    ↓
Store in localStorage
    ↓
Include in Authorization header for protected routes
```

### 3. **Quiz Taking Flow**
```
Quiz Start
    ↓
Load questions from database
    ↓
Display questions in UI
    ↓
User submits answers
    ↓
Backend calculates score
    ↓
Store attempt in QuizAttempt collection
    ↓
Update UserProgress statistics
    ↓
Display results to user
```

---

## 🔐 Security Features

| Feature | Implementation |
|---------|----------------|
| **Authentication** | JWT tokens with 24-hour expiration |
| **Password Security** | Bcrypt hashing with salt |
| **API Protection** | Token validation on protected routes |
| **CORS** | Configured for specific origins |
| **File Upload** | Temporary storage with validation |
| **API Keys** | Environment variables (.env) |
| **Error Handling** | Generic error messages (no info disclosure) |

---

## 📈 Performance Metrics

| Component | Optimization | Target |
|-----------|-------------|--------|
| **PDF Processing** | Chunked text (700 words) | <5s for 50-page PDF |
| **Summarization** | BART model | <10s response |
| **Flashcard Gen** | Parallel requests | <15s for 10 cards |
| **Quiz Gen** | Parallel requests | <15s for 5 questions |
| **TTS Generation** | Streaming MP3 | <30s for 500 words |
| **Frontend Load** | Code splitting | <3s initial page load |
| **API Response** | Caching layer | <500ms avg |

---

## 🚀 Deployment Considerations

### Development
```
Frontend: npm run dev (Vite on :5173)
Backend: python app.py (Flask on :5000)
Database: Local MongoDB or Docker container
```

### Production
```
Frontend: CDN (Vercel/Netlify) + CloudFront
Backend: Gunicorn + Load Balancer
Database: MongoDB Atlas with replicas
Cache: Redis for session management
Storage: Cloud storage for audio files
```

---

## 📋 Technology Stack

### Frontend
- **React 18** - UI framework
- **Vite** - Build tool
- **TailwindCSS** - Styling
- **Material-UI** - Components
- **React Router** - Navigation
- **Framer Motion** - Animations
- **ReCharts** - Data visualization

### Backend
- **Flask** - Web framework
- **Python 3.x** - Language
- **PyMuPDF** - PDF processing
- **PyJWT** - JWT handling
- **Bcrypt** - Password hashing
- **PyMongo** - MongoDB driver

### External Services
- **Hugging Face** - Text summarization (BART)
- **Google Gemini** - Content generation
- **ElevenLabs** - Text-to-speech

### Infrastructure
- **MongoDB** - Primary database
- **Redis** - Caching (optional)
- **Docker** - Containerization
- **Git** - Version control

---

## 📚 API Documentation Quick Reference

### Authentication Endpoints
```
POST /api/signup    - Register new user
POST /api/login     - Authenticate user, get JWT
```

### Content Processing
```
POST /summary       - Process PDF, generate content
GET /audio/<file>   - Serve audio file
```

### Error Codes
```
200 - OK
201 - Created
400 - Bad Request
401 - Unauthorized
404 - Not Found
500 - Internal Server Error
```

---

## 🧪 Testing Strategy

### Frontend Testing
- **Unit Tests**: Vitest for components
- **Integration Tests**: React Testing Library
- **E2E Tests**: Cypress/Playwright (optional)
- **Target Coverage**: 80%

### Backend Testing
- **Unit Tests**: pytest for services
- **Integration Tests**: API endpoint testing
- **Test Data**: Fixtures and mocks
- **Target Coverage**: 80%

---

## 🔄 CI/CD Pipeline

```
Git Push
    ↓
Run Tests (Frontend + Backend)
    ↓
Build Artifacts
    ↓
Deploy to Staging
    ↓
Run Smoke Tests
    ↓
Manual Approval
    ↓
Deploy to Production
    ↓
Monitor & Alert
```

---

## 📞 Getting Help

### For Architecture Questions
- Review `HLD_Architecture.md` for system overview
- Check `SYSTEM_DESIGN_INDEX.md` for detailed breakdown

### For Implementation Questions
- Consult `LLD_Architecture.md` for component details
- Check `COMPLETE_DIAGRAMS.md` for specific flows

### For Database Questions
- Reference `ERD_Database_Schema.md` for data models
- Check collection schemas and relationships

### For Specific Diagrams
- All diagrams available in `COMPLETE_DIAGRAMS.md`
- PlantUML diagrams can be viewed at: http://www.plantuml.com/plantuml/uml/
- Mermaid diagrams can be viewed at: https://mermaid.live/

---

## 🎓 Learning Path

### For New Developers
1. Read `SYSTEM_DESIGN_INDEX.md` - Understand overall structure
2. Study `HLD_Architecture.md` - Learn system components
3. Review `LLD_Architecture.md` - Understand implementation details
4. Reference `ERD_Database_Schema.md` - Learn data models
5. Study specific diagrams in `COMPLETE_DIAGRAMS.md`

### For DevOps Engineers
1. Review deployment architecture in `SYSTEM_DESIGN_INDEX.md`
2. Check technology stack in `HLD_Architecture.md`
3. Study `COMPLETE_DIAGRAMS.md` - Deployment architecture section

### For Database Developers
1. Study `ERD_Database_Schema.md` completely
2. Review indexing strategy and backup procedures
3. Check aggregation pipeline examples

### For QA Engineers
1. Read data flows in `SYSTEM_DESIGN_INDEX.md`
2. Study user journeys in `COMPLETE_DIAGRAMS.md`
3. Review error handling flow

---

## 📝 Document Maintenance

| Document | Last Updated | Version | Status |
|----------|-------------|---------|--------|
| HLD_Architecture.md | 2024-01-15 | 1.0 | Active |
| LLD_Architecture.md | 2024-01-15 | 1.0 | Active |
| ERD_Database_Schema.md | 2024-01-15 | 1.0 | Active |
| COMPLETE_DIAGRAMS.md | 2024-01-15 | 1.0 | Active |
| SYSTEM_DESIGN_INDEX.md | 2024-01-15 | 1.0 | Active |
| README.md | 2024-01-15 | 1.0 | Active |

### Update Guidelines
- Update diagrams when architecture changes
- Add new diagrams for new features
- Review quarterly or when major changes occur
- Keep version numbers updated

---

## 🔗 Related Resources

- **Repository**: https://github.com/[your-org]/GW-ImpactAI-Hackathon
- **API Documentation**: `/server/README.md`
- **Frontend Setup**: `/client/README.md`
- **Contributing**: `/CONTRIBUTING.md`
- **License**: `/LICENSE`

---

## 📞 Support & Feedback

- **Questions?** Create an issue in the repository
- **Design Feedback?** Open a discussion
- **Bug in Documentation?** Submit a PR
- **Architecture Review?** Schedule a sync with the team

---

## 📄 License

This documentation is part of the GW-ImpactAI Learning Platform project and is licensed under the same terms as the main project.

---

**Last Updated**: January 15, 2024  
**Documentation Version**: 1.0  
**Maintainer**: Architecture Team  
**Status**: Active & Complete

---

## Quick Links

- 📖 [System Design Index](SYSTEM_DESIGN_INDEX.md)
- 🏗️ [HLD Architecture](HLD_Architecture.md)
- 🔧 [LLD Architecture](LLD_Architecture.md)
- 🗄️ [ERD Database Schema](ERD_Database_Schema.md)
- 📊 [Complete Diagrams](COMPLETE_DIAGRAMS.md)

---

**Start here** → Choose your role above and follow the learning path! 🚀
