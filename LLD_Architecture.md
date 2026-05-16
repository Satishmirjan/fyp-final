# Low-Level Design (LLD) - GW-ImpactAI Learning Platform

## System Architecture Overview

Detailed component-level design showing module interactions, data structures, and service APIs.

## Frontend Architecture (PlantUML)

```plantuml
@startuml LLD_Frontend
package "React Components" {
  component "App.jsx" as APP
  component "Main.jsx" as MAIN
  
  package "Pages" {
    component "Home" as HOME
    component "Login" as LOGIN
    component "Signup" as SIGNUP
    component "Upload" as UPLOAD
    component "Summary" as SUMMARY
    component "StudyRoom" as STUDY
    component "QuizSession" as QUIZ
    component "Analytics" as ANALYTICS
    component "TeacherTools" as TEACHER
    component "Settings" as SETTINGS
  }
  
  package "Components" {
    component "Navbar" as NAVBAR
    component "VoiceAssistant" as VOICE
    component "AnimatedBackground" as BG
    component "ProtectedRoute" as PROTECTED
  }
  
  package "Context" {
    component "AuthContext" as AUTH_CTX
  }
  
  package "Services" {
    component "geminiService.js" as GEMINI_SVC
  }
}

package "External Libraries" {
  component "React Router DOM" as ROUTER_LIB
  component "Material-UI" as MUI_LIB
  component "Framer Motion" as FRAMER_LIB
  component "ReCharts" as CHARTS_LIB
  component "React Dropzone" as DROPZONE_LIB
  component "React Speech Kit" as SPEECH_LIB
}

MAIN --> APP
APP --> ROUTER_LIB
APP --> AUTH_CTX
APP --> NAVBAR
APP --> VOICE
APP --> BG

HOME --> FRAMER_LIB
LOGIN --> MUI_LIB
SIGNUP --> MUI_LIB
UPLOAD --> DROPZONE_LIB
SUMMARY --> CHARTS_LIB
STUDY --> FRAMER_LIB
QUIZ --> MUI_LIB
ANALYTICS --> CHARTS_LIB
TEACHER --> MUI_LIB
SETTINGS --> MUI_LIB

VOICE --> SPEECH_LIB
VOICE --> SPEECH_LIB

PROTECTED --> AUTH_CTX
PROTECTED --> ROUTER_LIB

@enduml
```

## Backend Architecture (PlantUML)

```plantuml
@startuml LLD_Backend
package "Flask Application" {
  component "app.py" as APP_MAIN
  
  package "Route Handlers" {
    component "/summary (POST)" as ROUTE_SUMMARY
    component "/api/signup (POST)" as ROUTE_SIGNUP
    component "/api/login (POST)" as ROUTE_LOGIN
    component "/audio/<filename> (GET)" as ROUTE_AUDIO
  }
  
  package "Middleware" {
    component "CORS Middleware" as CORS
    component "Error Handler" as ERROR_HANDLER
    component "Token Validator" as TOKEN_VALID
  }
  
  package "Services" {
    component "AuthService" as AUTH_SVC
    component "PDFProcessingService" as PDF_SVC
    component "SummarizationService" as SUMM_SVC
    component "FlashcardService" as FLASH_SVC
    component "QuizService" as QUIZ_SVC
    component "TTSService" as TTS_SVC
  }
  
  package "External Integrations" {
    component "PyMuPDF (fitz)" as PDFLIB
    component "HF Transformers" as HF_TRANS
    component "JWT Handler" as JWT
    component "Werkzeug Security" as WERKZEUG
    component "ElevenLabs API" as ELEVEN_API
  }
  
  package "Database" {
    component "MongoDB Connection" as MONGO_CONN
    component "Users Collection" as USERS_COL
  }
  
  package "Storage" {
    component "Temp File Handler" as TEMP_FILES
    component "Audio Output Dir" as AUDIO_DIR
  }
}

APP_MAIN --> CORS
APP_MAIN --> ERROR_HANDLER

ROUTE_SUMMARY --> PDF_SVC
ROUTE_SUMMARY --> SUMM_SVC
ROUTE_SUMMARY --> FLASH_SVC
ROUTE_SUMMARY --> QUIZ_SVC
ROUTE_SUMMARY --> TTS_SVC

ROUTE_SIGNUP --> AUTH_SVC
ROUTE_LOGIN --> AUTH_SVC

AUTH_SVC --> JWT
AUTH_SVC --> WERKZEUG
AUTH_SVC --> MONGO_CONN

PDF_SVC --> PDFLIB
PDF_SVC --> TEMP_FILES

SUMM_SVC --> HF_TRANS

FLASH_SVC --> HF_TRANS

QUIZ_SVC --> HF_TRANS

TTS_SVC --> ELEVEN_API
TTS_SVC --> AUDIO_DIR

MONGO_CONN --> USERS_COL

ROUTE_AUDIO --> AUDIO_DIR

@enduml
```

