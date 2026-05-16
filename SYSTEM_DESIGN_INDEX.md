# Complete System Design Documentation Index

## GW-ImpactAI Learning Platform - Design Artifacts

This document serves as the master index for all architectural documentation of the GW-ImpactAI Learning Platform.

---

## 📊 Quick Reference - All Diagrams

### 1. High-Level Design (HLD) - `HLD_Architecture.md`

**Purpose**: System-level architecture showing major components and their interactions.

**Key Diagrams**:
- **PlantUML Architecture Diagram**: Components and data flow
- **Mermaid System Diagram**: Service boundaries and dependencies

**Covers**:
- Client-Server architecture
- API Gateway layer
- Backend services
- External integrations
- Technology stack
- Security considerations

**Key Components**:
```
Frontend (React) → API Gateway (Flask) → Backend Services → External APIs & Storage
```

---

### 2. Low-Level Design (LLD) - `LLD_Architecture.md`

**Purpose**: Detailed component-level design with API specifications and interactions.

**Key Diagrams**:
- **Frontend Architecture**: React components, pages, context, services
- **Backend Architecture**: Flask route handlers, middleware, services
- **Component Hierarchy**: React component tree structure

**Covers**:
- Component breakdown and responsibilities
- API endpoint specifications (with request/response examples)
- Data flow for major use cases
- Service details and functions
- Error handling strategies
- Performance optimizations

**API Endpoints**:
- `POST /api/signup` - User registration
- `POST /api/login` - User authentication
- `POST /summary` - PDF processing & content generation
- `GET /audio/<filename>` - Audio file serving

---

### 3. Entity Relationship Diagram (ERD) - `ERD_Database_Schema.md`

**Purpose**: Database schema design and data relationships.

**Key Diagrams**:
- **PlantUML ER Diagram**: All collections and relationships
- **Mermaid ER Diagram**: Detailed entity definitions
- **Schema Examples**: JSON structure for each collection

**Collections**:
1. **Users** - User accounts and preferences
2. **LearningSession** - Study sessions created by users
3. **ProcessedContent** - Extracted and summarized content
4. **Flashcard** - Study flashcards generated
5. **QuizQuestion** - Quiz questions with multiple choice
6. **QuizAttempt** - User quiz attempt records
7. **AudioFile** - Generated speech audio files
8. **UserProgress** - Aggregated user learning statistics

**Relationships**:
- Users (1) → (Many) LearningSession, Flashcard, QuizQuestion, QuizAttempt, AudioFile
- LearningSession (1) → (Many) ProcessedContent, Flashcard, QuizQuestion, AudioFile
- Users (1) ↔ (1) UserProgress

---

## 🏗️ Architecture Layers

### Layer 1: Presentation Layer (Frontend)
```
├── React Application
├── Pages (Home, Upload, Summary, Quiz, Analytics, etc.)
├── Components (Navbar, Voice Assistant, Protected Routes)
├── Context (Auth Context for state management)
├── Services (API calls, Gemini integration)
└── Styling (Material-UI, TailwindCSS, Framer Motion)
```

### Layer 2: API/Integration Layer (Flask)
```
├── Route Handlers (API endpoints)
├── Middleware (CORS, Error handling, Auth)
├── Business Logic Services
│   ├── Auth Service (JWT, password hashing)
│   ├── PDF Processing Service
│   ├── Summarization Service
│   ├── Flashcard Service
│   ├── Quiz Service
│   └── TTS Service
└── External Integrations
    ├── Hugging Face Transformers
    ├── Google Gemini AI
    ├── ElevenLabs API
    └── PyMuPDF
```

### Layer 3: Data Access Layer
```
├── MongoDB Connection
├── Users Collection
├── Content Collections (LearningSession, ProcessedContent)
├── Study Materials (Flashcard, QuizQuestion)
└── User Metrics (QuizAttempt, UserProgress, AudioFile)
```

### Layer 4: External Services
```
├── ML/AI Services
│   ├── Hugging Face (Text Summarization - BART)
│   └── Google Gemini (Content Generation)
├── TTS Service
│   └── ElevenLabs API
└── Database
    └── MongoDB Atlas/Self-hosted
```

---

## 🔄 Data Flow Diagrams

### PDF Upload & Processing Flow
```
1. User selects PDF via Upload component
2. Frontend FormData includes file + processing options
3. POST /summary request sent to backend
4. Backend extracts text using PyMuPDF
5. If summary requested → BART summarization
6. If flashcards requested → Gemini generation
7. If quiz requested → Gemini generation
8. If audio requested → ElevenLabs TTS
9. All results compiled into response
10. Frontend displays results in Summary component
```

### Authentication Flow
```
1. User enters credentials on Login/Signup page
2. Frontend sends credentials to /api/login or /api/signup
3. Backend validates with MongoDB
4. Password verification using bcrypt
5. JWT token generated (24-hour expiration)
6. Token + user info returned to frontend
7. Token stored in localStorage
8. Token included in Authorization header for future requests
9. ProtectedRoute validates token before rendering
10. Protected pages accessible only with valid token
```

### Quiz Taking Flow
```
1. User navigates to QuizSession
2. Quiz questions loaded from database
3. User answers displayed in UI
4. Answers submitted to backend
5. Backend calculates score
6. Results stored in QuizAttempt collection
7. UserProgress updated with new statistics
8. Results displayed to user with analysis
```

---

## 🗄️ Database Collections Reference

### Users
- Stores user accounts, authentication credentials, preferences
- **Key Fields**: email (unique), password (hashed), role, preferences
- **Indexes**: email (unique), role

### LearningSession
- Represents each study session initiated by user
- **Key Fields**: userId, title, contentType, status
- **Indexes**: userId, status, createdAt

### ProcessedContent
- Contains extracted and summarized text from PDFs
- **Key Fields**: sessionId, originalText, summary, processingTime
- **Indexes**: userId, sessionId, createdAt

### Flashcard
- Study flashcards generated from content
- **Key Fields**: question, answer, difficulty, tags, reviewCount
- **Indexes**: userId, sessionId, difficulty, tags

### QuizQuestion
- Quiz questions with options and answers
- **Key Fields**: question, options[], correctAnswer, difficulty
- **Indexes**: sessionId, difficulty, category

### QuizAttempt
- Records of quiz attempts by users
- **Key Fields**: userId, answers[], score, percentage, status
- **Indexes**: userId, completedAt

### AudioFile
- Generated speech audio files
- **Key Fields**: filename, filePath, duration, format
- **Indexes**: contentId, userId, createdAt

### UserProgress
- Aggregated statistics about user learning
- **Key Fields**: totalSessionsCompleted, averageScore, streakDays
- **One-to-One with Users**: userId reference

---

## 🔐 Security Architecture

### Authentication
- **Method**: JWT (JSON Web Tokens)
- **Expiration**: 24 hours
- **Storage**: Browser localStorage
- **Validation**: Token_required decorator on protected routes

### Authorization
- **Frontend**: ProtectedRoute component checks auth context
- **Backend**: Token validation middleware on API routes
- **Roles**: Student, Teacher, Admin (extensible)

### Data Protection
- **Passwords**: Bcrypt hashing with salt
- **API Keys**: Environment variables (.env file)
- **CORS**: Configured for specific origins
- **File Upload**: Temporary storage with validation

### Network Security
- **HTTPS**: Recommended for production
- **CORS Headers**: Controlled cross-origin requests
- **Content Validation**: Input sanitization on backend
- **Error Handling**: Generic error messages (no info disclosure)

---

## 📈 Performance Optimization

### Frontend Optimizations
- **Code Splitting**: Route-based lazy loading
- **Component Memoization**: React memo for expensive components
- **State Management**: Context API for efficient state sharing
- **Image Optimization**: Material-UI icons (vector-based)
- **Caching**: Browser localStorage for tokens and preferences

### Backend Optimizations
- **Text Chunking**: Split large texts into 700-word chunks for summarization
- **Model Selection**: BART for efficient summarization
- **Async Processing**: Parallel flashcard and quiz generation
- **Database Indexing**: Strategic indexes on frequently queried fields
- **Connection Pooling**: MongoDB connection reuse

### Storage Optimization
- **Temporary Files**: Cleanup after processing
- **Audio Compression**: MP3 format for efficient storage
- **CDN**: Recommended for serving audio files

---

## 🧪 Testing Strategy

### Frontend Testing
- **Unit Tests**: Vitest for React components
- **Integration Tests**: React Testing Library for component interactions
- **E2E Tests**: Optional, can use Cypress or Playwright

### Backend Testing
- **Unit Tests**: pytest for service functions
- **Integration Tests**: API endpoint testing with sample data
- **Mocking**: External API mocking for consistent tests

### Test Coverage Target: 80%

---

## 🚀 Deployment Architecture

### Frontend Deployment
- **Build Tool**: Vite
- **Build Output**: Static assets
- **Hosting**: CDN (Vercel, Netlify, AWS CloudFront)
- **Environment**: Production, Staging, Development