## Data Models

### User Model (MongoDB)
```json
{
  "_id": ObjectId,
  "name": String,
  "email": String,
  "password": String (hashed),
  "createdAt": DateTime,
  "updatedAt": DateTime
}
```

### Flashcard Model
```json
{
  "id": String,
  "question": String,
  "answer": String,
  "difficulty": String (easy/medium/hard),
  "category": String
}
```

### Quiz Question Model
```json
{
  "id": String,
  "question": String,
  "options": [String, String, String, String],
  "correctAnswer": String,
  "explanation": String
}
```

## API Endpoints

### Authentication APIs

#### POST /api/signup
```json
Request:
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secure_password"
}

Response:
{
  "message": "User created successfully"
}
```

#### POST /api/login
```json
Request:
{
  "email": "john@example.com",
  "password": "secure_password"
}

Response:
{
  "token": "jwt_token_here",
  "user": {
    "name": "John Doe",
    "email": "john@example.com"
  }
}
```

### Content Processing APIs

#### POST /summary
```json
Request:
{
  "file": File (PDF),
  "options": {
    "summary": Boolean,
    "audio": Boolean,
    "flashcards": Boolean,
    "quiz": Boolean
  }
}

Response:
{
  "summary": String,
  "audio_url": "/audio/filename.mp3",
  "flashcards": [Flashcard],
  "quiz": [QuizQuestion]
}
```

#### GET /audio/<filename>
Returns audio file for streaming

## Component Interactions

### PDF Upload Flow
```
User Upload → Upload Component → FormData (File + Options)
  ↓
API POST /summary
  ↓
Backend: File Received
  ↓
Extract Text (PyMuPDF)
  ↓
Summarize (BART)
  ↓
Generate Flashcards (Gemini)
  ↓
Generate Quiz (Gemini)
  ↓
Generate Audio (ElevenLabs)
  ↓
Response with all results
  ↓
Summary Component displays results
```

### Authentication Flow
```
User Input (Login/Signup)
  ↓
AuthContext.login() / AuthContext.signup()
  ↓
API POST /api/login or /api/signup
  ↓
Backend validates credentials / creates user
  ↓
Returns JWT token + user info
  ↓
Store token in localStorage
  ↓
Update AuthContext state
  ↓
Redirect to protected routes
```

## Frontend Component Hierarchy

```
AppWrapper
├── ThemeProvider
│   └── Router
│       └── AuthProvider
│           ├── Navbar
│           ├── VoiceAssistant
│           ├── AnimatedBackground
│           └── Routes
│               ├── Home
│               ├── Login
│               ├── Signup
│               ├── ProtectedRoute
│               │   ├── Upload
│               │   ├── Summary
│               │   ├── StudyRoom
│               │   ├── QuizSession
│               │   ├── Analytics
│               │   ├── TeacherTools
│               │   └── Settings
```

## Backend Service Details

### AuthService
- `signup(name, email, password)` → Creates user with hashed password
- `login(email, password)` → Validates credentials, returns JWT
- `validate_token(token)` → Validates JWT token

### PDFProcessingService
- `extract_text_from_pdf(file_path)` → Extracts text using PyMuPDF
- `split_into_chunks(text, max_words)` → Splits text into manageable chunks

### SummarizationService
- `summarize_large_text(text)` → Uses BART for summarization
- Handles chunking for large texts

### FlashcardService
- `generate_flashcards(text, max_flashcards)` → Generates flashcards using Gemini

### QuizService
- `generate_quiz(text, num_questions)` → Generates quiz questions using Gemini

### TTSService
- `generate_speech(text)` → Converts text to speech via ElevenLabs
- Returns audio file path

## Error Handling

### Frontend
- Try-catch blocks for API calls
- User-friendly error messages
- Token refresh on 401 responses

### Backend
- Global error handler middleware
- Traceback logging
- Graceful degradation for failed services

## Performance Considerations

1. **PDF Processing**: Chunked text processing for large PDFs
2. **Model Inference**: Summarization limited to 700 words per chunk
3. **Caching**: Browser localStorage for auth tokens
4. **Lazy Loading**: Route-based code splitting in React
5. **Audio Streaming**: Chunked audio file serving

## Security Implementation

1. **Authentication**
   - JWT tokens with 24-hour expiration
   - Bcrypt password hashing
   - Secure token storage in localStorage

2. **API Security**
   - CORS configuration
   - Token validation on protected routes
   - Input validation

3. **File Handling**
   - Temporary file cleanup
   - File type validation
   - Path traversal prevention