### Backend Deployment
- **Framework**: Flask with Gunicorn/uWSGI
- **Container**: Docker recommended
- **Orchestration**: Kubernetes or Docker Compose
- **Environment**: Production, Staging, Development

### Database Deployment
- **Platform**: MongoDB Atlas (cloud) or self-hosted
- **Replication**: Multi-region setup recommended
- **Backup**: Automated daily backups with 30-day retention

---

## 📋 API Documentation Summary

### Base URL
```
Development: http://localhost:5000
Production: https://api.impactai-learning.com
```

### Authentication Endpoints
```
POST /api/signup      - Create new user
POST /api/login       - Authenticate user, get JWT
```

### Content Processing
```
POST /summary         - Process PDF, generate summary, flashcards, quiz, audio
GET /audio/<filename> - Serve generated audio file
```

### Error Responses
```
400 - Bad Request (validation error)
401 - Unauthorized (invalid/missing token)
404 - Not Found (resource doesn't exist)
500 - Internal Server Error (server error)
```

---

## 📚 Technology Stack Summary

| Category | Technology | Purpose |
|----------|-----------|---------|
| **Frontend** | React 18 | UI framework |
| | Vite | Build tool |
| | React Router | Navigation |
| | Material-UI | Component library |
| | TailwindCSS | Styling |
| | Framer Motion | Animations |
| | ReCharts | Data visualization |
| | React Dropzone | File uploads |
| | React Speech Kit | Voice interaction |
| **Backend** | Flask | Web framework |
| | Python 3.x | Language |
| | JWT | Authentication |
| | Bcrypt | Password hashing |
| | PyMuPDF | PDF processing |
| **ML/AI** | Hugging Face | Text summarization |
| | Google Gemini | Content generation |
| | ElevenLabs | Text-to-speech |
| **Database** | MongoDB | NoSQL database |
| **DevOps** | Docker | Containerization |
| | Git | Version control |
| | pytest/vitest | Testing |

---

## 🔧 Configuration Files Reference

### Frontend Configuration
- `vite.config.js` - Vite build configuration
- `tailwind.config.js` - TailwindCSS configuration
- `eslint.config.js` - Code linting rules
- `.env` - Environment variables (API endpoints)

### Backend Configuration
- `.env` - MongoDB URI, API keys, JWT secret
- `app.py` - Flask app configuration and routes
- `requirements.txt` - Python dependencies

---

## 📞 Service Dependencies

```
Frontend Dependencies:
  ├── React 18
  ├── React Router
  ├── Material-UI
  ├── TailwindCSS
  └── External: Google Gemini API

Backend Dependencies:
  ├── Flask
  ├── PyMuPDF (PDF processing)
  ├── Hugging Face Transformers
  ├── PyMongo (MongoDB driver)
  ├── PyJWT (JWT handling)
  ├── Werkzeug (Security utilities)
  └── External APIs:
      ├── Google Gemini
      ├── ElevenLabs
      └── MongoDB
```

---

## 📱 User Roles & Permissions

### Student Role
- ✅ Upload PDF files
- ✅ Generate summaries
- ✅ Create/review flashcards
- ✅ Take quizzes
- ✅ View personal analytics
- ✅ Access study room

### Teacher Role
- ✅ All student permissions
- ✅ Create quiz questions
- ✅ View class analytics
- ✅ Manage class materials
- ✅ Access teacher tools

### Admin Role
- ✅ All permissions
- ✅ User management
- ✅ System configuration
- ✅ Access audit logs

---

## 🎯 Future Enhancements

1. **Real-time Collaboration**
   - Study groups
   - Shared flashcards
   - Live quiz sessions

2. **Advanced Analytics**
   - Learning curve analysis
   - Predictive performance scoring
   - Personalized recommendations

3. **Mobile Application**
   - React Native mobile app
   - Offline studying capability
   - Mobile-optimized TTS

4. **Advanced AI Features**
   - Adaptive learning paths
   - Natural language Q&A
   - Video summarization

5. **Integration**
   - LMS integration (Canvas, Blackboard)
   - Calendar integration
   - Notification system (Email, SMS, Push)

---

## 📖 How to Use These Diagrams

1. **For System Overview**: Start with HLD_Architecture.md
2. **For Implementation Details**: Reference LLD_Architecture.md
3. **For Database Work**: Use ERD_Database_Schema.md
4. **For API Integration**: Check API Endpoints in LLD
5. **For Deployment**: Review Deployment Architecture section

---

## 📝 Document Maintenance

- **Last Updated**: 2024-01-15
- **Version**: 1.0
- **Maintainer**: Architecture Team
- **Review Frequency**: Quarterly or when major changes occur
